Wolfenstein 3D code design document
===================================
Analysing and documenting the source code workings of Wolfenstein 3D for DOS. (This document should be read in an editor with a monospaced font and line wrapping enabled. This document follows the Markdown syntax.)


Table of contents
-----------------

	- Conventions and nomenclature
	|
	- 2D or 3D
	|
	- Compression algorithms
	| |- RLEW compression
	| |- Carmack compression
	| |- Huffman compression
	|
	- Data files
	| |- File extensions
	| |- Graphics
	|    |- Pics
	|    |- Sprites
	|
	| |- Audio
	| |- Maps
	|    |- MAPHEAD
	|    |- GAMEMAPS
	|    |- Extracting the maps
	|
	- Known bugs and limitations


Conventions and nomenclature
----------------------------
Decimal integer numbers are given as regular numbers or rarely suffixed with "d", binary numbers are suffixed with "b" or prefixed with "0b", octal numbers with "o" and hexadecimal numbers are either suffixed with "h" or prefixed with "0x" or "#". Example:

	13 = 13d = 1011b = 15o = Dh = 0xD

All continuous numbers are written in the standard big-endian notation used in the English language. This means the left-most digit is the most significant one and a number like 1011b is computed as (1 * 2^3 + 0 * 2^2 + 1 * 2^1 + 1 * 2^0). Multibyte numbers are given in little-endian notation and the bytes are seperated by whitesopace charakters, thus the multibyte number 0x1A 0x3C corresponds to the hexadecimal number 0x3C1A or decimal 15386. If the endianness needs to be explicitly stated it will be given as (BE) for big-endian and as (LE) for little-endian.

Notes are written as "//NOTE", tasks are written as "//TODO" and bugs in the original implementation are written as "//BUG", all without the quotation marks. This allows using the editor's search function to quickly jump to these points. An optional colon (:) can be suffixed, so make sure the editor ignores these.

Pseudocode is based on how C works, so if an integer is added to or subtracted from a pointer it means the pointer is advanced by the amount of bytes its value takes up. For instance, if `p` is a pointer to a word and a word is two bytes in size, then `p + 2` is a pointer that points four bytes (two words) further away from `p`. Variables in pseudocode are considered immutable, every arithmetic operation returns a copy. In the above example the pointer `p` would not advance, instead `p + 2` would be a new pointer. To mutate a variable use a verb such as "increment" or "advance".


2D or 3D
--------
Despite its name Wolfenstein 3D is not a true 3D game; the game's data and simulation all happen in a flat 2D space on a strict grid, while the rendering appears to take place in a 3D world to the player. In this document the game world (called "world space") will be treated as if it was actually a three-dimensional space, since that is what the player is experiencing. On the other hand, simulation space, where all the game's actual mechanics are implemented, will be treated like a two-dimensional plane, since that is the way the game works.


Compression algorithms
----------------------
The compression algorithms all assume little-endian multibyte numbers, a byte size of 8 bits and a word size of two bytes.

###RLEW compression###
A variant of RLE (Run Length Encoding) that uses words instead of bytes as the underlying unit. Repeating words are stored as a word triplet `(tag, count, word)` where `tag` is a constant word used to identify the triplet, `count` is how many times to copy the word and `word` is the word to copy. Aside from these triplets there are also uncompressed words that are copied verbatim. Here is the pseudocode:

	Prerequisites: source      = pointer to the start of the compressed input stream
	               destination = pointer to the start of the decompressed output stream
	               tag         = a word used to identify a triplet
	               length      = integer length of the decompressed data
	               Must allocete enough memory to hold the decompressed sequence
	
	Side effects: The pre-allocated memory will be filled with decompressed data
	
	1) Make new pointers: `read` = `start`, `write` = `desination`
	   These pointers will be moved forward while the original pointers remain fixed
	2) While `length` > 0
		2.1) Read `word` pointed at by `read`
		2.2) If `word` is `tag`
			2.2.1) Advance `read` by one word
			2.2.2) Make new integer `count` from word pointed at by `read`
			2.2.3) Advance `read` by one word
			2.2.4) while `count` > 0
				2.2.4.1) Copy word under `read` to `write`
				2.2.4.2) Advance `write` by one word
				2.2.4.3) Decrement `count` and `length` by one
			2.2.5) Advance `read` by one word
		2.3) Else
			2.3.1) Copy word under `read` to `write`
			2.3.2) Advance `read` and `write` by one word
			2.3.3) Decrement `length` by one

What about the word that's identical to `tag`? It will be compressed as `(tag, 0x01 0x00, tag)`, i.e. copy the word `tag` one time. This is actually a threefold increase in data compared to the uncompressed version, but in practice this is a better solution than having special cases.

###Carmack compression###
The underlying idea of this compression method is that certain information patterns are going to be repeated several times. Instead of repeating the pattern each time a reference to previous instances of the pattern is stored; the already uncompressed data is referenced by the still compressed data.

The compressed data consists of uncompressed words, one of two types of pointers, near pointers and far pointers, and exceptions where all four can appear in the same file depending on which is necessary. Near pointers are byte triplets and far pointers are byte quadruples. On top of this ther are special exceptions for words that might be confused for pointers. Remember that all offsets are given in *words* and that a word on the original target architecture was two bytes in size. To get the *byte* offset multiply the word offset by two.

####Near pointers####
Near pointers are a sequence of three bytes (count, tag, offset). The first byte tells us how many words to copy, it is an usingned 8-bit integer. The second byte is the tag and always 0xA7, it is used to identify a near pointer. The third byte is the unsigned 8-bit integer offset relative from the last written word to the word to copy. Take the following example:

	// near pointer
	04 A7 04
	// already decompresssed data so far
	76 0C 00 20 CD AB 9C 01 01 00 CD AB 0F 00 0C 00 CD AB 31 00 01 00 0C 00 0C 00 0C 00 0A 00 CD AB 05 00 ??

The `??` is the current position of the destination pointer; it points at memory that has been allocated but not yet been written to, its content is at this point undefined. The near pointer tells us to copy four words (eight bytes) from four words ago. The resulting output would then be:

	76 0C 00 20 CD AB 9C 01 01 00 CD AB 0F 00 0C 00 CD AB 31 00 01 00 0C 00 0C 00 0C 00 0A 00 CD AB 05 00 0C 00 0A 00 CD AB 05 00 ??

First a copy of the destination pointer (called *copy pointer*) is moved four words back, pointing at the byte `0C`. The byte pointed at by the copy pointer is copied to the value pointed at by the destination pointer and both pointers are incremented. This is repeated eight times, at which point the copy pointer has reached the original position of the destination pointer.

####Far pointers####
The disadvantage of near pointers is that the offset is an 8-bit integer, so it can only reach 255 words back. Far pointers (count tag low_offset high_offset) use a 16-bit offset, so they take up one more byte in memory. The offset is given relative to the start of the decompressed sequence, i.e. the first destination pointer. Aside from the offset they work the same as near pointers, their tag is 0xA8.

####Exception####
Words with a high byte (second byte) of `0xA7` or `0xA8` can be confused for pointers. In compressed form the low byte is replaced by the byte `0x00` and the low bytes value is appened after the high byte. A count of 0 would make no sense for a pointer, so the algoruthm can tell when an exception has occured. Since the low byte comes after the high byte the word is actually stores in big-endian notation and needs to be swapped around when written to the destination.

####Extraction####
To decompress the data we need to know the length of the decompressed data because there is no indication when the end of the compressed sequence is reached; the compressed data is often stored adjacent to other compressed data in the same file. On top of that there is also uncompressed data between near- and far pointers which must be copied verbatim.

Keep count of the bytes or words already written. When using words instead of bytes to keep track make sure you divide the byte count by two. At first the count is 0 and it is incremented every time we write a word or byte. Once the count reaches the size of the decompressed data the extraction is done. After each write increment the count and advance the pointers appropriately. This means the destination pointer is advanced by one byte for every byte written and the source pointer is advanced by three bytes for near pointers and exceptions, four for far pointers, and two for regular words.

During each iteration step read a word. If the word's high byte (second byte) is neither the near- nor the far flag copy the word to the destination. If it's the near flag and the count is not 0x00 step `offset` words back through the decompressed data and copy `count` words from there to the decompressed data. If it's a far pointer and the count is not 0x00 copy `count` words `offset` words from the start of the decompressed data. If the count is zero advance the pointer by one byte and copy the reversed word.

####Pseudocode####
This pseudocode is similar to the original implementation by Id but independent of language or architecture. It operates on words, but that's just one way to do it.

	Constants: zero = 0x00
	           near = 0xA7
	           far  = 0xA8
	
	Prerequisites: source      = pointer to the start of the compressed input stream
	               destination = pointer to the start of the decompressed output stream
	               length      = length of the decompressed data sequence in words
	               Must allocete enough memory to hold the decompressed sequence
	
	Side effects: The pre-allocated memory will be filled with decompressed data
	
	1) Make new pointers: `read` = `start`, `write` = `desination`
	   These pointers will be moved forward while the original pointers remain fixed
	2) While `length` > 0
		2.1) Read the word pointed at by `read`
		2.2) Make new integer `count` the numeric value of its low byte
		2.3) Make new integer `flag` the numeric value of its high byte
		2.4) If `flag` is `near` and `count` is not `zero`
			2.4.1) Advance `read` by one byte
			2.4.2) Read the word under `read`
			2.4.3) Make the new integer `offset` the numeric value of the word's high byte
			2.4.4) Make the new pointer `copy` = `write` - `offset`
			2.4.5) While `count` > 0
				2.4.5.1) Copy word under `copy` to `write`
				2.4.5.2) Advance `copy` and `write` by one word each
				2.4.5.3) Decrement `count` and `length` by one each
		2.5) Else if `flag` is `far` and `count` is not `zero`
			2.5.1) Advance read by one word
			2.5.2) Read the word under `read`
			2.5.3) Make the new integer `offset` the numeric value of the word
			2.5.4) Make the new pointer `copy` = `destination` + `offset`
			2.5.5) While `count` > 0
				2.5.5.1) Copy word under `copy` to `write`
				2.5.5.2) Advance `copy` and `write` by one word each
				2.5.5.3) Decrement `count` and `length` by one each
		2.6) Else if `flag` is `near` or `far` and `count` is `zero`
			2.6.1) Advance `read` by one byte
			2.6.2) Copy word under `read` to `write`
			2.6.3) Swap bytes of word under `write`
			2.6.4) Advance `read` and `write` by one word each
			2.6.5) Decrement `length` by one
		2.7) Else
			2.7.1) Copy word under `read` to `write`
			2.7.2) Advance `read` and `write` by one word each
			2.7.3) Decrement `length` by one

Near- and far pointers are very similar, the only difference is in how the offset is computed and that near pointer have to advance by one byte while far pointers advance by one word.

###Huffman compression###
Id's implementation of the Huffman compression algorithm uses a 255 node large Huffman tree stored as a flat array where each node consist of two words. Node number 255 is always the root node.

Data files
----------
endianness: little-endian, byte-size: 1 byte = 8 bits, word-size: 1 word = 2 bytes = 16 bits

The game assets for WS3D are stored in various files with the same extension, which is depending on the version of the game. For simplicity the file extension will be omitted form here on unless a specific file extension is needed. The assets are distributed as follows:

	Graphics -> VGADICT, VGAHEAD, VGAGRAPH
	Audio    -> AUDIOHED, AUDIOT
	Maps     -> MAPHEAD, GAMEMAPS (or MAPTEMP in some versions?) //NOTE: do such files even exist in official releases?

(//TODO: all of this needs to be properly confirmed)

The header files contain information about the structure of the actual asset files

### File extensions ###

The file extension of the data files depends on the version of the game. They are as follows:

- WL1 Shareware
- WL3 Early three-episode full version
- WL6 Six-episode full version
- WJ1 Japanese shareware?
- WJ6 Japanese full version?
- SOD Spear of Destiny?
- SDM ???
- SD1 ???
- SD2 ???
- SD3 ???

### Graphics ###
There are two types of graphics in the game: *pics* and *sprites*. Pics are rectangular pictures of any size without any transparent holes and used outside the 3D portions of the game. An alternative name is *bitmaps*. Sprites are in-game object graphics using the colur 0x980088 for transparency and are always 64x64 pixels large.

####Pics####
To extract pics three files are needed:

	VGADICT     Huffman-tree for decopressing the pics
	VGAHEAD     Headers describing where to find the pics
	VGAGRAPH    Compressed pics lumped together

The pics are all Huffman-compressed, so first the Huffman tree has to be loaded.

#### VGADICT ####
This file is 1024 bytes large, but the last four bytes are just 0x00 byte padding. Four consequtive bytes each form a Huffman tree node and the node type itself is made of two words, so the file describes 255 individual Huffman nodes (255 * 4 = 1020). Only those 1020 bytes are read and stored verbatim in an array of Huffman-node type of length 255 (size hard coded). As explained above a Huffman-node is a struct holding two words.

#### VGAHEAD ####
This file holds the offsets of the pics and is uncompressed. Each offset is a 32-bit signed number, but it is stored using only three bytes instead of four. The number of offsets is one more than the number of actual chunks; this last offset points to the end of the file. It is necessary because the length of a compressed chunk is not encoded anywhere, it needs to be computed using the starting offset of the next chunk.

#### VGAGRAPH ####
This is the file containing the Huffman-compressed chunks. The number of pics is hard-coded into the executable and cannot be learned from this file as not all chunks are actually pics, some are text or palettes. The first chunk is the *picture table*, an array of widths and heights for each pic. Each array element is a pair of two words, the first being the width and the second being the height.

#### Extracting the pics ####
Pics are stored Huffman-compressed, so first we need to read the Huffman-table. This is straight forward, simply dump the contents of VGADICT into a pre-allocated array. All sizes are hard coded. Next we need to read the pic headers from VGAHEAD.

First we need to know that number of pics used by the game. This can vary depending on which version of the game is played and the number is hard coded into the executable. It can also be computed by getting the size of the VGAHEAD file in bytes and dividing by three since each head is stored as three bytes. Both approaches are valid and there is a proposal below under "Distributions of the game and magic numbers" for using hard-coded numbers in a way that's compatible with multiple versions of the game at runtime.

Using that number allocate space for an array of that many 32-bit integers and fill each one with the corresponding offset value. Beware that the offsets are stored in the file using only three bytes, not four. One exception is the number 0x00FFFFFF or its corressponding byte sequence `FF FF FF` which gets mapped to the offset -1. It does not appear in neither the registered six-episode release nor in the shareware release. I am not sure what the reason is here, but the original release has the following line in the `CA_FarRead` function:

	if (length>0xffffl)
		Quit ("CA_FarRead doesn't support 64K reads yet!");

This seems to be a safety check for technical reasons and since that value does not appear among the offsets anyway I am not certain if it is worth replicating.

Now we need to read the picture table, an array of widths and heights for the individual pics. Open the VGAGRAPH file and jump to the first offset. Now compute the length of this first chunk by taking the offset to the next chunk, substracting the offset of the current chunk and subtracting four. The result is the compressed length of the first chunk, the list of picture dimensions, in bytes. Now allocate enough bytes to hold that sequence and fill it with the first chunk. Allocate enough memory to hold the decompressed picture table and Huffman-expand the first chunk into it.

Now that the preperation work is done we can start extracting the individual pics. So far we have the Huffman tree, an array of offsets, a pic table describing the size of each pic and an open VGAGRAPH file. A chunk is identified using its magic number. Get the offset of the chunk and that of the next chunk using their magic numbers. If the offset of the chunk is -1 abort. We can get the magic number of the next chunk by adding +1 to the magic number of the current chunk. If the offset of the next chunk is -1 keep adding +1 to the magic number until the offset is a proper value. Compute the length of the compressed chunk as the difference in chunk offsets and fill a buffer of that size and type 32-bit signed integer with the data of the chunk.

Now we can expand the data. We need to know the expanded size of the chunk, which can be read from the compressed chunk: the first four bytes are a signed 32-bit integer that tells us the size, so read it and advance the pointer by four bytes. There is an exception if the chumk number is greater or equal to `STARTTILE8` and less than `STARTEXTERNS`; I don't really under stand what that is supposed to represent, but the size is hard coded in that case and the pointer is not advanced. Here it the code in question:

	if (chunk >= STARTTILE8 && chunk < STARTEXTERNS) {
		// expanded sizes of tile8/16/32 are implicit
		#define BLOCK           64
		#define MASKBLOCK       128
		
		if (chunk<STARTTILE8M)          // tile 8s are all in one chunk!
			expanded = BLOCK*NUMTILE8;
		else if (chunk<STARTTILE16)
			expanded = MASKBLOCK*NUMTILE8M;
		else if (chunk<STARTTILE16M)    // all other tiles are one/chunk
			expanded = BLOCK*4;
		else if (chunk<STARTTILE32)
			expanded = MASKBLOCK*4;
		else if (chunk<STARTTILE32M)
			expanded = BLOCK*16;
		else
			expanded = MASKBLOCK*16;
	}

Allocate enough memory for the uncompressed chunk and pass the pointer to the compressed source, decompressed destination, expanded size and Huffman tree to the Huffman decompression routine. The destination will then hold the address of the decompressed pic chunk. All that is left now is interpreting the chunk as an image.

#### Interpreting pics ####

####Sprites####

### Audio ###

### Levels & Maps ###
Levels are laid out on a 64 x 64 tile-based square map. This size is not hard-coded into the game, so one should not make assumptions about the levels's size, instead the size should be read from the map file. Although there are no official levels of any other size an engine or interpreter should be able to support custom-made maps of different size. Each level in the game actually consists for three maps overlaying each other:

- **Architecture:** The first map contains information about the level's architecture, i.e. walls, doors and floors.
- **Objects:** The second map contains the level's objects, i.e. enemies, decorations and pick-ups.
- **Logic:** The third map contains the level's logic data, such as waypoints.

These three individual maps together form the level the player will be playing. Usually when speaking about maps one means the entire level, but here we will maintain this distinction to avoid confusion or ambiguity.

Each of the tiles in a level describes a three-dimensional cube in the game world with 64 units in length to both sides and 64 units in height (i.e. a cube in 3D world space).

#### MAPHEAD ####
The file starts with the signature 16-bit integer 0xABCD (represented as 0xCD 0xAB bytes in the file). This signature appears always to be the same, but we should not make any assumptions; it is used as the signature for the RLEW compression algorithm. The file is described by the structure `mapfiletype` in the original source code.

Next are exactly 100 32-bit (4 Byte) signed integer values containing the header offsets of the actual levels, that amount is hardcoded into the source. Not all of these 32-bit numbers have meaningful values, only the first n do, where n is the total amount of levels in the game, i.e. 10 in the shareware version and 30 or 60 in the full version. The remaining numbers are all padding with 0x00000000 as their value. This means the level offsets are stored in a 0-terminated 4-byte array with a fixed length of 100.

The last remaining byte always appears to be be 0x00 and it's called the `tileinfo` in the original source code and is declared as an array of unspecified size of type `byte`. The type `byte`is a typedef for `unsigned char` and equal to an 8-bit integer on the target architecture of Wolfenstein 3D's original code. It appears to be a leftover from the map format of previous Id Software games that did use it.

Note that there is no information in this file as to how many levels there are in the game. This information would have to be calculated from the file's size itself. To compute that number one would have to step through the list of header offsets until reaching the first offset that's 0x00000000 (start of the padding). The number of steps is equal to the number of levels.

#### GAMEMAPS ####
This file contains the actual information about the levels and their individual maps. A level is made from a *level header*, which describes where to find the level's maps, their compressed sizes, the size of the level and finally the name of the level.

The header can be found using the offset from the MAPHEAD file as an absolute value, i.e. relative to the start of the file. From there on the header is stored as an uncompressed sequence of raw information.

The first three values are 32-bit signed integer values each. The first one is holding the offset to the level's architecture map, the next value is the offset to the level's object map and the third value is the offset to the level's logic map. All values are absolute offsets from the beginning of the file, not relative offsets from the header or relative to each other.

The next three values are unsigned 16-bit integer values describing the Carmack-compressed length in bytes of each map; this is important because the maps are lumped together adjacent to each other with no seperator. Their order is again first architecture, then objects and then logic.

Next are two unsigned 16-bit integers describing the width and height of the level, in that order. The size appears to always be 64 x 64, but since it's not hardcoded it should not be assumed.

Finally 16 characters, 8-bit ASCII each, form the level's name. In the original implementation the characters are stored in an array of type `char` with unspecified size. This is the standard way of storing ASCII strings in C, but the string needs to be terminated with `\0` (the null character). In the file any remaining bytes are filled with `\0`, but in the code there is nothing to ensure that the string is indeed properly terminated, leaving a possibility for an error to happen.

#### Extracting the maps ###
Maps are compressed using the RLEW compression and then compressed on top of that using Carmack compression. To decompress them one has to first Carmack-decompress the data and then RLEW-decompress it. For Carmack compression one can find the decompressed length encoded into the compressed map as the fist word, it is given in bytes. This means the pointer to the compressed sequence must be advanced by one before starting the decompression. For some reason the pointer to the Carmack-decompressed but still RLEW-compressed sequence must be advanced by one word as well; could be a leftover from a previouse map format. The size of the uncomperessed RLEW data is hardcoded as `64*64*2` bytes or 4096 words. Since the size is also stored in the map format it might be a better idea to use that value instead and allow levels of different size for mods. The RLEW tag can be found in the MAPHEAD file as described above.


Known bugs and limitations
--------------------------

1. A map needs at least 1 enemy, 1 piece of treasure and 1 secret door, or else the game will crash. This is the result of the game trying to calculate the percentage the player has picked up and ending up dividing by zero.

### Distributions of the game and magic numbers ###
Different versions of the game assign different key numbers to the graphics. Each graphic can be identified using an integer "key" magic number and that number depended on what the graphics artists produced. This means the programmers would have had to keep a list of key numbers and always change their code when the graphics changed. Instead the software used by the artists produced alongside the data files also a header file that mapped each number to a macro, so the developers could simply use the macro and the generated header would map the macro to the correct number.

The problem with this is that each build was only suitable for a particular distribution of the game, like shareware, registered, Japanese or Spear of Destiny. The simple solution is to use `#ifdef` directives and set the version at compile time, which is an acceptable solution when building for a particular release, but ill-suited for a source port that needs to be as compatible as possible. I propose the following solution that moves the version detection from compile-time to run-time.

There will be one global header file that has a universal mapping that assigns a number to any image that might exist in any distribution. It is not necessary for it to be compatible to any existing distribution. The programmers will use these global macros then. For each distribution there will be a mapping that maps the universal macro to the distribution's corresponding key number. At run-time when the program starts determine the distribution and assign a global mapping variable to be the mapping for that distribution. The mapping could for example be done using an array where the universal macro is the index and the distribution's key the value. Trying to access an image that does not exist in that particular distribution could be mapped to an invalid number such as -1.
