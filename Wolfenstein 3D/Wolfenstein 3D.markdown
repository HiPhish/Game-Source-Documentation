Wolfenstein 3D code design document
===================================
Analysing and documenting the source code workings of Wolfenstein 3D for DOS. (This document should be read in an editor with a monospaced font and line wrapping enabled. This document follows the Markdown syntax.)


Table of contents
-----------------

	- Wolfenstein 3D code design document
	| |- Conventions and nomenclature
	| |- 2D or 3DGame versions and file extensions
	| |- Game versions and file extensions
	|
	- Part I - File formats
	| |- Compression algorithms
	| |  |- RLEW compression
	| |  |- Carmack compression
	| |  |- Huffman compression
	| |
	| |- Data files
	|    |- Graphics
	|    |  |- Pics
	|    |  |- Sprites
	|    |  |- Textures
	|    |
	|    |- Audio
	|    |  |- Sound effects
	|    |  |- Music
	|    |
	|    |- Maps
	|       |- MAPHEAD
	|       |- GAMEMAPS
	|       |- Extracting the maps
	|
	- Part II - Game rules
	| |- Mathematics
	| |- Levels
	| |- Actors / Entities
	|    |- The actor structure
	|    |- Actor states
	|
	- Appendix
	  |- Tables for actors
	  |  |- List of actor classes and starting hit points
	  |  |- List of state classes
	  |  |- Table of actor states 
	  |
	  |- Known bugs and limitations
	  |- References

--------------------------------------------------------------------------------

Conventions and nomenclature
----------------------------
Decimal integer numbers are given as regular numbers or rarely suffixed with "d", binary numbers are suffixed with "b" or prefixed with "0b", octal numbers with "o" and hexadecimal numbers are either suffixed with "h" or prefixed with "0x" or "#". Example:

	13 = 13d = 1011b = 15o = Dh = 0xD

All continuous numbers are written in the standard big-endian notation used in the English language. This means the left-most digit is the most significant one and a number like 1011b is computed as (1 * 2^3 + 0 * 2^2 + 1 * 2^1 + 1 * 2^0). Multibyte numbers are given in little-endian notation and the bytes are seperated by whitesopace charakters, thus the multibyte number 0x1A 0x3C corresponds to the hexadecimal number 0x3C1A or decimal 15386. If the endianness needs to be explicitly stated it will be given as (BE) for big-endian and as (LE) for little-endian.

Notes are written as "//NOTE", tasks are written as "//TODO" and bugs in the original implementation are written as "//BUG", all without the quotation marks. This allows using the editor's search function to quickly jump to these points. An optional colon (:) can be suffixed, so make sure the editor ignores these.

Pseudocode is based on how C works, so if an integer is added to or subtracted from a pointer it means the pointer is advanced by the amount of bytes its value takes up. For instance, if `p` is a pointer to a word and a word is two bytes in size, then `p + 2` is a pointer that points four bytes (two words) further away from `p`. Variables in pseudocode are considered immutable, every arithmetic operation returns a copy. In the above example the pointer `p` would not advance, instead `p + 2` would be a new pointer. To mutate a variable use a verb such as "increment" or "advance".

Pointers and arrays are used more or less interchangably in the pseudocode as well. Usually when a pointer is described as a "sequence" of something it can be an array as well. Similarly, the square bracked notation `pointer[i]` means "the value of `pointer + i`". Whether the actual implementation uses pointers, arrays, vectors, linked lists of whatever is irrelevant.


2D or 3D
--------
Despite its name Wolfenstein 3D is not a true 3D game; the game's data and simulation all happen in a flat 2D space on a strict grid, while the rendering appears to take place in a 3D world to the player. In this document the game world (called "world space") will be treated as if it was actually a three-dimensional space, since that is what the player is experiencing. On the other hand, simulation space, where all the game's actual mechanics are implemented, will be treated like a two-dimensional plane, since that is the way the game works.

Game versions and file extensions
---------------------------------

The file extension of the data files depends on the version of the game. They are as follows:

| Extension | Version                          |
|-----------|----------------------------------|
| WL1       | Shareware                        |
| WL3       | Early three-episode full version |
| WL6       | Six-episode full version         |
| WJ1       | Japanese shareware?              |
| WJ6       | Japanese full version?           |
| SOD       | Spear of Destiny?                |
| SDM       | ???                              |
| SD1       | SoD M2: Return to Danger?        |
| SD2       | SoD M3: Ultimate challenge       |
| SD3       | ???                              |

--------------------------------------------------------------------------------

Part I - File Formats
=====================

Compression algorithms
----------------------
The compression algorithms all assume little-endian multibyte numbers, a byte size of 8 bits and a word size of two bytes.

### RLEW compression ###
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

### Carmack compression ###
The underlying idea of this compression method is that certain information patterns are going to be repeated several times. Instead of repeating the pattern each time a reference to previous instances of the pattern is stored; the already uncompressed data is referenced by the still compressed data.

The compressed data consists of uncompressed words, one of two types of pointers, near pointers and far pointers, and exceptions where all four can appear in the same file depending on which is necessary. Near pointers are byte triplets and far pointers are byte quadruples. On top of this ther are special exceptions for words that might be confused for pointers. Remember that all offsets are given in *words* and that a word on the original target architecture was two bytes in size. To get the *byte* offset multiply the word offset by two.

#### Near pointers ####
Near pointers are a sequence of three bytes (count, tag, offset). The first byte tells us how many words to copy, it is an usingned 8-bit integer. The second byte is the tag and always 0xA7, it is used to identify a near pointer. The third byte is the unsigned 8-bit integer offset relative from the last written word to the word to copy. Take the following example:

	// near pointer
	04 A7 04
	// already decompresssed data so far
	76 0C 00 20 CD AB 9C 01 01 00 CD AB 0F 00 0C 00 CD AB 31 00 01 00 0C 00 0C 00 0C 00 0A 00 CD AB 05 00 ??

The `??` is the current position of the destination pointer; it points at memory that has been allocated but not yet been written to, its content is at this point undefined. The near pointer tells us to copy four words (eight bytes) from four words ago. The resulting output would then be:

	76 0C 00 20 CD AB 9C 01 01 00 CD AB 0F 00 0C 00 CD AB 31 00 01 00 0C 00 0C 00 0C 00 0A 00 CD AB 05 00 0C 00 0A 00 CD AB 05 00 ??

First a copy of the destination pointer (called *copy pointer*) is moved four words back, pointing at the byte `0C`. The byte pointed at by the copy pointer is copied to the value pointed at by the destination pointer and both pointers are incremented. This is repeated eight times, at which point the copy pointer has reached the original position of the destination pointer.

#### Far pointers ####
The disadvantage of near pointers is that the offset is an 8-bit integer, so it can only reach 255 words back. Far pointers (count tag low_offset high_offset) use a 16-bit offset, so they take up one more byte in memory. The offset is given relative to the start of the decompressed sequence, i.e. the first destination pointer. Aside from the offset they work the same as near pointers, their tag is 0xA8.

#### Exception ####
Words with a high byte (second byte) of `0xA7` or `0xA8` can be confused for pointers. In compressed form the low byte is replaced by the byte `0x00` and the low bytes value is appened after the high byte. A count of 0 would make no sense for a pointer, so the algoruthm can tell when an exception has occured. Since the low byte comes after the high byte the word is actually stores in big-endian notation and needs to be swapped around when written to the destination.

#### Extraction ####
To decompress the data we need to know the length of the decompressed data because there is no indication when the end of the compressed sequence is reached; the compressed data is often stored adjacent to other compressed data in the same file. On top of that there is also uncompressed data between near- and far pointers which must be copied verbatim.

Keep count of the bytes or words already written. When using words instead of bytes to keep track make sure you divide the byte count by two. At first the count is 0 and it is incremented every time we write a word or byte. Once the count reaches the size of the decompressed data the extraction is done. After each write increment the count and advance the pointers appropriately. This means the destination pointer is advanced by one byte for every byte written and the source pointer is advanced by three bytes for near pointers and exceptions, four for far pointers, and two for regular words.

During each iteration step read a word. If the word's high byte (second byte) is neither the near- nor the far flag copy the word to the destination. If it's the near flag and the count is not 0x00 step `offset` words back through the decompressed data and copy `count` words from there to the decompressed data. If it's a far pointer and the count is not 0x00 copy `count` words `offset` words from the start of the decompressed data. If the count is zero advance the pointer by one byte and copy the reversed word.

#### Pseudocode ####
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

### Huffman compression ###
Id's implementation of the Huffman compression algorithm uses a 255 node large Huffman tree stored as a flat array where each node consist of two words, and node number 255 (inde 254) is always the root node. Here is how the nodes work: a byte called the *node value* is being kept track of, it is initially 254, the array position of the root node of the tree. From there the input of the compressed stream is being read bit-wise, if the bit is `0` the node value is set to the node's first word, otherwise to the node's second word. If the node value is less than 256 (i.e. within the value range of a byte) the node value is written as a byte and the node pointer is reset back to the root node. Otherwise, if the node value is eaqual to or greater than 256 the node pointer is set to the node at array index (node value - 256).

#### Pseudocode ####
Since the input cannot be read bit-wise it has to be read one byte at a time and then the input byte is being examined using a masking byte. This byte starts out as 0x01 and is bitewise ANDed with the input byte to decide which path down the tree to take. Afterwards the 1-bit of the masking byte is left-shifted by one to be able to examine the next input-bit. Once the mask byte reached 0x80 the masking bit is all the way to the left, so we need to reset it back to 0x01 and read the next input byte.

	Constants: root = 254
	
	Prerequisites: source       = pointer to the start of the compressed input stream as bytes
	               destination  = pointer to the start of the decompressed output stream as bytes
	               length       = length of the decompressed data sequence in words
	               huffman_tree = array of Huffman-tree nodes for decompression
	               Must allocete enough memory to hold the decompressed sequence
	
	Data structures: struct huffman_node {word word_0, word_1} : a structure holding two words
	
	Side effects: The pre-allocated memory will be filled with decompressed data
	
	1) Make new pointer `node` of type `huffman_node` and set it to `huffman_tree[root]`
	   Make new pointers `read` and `write` and set them to `source` and `destination` respectively
	   Make new byte `mask` = 0x01 and `input`, set input to value of `read`, advance `read`
	   Make new word `node_value`
	2) Repeat indefinitely
		2.1) If (`input` & `mask`) == 0x00
			2.1.1) `node_value` = `node`->`word_0`
		2.2) Else
			2.2.1) `node_value` = `node`->`word_1`
		2.3) If `mask`== 0x80, i.e. the masking bit is all the way to the right
			2.3.1) Set `input` to value pointed at by read, advance read
			2.3.2) Set `mask` back to 0x01
		2.4) Else
			2.4.1) Bit-shift `mask` by one bit to the left
		2.5) If `node_value` < 256 (hex 0xFF)
			2.5.1) Write the value of `node_vale` as a byte to `write`, advance `write`
			2.5.2) Reset `node_pointer` back to `huffman_tree`[`root`]
			2.5.3) If the end of the output stream has been reached break out of the loop
		2.6) Else
			2.6.1) `node_pointer` = `huffman_tree`[`node_value` - 256]

Data files
----------
endianness: little-endian, byte-size: 1 byte = 8 bits, word-size: 1 word = 2 bytes = 16 bits

The game assets for WS3D are stored in various files with the same extension, which is depending on the version of the game. For simplicity the file extension will be omitted form here on unless a specific file extension is needed. The assets are distributed as follows:

	Graphics -> VGADICT, VGAHEAD, VGAGRAPH
	Audio    -> AUDIOHED, AUDIOT
	Maps     -> MAPHEAD, GAMEMAPS (or MAPTEMP in some versions?) //NOTE: do such files even exist in official releases?

(//TODO: all of this needs to be properly confirmed)

The header files contain information about the structure of the actual asset files

### Graphics ###
There are two types of graphics in the game: *pics* and *sprites*. Pics are rectangular pictures of any size without any transparent holes and used outside the 3D portions of the game. An alternative name is *bitmaps*. Sprites are in-game object graphics using the colur 0x980088 for transparency and are always 64x64 pixels large.

#### Pics ####
To extract pics three files are needed:

	VGADICT     Huffman-tree for decopressing the pics
	VGAHEAD     Headers describing where to find the pics
	VGAGRAPH    Compressed pics lumped together

The pics are all Huffman-compressed, so first the Huffman tree has to be loaded.

##### VGADICT #####
This file is 1024 bytes large, but the last four bytes are just 0x00 byte padding. Four consequtive bytes each form a Huffman tree node and the node type itself is made of two words, so the file describes 255 individual Huffman nodes (255 * 4 = 1020). Only those 1020 bytes are read and stored verbatim in an array of Huffman-node type of length 255 (size hard coded). As explained above a Huffman-node is a struct holding two words.

##### VGAHEAD #####
This file holds the offsets of the pics and is uncompressed. Each offset is a 32-bit signed number, but it is stored using only three bytes instead of four. The number of offsets is one more than the number of actual chunks; this last offset points to the end of the file. It is necessary because the length of a compressed chunk is not encoded anywhere, it needs to be computed using the starting offset of the next chunk.

##### VGAGRAPH #####
This is the file containing the Huffman-compressed chunks. The number of pics is hard-coded into the executable and cannot be learned from this file as not all chunks are actually pics, some are text or palettes. The first chunk is the *picture table*, an array of widths and heights for each pic. Each array element is a pair of two words, the first being the width and the second being the height.

##### Extracting the pics #####
Pics are stored Huffman-compressed, so first we need to read the Huffman-table. This is straight forward, simply dump the contents of VGADICT into a pre-allocated array. All sizes are hard coded. Next we need to read the pic headers from VGAHEAD.

First we need to know that number of pics used by the game. This can vary depending on which version of the game is played and the number is hard coded into the executable. It can also be computed by getting the size of the VGAHEAD file in bytes and dividing by three since each head is stored as three bytes. Both approaches are valid and there is a proposal below under "Distributions of the game and magic numbers" for using hard-coded numbers in a way that's compatible with multiple versions of the game at runtime.

Using that number allocate space for an array of that many 32-bit integers and fill each one with the corresponding offset value. Beware that the offsets are stored in the file using only three bytes, not four. One exception is the number 0x00FFFFFF or its corressponding byte sequence `FF FF FF` which gets mapped to the offset -1. It does not appear in neither the registered six-episode release nor in the shareware release. I am not sure what the reason is here, but the original release has the following line in the `CA_FarRead` function:

	if (length>0xffffl)
		Quit ("CA_FarRead doesn't support 64K reads yet!");

This seems to be a safety check for technical reasons and since that value does not appear among the offsets anyway I am not certain if it is worth replicating.

Now we need to read the picture table, an array of widths and heights for the individual pics. Open the VGAGRAPH file and jump to the first offset. We can read the expanded length of the chunk in bytes as a signed 32 bit integer from the first four bytes. Now compute the compressed length of this first chunk in bytes by taking the offset to the next chunk, substracting the offset of the current chunk and subtracting four (the extpanded length). Now allocate enough bytes to hold that sequence and fill it with the first chunk minus the first four bytes. Allocate enough memory to hold the decompressed picture table and Huffman-expand the first chunk into it.

Now that the preperation work is done we can start extracting the individual pics. So far we have the Huffman tree, an array of offsets, a pic table describing the size of each pic and an open VGAGRAPH file. A chunk is identified using its magic number. Get the offset of the chunk and that of the next chunk using their magic numbers. If the offset of the chunk is -1 abort. We can get the magic number of the next chunk by adding +1 to the magic number of the current chunk. If the offset of the next chunk is -1 keep adding +1 to the magic number until the offset is a proper value. Compute the length of the compressed chunk as the difference in chunk offsets and fill a buffer of that size and type 32-bit signed integer with the data of the chunk.

Now we can expand the data. We need to know the expanded size of the chunk, which can be read from the compressed chunk: the first four bytes are a signed 32-bit integer that tells us the size, so read it and advance the pointer by four bytes. There is an exception if the chumk number is greater or equal to `STARTTILE8` and less than `STARTEXTERNS`; I don't really understand what that is supposed to represent, but the size is hard coded in that case and the pointer is not advanced. Here it the code in question:

	if (chunk >= STARTTILE8 && chunk < STARTEXTERNS) {
		// expanded sizes of tile8/16/32 are implicit
		#define BLOCK        64
		#define MASKBLOCK    128
		
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

##### Interpreting pics #####
Uncompressed pics are stored as sequences of bytes. A byte's unsigned integer value can range from 0 to 255, which is exactly how many colours the VGA standard supports. Each byte stands for a colour index of a pixel that can be mapped to a colour value using a palette. The palette depends on the game and can be loaded from an external file or be hard-coded, it maps the indices to whatever format the target API uses, such as RGBA. In order to display the image as a two-dimensional surface we also need the width and height from the picture table above.

Given the size of the picture and a palette we can then assemble the image the following way:

	rgb_pixel[i + j*width] = palette[vga_pixel[(j*(width>>2)+(i>>2))+(i&3)*(width>>2)*height]] 

Here `rgb_pixel` is a linear array of output pixels starting in the top-left corner and growing width-first, height-second. `palette` is an array that maps a colpur index to an RGB colour value. `vga_pixel` is the array of picture pixels. The variables `i` and `j` stand for the current width and height while building the output image. The operators `>>` and `&` are bitwise right-shift and bitwise `AND` respectively.

I don't understand how or why pictures need to be "woven" in such a way, I assume it has to do with the way that the VGA standard works. Trying to order the pixels linearly instead of weaving them results in 4x4 tiles of down-scaled versions of the picture; the original picture can still be recognised. The original code does mention four "layers" when it is about to send the picture to memory.

#### Sprites ####
Sprites are stored in the file VSWAP, together with textures and sound effects, there are no other files involved. Each sprite is 64x64 pixels large. They are drawn column-wise and since there is a lot of empty columns left and right of the visible picture. Only the columns between and including the outer-most non-empty columns are given. Each column is described via a variable-length list of drawing instructions, each instruction being six bytes in size.

##### VSWAP #####
The first six bytes of this file is the header consisting of three signed 16-bit integers. The first integer is the total number of chunks in the file, regardless of type. The second integer is the starting index of the sprite chunks relative to the beginning of the file. The third integer is the starting index of the sound effects. I will only be focusing on the sprites here.

Next up is a list of all chunk offsets. They are stored as unsigned 32-bit integers and their amount is the number of chunks. It is followed by a second list, the list of chunk lengths, same amount but stored as words. To decide whether a chunk is a texture, a sprite or a sound one has to use the chunk's index and compare it to the number of sprite- and sound chunks and their starting index. If you want to read a sprite or a sound you have to add the starting index to the magic number, for example if the sprite index is 35 and we want to read sprite 8 we have to read chunk 43.

Once we have a sprite's offset and length we can read it. The sprite has its own header consisting of two words followed by an array of up to 64 words. The first word is the index of the left-most non-empty column, the second word is the index of the right-most column. The array is of variable length and contains the offsets to the head of the drawing instruction list of each column; the first array element is the offset to the drawing instruction list of the left-most non-empty column, the last array element is the offset for the right-most non-empty column, and evey element in between belongs to the column after the previous one. All these offsets are relative to the beginning of the sprite, not the VSWAP file.

The number of instruction offsets can be computed as follows: `last_column - first_column + 1`. The index of the beginning of the pixel data within the sprite can thus be found as follows:

	(last_column - first_column + 1 + 2) * sizeof(word)

Here is a schematic of a sprite chunk:
	
	W- first_column
	|
	W- last_column
	|
	W- offset[0] -> |W|W|W| ... |W|W|W|
	:
	W- offset[n] -> |W|W|W| ... |W|W|W|
	|
	B- data
	:
	B- data

A `W` means `word`, a `B` means `byte`, a `- ` means "is" and a `->` means "points to" or "is an offset to", offsets are relative to the beginnig of the chunk. The data stands to any remaing data that's in the sprite, regardless of what it represents. It is given in bytes, because that's how the pixels will be read, but the column instructions are three *words*, so take care to read three words or six bytes, not three bytes.

To fill the image with pixels we fill the entire image with transparency (byte `0xFF`). Next we iterate over the non-empty columns. Here the variable `x` will refer to the index of the current column, it gives us the horizontal position of the pixel. The vertical position is derived from the drawing instructions: the first word divided by two is the lower starting point of the pixel sequence, the third word is the upper end point of the sequence (columns are drawn from bottom to top). If the first word is 0x0000 it means the end of the column has been reached and we can advance `x` to the next one. The middle word is used to reference which pixels to use, but oddly enough it is not necessary.

All that's missing now is how which pixels to draw onto the sprite. Sprites use a sort of RLE-compression: in the compressed sprite data each byte after the instruction offsets is a pixel sequence and the n-th sequence belongs to the n-th instruction. The extents of the instruction tell us how many pixels from that sequence to draw. After an instruction has been executed move on to the next pixel. here is the pseudocode:

	constants: transparency = 0xFF
	
	prerequsites: chunk       = pointer to the compressed chunk as a byte sequence
	              first_colum = index of the first column (within range [0, 63), less than last_column)
	              last_colum  = index of the last column (within range (0, 63], greater than first_column)
	              offsets     = offsets of the column drawing instructions
	              i           = (last_column - first_column + 1 + 2) * sizeof(word)
	              Must allocate enough space to hold decompressed sprite (64*64 bytes)
	
	1) Fill entire sprite with the colour for transparency
	2) Make pointer to word `column_offset_reader` and set it to the first column instruction offset
	3) For (word `column` = `first_column`, while `column` <= `last_column`, iterate ++`column`)
		3.1) Make pointer to word `drawing_instruction` and set to `chunk` + value of `column_offset_reader` (as word)
		3.2) Make integer `idx` = 0
		3.3) While `drawing_instruction`[`idx`] != 0x0000
		    3.3.1) For (word row = `drawing_instruction`[`idx`+2] / 2, while row < `drawing_instruction`[`idx`] / 2, iterate ++row)
		        3.3.1.1) `result`[`column` + (63 - `row`) * 64] = `chunk`[`i`]
		        3.3.1.2) ++`i`
		    3.3.2) `idx` += 3
		3.4) Advance `column_offset_reader` by one word

Now about the second word of the instruction; rather than using the above method to get the pixel sequence it is possible to use that word. Use the numeric value of the word plus the current row as the offset from the beginning of the compressed chunk. As far as I can tell both ways yield the same result, so I don't know which one to prefer. If in doubt go with this one though, just in case that there is a weird exception somewhere out there. Here is the modified pseudocode from above:

	1) ...
	1) ...
	3) ...
		3.1) ...
		3.2) ...
		3.3) ...
			3.3.1) ...
				3.3.1.1) `result`[`column` + (63 - `row`) * 64] = `chunk`[`drawing_instruction`[`idx`+1] + row]
			3.3.2) ...
		3.4) ...

We don't need the variable `i` anymore, and so we don't increment it either.

##### Interpreting sprites #####
Sprites use the same palette as bitmap pictures, but the order in which pixels are stored is different. If you have been following the above instructions the sprite will be flipped horizontally, i.e. upside-down. This means the first row in the raw byte data is the last row in the RGB data, the second row is the second-to-last and so on. Columns are not affected.

#### Textures ####
Textures are simple since they are not compressed. Just like sprites they are always 64x64 pixels large, but they have no holes. They are also stored in the VSWAP file, but their type has no offset, the magic number of a texture is the number of its chunk. To read the texture simply read 4096 bytes from the chunk verbatim. That fixed number can be replaced by the chunk length as discussed above for sprites.

Textures use the same palette as bitmap pictures and sprites as well, but the order of their pixels is different. The entire image is transposed, meaning that the row and column of each pixel needs to be swapped, like a transposed matrix. Or in other words, Wolfenstein 3D drew the textures column-first, row-second.

### Audio ###
Audio is divided into two categories: sound effect and music tracks and they share the same files. There is a head file called *AUDIOHED* that contains the offsets to the the individual chunks as signed 32-bit integers and the chunks are stored uncompressed in the *AUDIOT* file.

##### AUDIOHED #####
There are three types of sound effects: PC speaker, AdLib sound and digitised sound. Every sound effect exists in every format and they are stored in the same order, so the magic number of a sound effect needs to be mapped to the appropriate chunk. Given the number of sound effects, which is hard-coded, we can compute the starting offsets of a format the following way:

	start_pc    = 0 * number_of_sounds
	start_adlib = 1 * number_of_sounds
	start_digi  = 2 * number_of_sounds
	start_music = 3 * number_of_sounds

To get the AdLib version of sound `n` we can thus compute its index as `1 * number_of_sounds + n`. We can also see that the music chunks follow the sound effect chunks, and their amount is also hard-coded. We can thus compute the total number of chunk offsets as follows:

	number_of_offsets = start_music + number_of_tracks + 1

Where does that extra `1` come from? That's the offset to an imaginary chunk one past the last chunk. It does not exist, but it is necessary for computing the length of the last chunk. Computing the length of a chunk is done using the offset of the next chunk; for the i-th chunk that would be:

	size[i] = offset[i+1] - offset[i]

It is possible that the size of some chunks is 0, in this case the chunk can be seen as non-existent and should be skipped. In fact, all the digitised sound effects are like this, they are actually stored in the *VSWAP* file instead, right afer the sprite chunks.

##### AUDIOT #####
This file is a container for various other files, stored as uncompressed chunks all lumped together. To find a particular chunk use its offset and size gotten from the *AUDIOHED* file. What to do with that chunk varies on a type-by-type basis. There are also tags of the form `!ID!` (`0x21 0x49 0x44 0x21`) the the end of each file format group, but they are skipped by the offsets anyway.

The AdLib sound effects and the music are stored in a format that has been specifically designed for AdLib sound cards, so unlike the other data it cannot be simply converted to wave data. One would have to emulate the AdLib hardware, at least the necessary parts, or use a library.

#### Sound effects ####
As explained above there are three different types of sound effects and they are stored ordered by format first and magic number second. Digitised sound is an exception though: MUSE, the program used by Id, offered that format but never supported it. The data structures are all there, but they are never used and the chunks in the AUDIOT file all the length 0. They are stored in another file instead.

##### PC speaker #####
PC speaker sound effects are a form of *inverse frequency sound format* where the data bytes represent the inverse of the frequency to play. Here is how the file is composed: the first four bytes are an unsigned 32-bit integer giving the length of the sound data, it should be the size of the chunk minus 7. It is followed by two bytes of unsigned 16-bit integer giving the priority of the sound effect. Since in the original engine only one sound could play at a time a sound will interrupt any sound of lower or equal priority. Next up is the sequence of data bytes of the length encoded in the first four bytes. Finally one single byte is used to terminate the file, it is usually (always?) 0x00. The file has therefore 7 bytes of non-sound data (length, priority and terminator). There is no file name encoded, so the file can only be accessed using the magic number of the sound effect.

Each byte (unsigned 8-bit integer) of the audio data sequence represents a certain sound frequency measured in *Hz*. The frequency can be computed this way:
	
	frequency = 1193181 / (value * 60)    // for value != 0
	          = 0                         // for value == 0
	
The number `1193181` has the hexadecimal value `0x1234DD`. The refresh rate of the speaker is 140 Hz, so each instruction lasts (1/140) seconds. Also keep in mind that multiplying a byte value by 60 can exceed the range of an 8-bit integer, so the computation has to be done at least using 16 bits.

| Data type    | Name       | Description                            |
|--------------|------------|----------------------------------------|
| uint_32      | length     | Length of sound data, chunk length - 7 |
| uint_16      | priority   | Highter priority wins                  |
| byte[length] | data       | Actual audio data                      |
| uint_8       | terminator | Unused by the game                     |

###### Interpreting the data ######
Aside from the raw audio data there is no playback information stored in the file, everything is hard-coded. Since the PC speaker was not able to play different tones many developers used a trick called *pulse-width modulation* to create the illusion. The frequency perceived by the listener is created by precisely controlling short bursts of audio pulses. Explaining the mathematical properties would be beyond the scope of this document, so I'll refer instead to its [Wikipedia article](http://en.wikipedia.org/wiki/Pulse-width_modulation).

Each byte tells us how long the the phase needs so be. First we read a byte and muliply its numeric value by 60 (hard-coded number). This lets us compute the length of the phase:

	tone         = input_byte * 60
	phase_length = sample_rate * (tone / 1193181) * 1/2

The *sample rate* depends on how precisely we want to sample the data. Higher numbers are more precise, but take up more space. We also need to make sure the sample rate matches the sample rate of our playback, i.e. it is the number of samples played per second. A value of 40,000 is adequate.

The formula works as follows: looking at the second formula we compute the inverse of the frequency we want to simulate. This means a higher frequency will have a shorter duration than a lower one. This inverse frequency is multiplied by the sample rate; frequencies are measured in Hz, which is just another way of writing *1/s*, i.e. one per second of something, so an inverse frequency is a duration, measured in seconds. The sample rate is measured in *samples/second* and by multiplying it with the duration we get the number of samples to generate. Finally we divide by two because we need to flip back-and forth between high and low volume at the half-point mark.

Now it's time to write the sample bytes. How many samples should be written per byte depends on the selected sample rate as well as the original playback rate of 140Hz:

	samples_per_byte = sample_rate / 140

For each byte written we also keep track of the "ticks": each written byte increments the counter, and if the ticks have reached the phase length we flip the sign and reset the counter. A tone of *0* interrupts everything, it writes the neutral sound (128) and keeps the tick counter at 0. The byte written is 128 plus the volume level of the simulated speaker. This level can be chose arbitrarily, as long as it's less or equal to 127.

Here is the pseudocode:

	Constants: base_timer = 1193181
	           pcs_rate   =     140 (playback rate of PC speaker)
	           volume     =      20 (arbitrarily chosen, must be <= 127)
	
	Prerequisites: source      = pointer to the start of the input stream as bytes.
	               destination = pointer to the start of the decompressed output stream as bytes
	               pcs_length  = length of the decompressed data sequence in words
	               sample_rate = how many samples to play back per second
	
	Side effects: The destination buffer will be allocated and filled with data
	
	1) Make new variable `samples_per_byte` = `sample_rate` / `pcs_rate`
	2) Make new variable `wav_length` = pcs_length * samples_per_byte * sizeof(byte)
	3) Allocate memory to `destination` of length `pcs_length` * `samples_of_bytes` *sizeof(byte)
	4) Make new pointers `read` and `write` and set them to `source` and `destination` respectively
	5) Make new signed integer variable `sign` = -1
	6) Make new unsigned integer variable `phase_tick` = 0
	7) While pcs_length > 0
		7.1) Make new variable `tone` = (value of `read`) * 60, advance read one byte
		7.2) Make new variable `phase_length` = sample_rate * (`tone` / `base_timer`) * 1/2
		7.3) For (int i = 0, while i < samples_per_byte, iterate ++i)
			7.3.1) If tone != 0
				7.3.1.1) Write (128 + `sign` * `volume`) to `write`, advance `write`
				7.3.1.2) If phase_tick >= phase_length
					7.3.1.2.1) `sign` *= -1
					7.3.1.2.2) `phase_tick` = 0
				7.3.1.3) ++phase_tick
			7.3.2) Else
				7.3.2.1) phase_tick = 0
				7.3.2.1) Write 128 to `write`, advance `write`
		7.4) --pcs_length

Bytes are in this document equivalent to unsigned 8-bit integers, so it might look conflicting that we use a signed integer and use it for multiplication. However, since the neutral sound is 128, the middle of the 8-bit value range, it doesn't matter in C. For other languages this might not necessarily hold true though, so make sure it is well-defined.

##### AdLib #####
AdLib sounds are written to specifically talk to the AdLib sound card. It starts with a header of six bytes: the first four bytes are an unsigned 32-bit integer for the *length* of the sound data in bytes, the remaining two bytes are the *priority*, similar to the priority for PC speaker sound.

Then comes the relevant part: 16 bytes of instrument settings followed by a byte for the octave number and then the data bytes with the length from above.

Finally we have a footer consisting of a terminator byte, not used by the game, and a null-terminated ASCII string for the file name, not used either.

| Data type    | Name       | Description              |
|--------------|------------|--------------------------|
| uint_32      | length     | Length of the sound data |
| uint_16      | priority   | Higher priority wins     |
| byte[16]     | instrument | Instrument settings      |
| byte         | octave     | Octave to play notes at  |
| byte[length] | data       | Actual audio data        |
| uint_8       | terminator | Unused by the game       |
| char[]       | file name  | Null-terminated string   |

The instrument settings are as follows:

| Data type | Name    | OPL register | Description                                        |
|-----------|---------|--------------|----------------------------------------------------|
| uint_8    | mChar   | 0x20         | Modulator characteristics                          |
| uint_8    | cChar   | 0x23         | Carrier characteristics                            |
| uint_8    | mScale  | 0x40         | Modulator scale                                    |
| uint_8    | cScale  | 0x43         | Carrier scale                                      |
| uint_8    | mAttack | 0x60         | Modulator attack/decay rate                        |
| uint_8    | cAttack | 0x63         | Carrier attack/decay rate                          |
| uint_8    | mSus    | 0x80         | Modulator sustain                                  |
| uint_8    | cSus    | 0x83         | Carrier sustain                                    |
| uint_8    | mWave   | 0xE0         | Modulator waveform                                 |
| uint_8    | cWave   | 0xE3         | Carrier waveform                                   |
| uint_8    | nConn   | 0xC0         | Feedback/connection (usually ignored and set to 0) |
| uint_8    | voice   | -            | unused by game                                     |
| uint_8    | mode    | -            | unused by game                                     |
| uint_8[3] | padding | -            | pad instrument definition up to 16 bytes           |

Sound effects are played on channel *0* because the other channels of the sound card are reserved for music; the replay rate is 140Hz. The octave value is written to AdLib register *0xB0* and it must be computed to following way to prevent it from interfering with other bits stored in the register:

	block = (octave & 7) << 2       // 7=00000111b
	regB0 = block | other_fields

The audio data consists oft he raw bytes to send to register *0xA0* and the byte *0x00* means silence. Silence can be achieved by setting the fifth bit (hexadecimal 0x20) to 0 in register 0xB0. Here is the pseudocode for playback:

	Constants: `block`   = (octave & 7) << 2
	           `note_on` = 0x20
	
	Prerequisites: Byte sequence of audio data to read
	
	1) Make boolean variable `note` and set to false
	2) Make byte variable `next_byte`
	3) While there is data to read
		3.1) Read `next_byte`
		3.2) If (`next_byte` == 0x00)
			3.2.1) Set register 0xB0 to `block`
			3.2.2) Set `note` to false
		3.3) Else
			3.3.1) Set register 0xA0 to `next_byte`
			3.3.2) If (`note` == false)
				3.3.2.1) Set register 0xB0 to (`block` | `note_on`)
				3.3.2.2) Set `note` to true
		3.4) Wait until next tick (playmback rate 140Hz)

The original code also checked if the next byte was equal to the previous one, and if so it kept playing the same note instead of sending the same data to the sound card again.

##### Digitised #####
Digitised sound effects, such as voices or gun shots are stored in the VSWAP file. That file has been discussed in the *sprites* section, so refer there for information on how to read the file. The data chunks are raw PCM data, played back at a sample rate of 7000Hz, mono sound and eight bytes per sample.

Where it gets complicated is that some audio files are split over multiple successive audio chunks; one example is the very first effect ("Achtung!), which is split over the first and second chunk. That is also why there are more sound effect chunks than there are sound effects (120 instead of 46). We must read the last chunk of the VSWAP file, it contains the audio list consisting of pairs of words; the first word is the index of the biginning of the first audio chunk, the second word is the length of the complete audio chunk. This means that the global list of lengths and offsets detailed in the *sprites* section is only needed for the offsets.

The index of a sound effect chunk can be learned by adding the effect's index from the audio list and the sound start index. This gives us the global file index of the first chunk of the sound effect. Using this global index we can find the offset of the chunk in the lgobal list. The length of the total audio sequence is the length from the audio list.

The number of digitised sound effects is the length of the audio list divided by four. The length of the list is the length of the VSWAP file minus the offset of the list, i.e. the list is the very last chunk of the file.

#### Music tracks ####
The music format is `WLF`, which is essentially type-1 `IMF` whith a playback rate of 700Hz instead of 560Hz. Here is a *WLF* file is composed:

| Type         | Name       | Description                            |
|--------------|------------|----------------------------------------|
| uint_16      | length     | Length of the sound data               |
| byte[length] | sound data | The sound data to play                 |
| byte[]       | metadata   | Arbitrary metadata, unused by the game |

The sound data consists of byte quartets of the following form:

| Type | Description    |
|------|----------------|
| byte | AdLib register |
| byte | AdLib data     |
| word | Delay          |

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

--------------------------------------------------------------------------------

Part II - Game rules
====================
Time is measured in *ticks* from now on. In the original implementation one tick was intended to last 1/70th of a second. The game was inteded to run at one ticks per frame or 70 frames per second.

Mathematics
-----------
//TODO This whole section might be superfluous
To faithfully recreate the gameplay of Wolfenstein 3D one has to understand how the developers worked around the technical limitations of the original hardware. Even if we were to use proper modern techniques we should at least know under what quirks the original implementation had.

### Fixed point instead of floating point ###
The processor of the target hardware, the Intel 286 and 386, did not natively support floating point operations, they would have to be implemente in software, which would have been too slow for gameplay. The solution was to use fixed-point arithmetic by using integers. That would give the programmers half the bits on both sides of the radix point. Truncating the fractional part of such a number can be done by right-shifting by half the type's size. Here is an example using a 32-bit integer:

	| 2^n | 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 | -1 | -2 | -3 | -4 | -5 | -6 | -7 | -8 |
	|-----|---|---|---|---|---|---|---|---|----|----|----|----|----|----|----|----|
	| bit | 0 | 0 | 1 | 0 | 0 | 0 | 1 | 0 |  0 |  0 |  1 |  0 |  0 |  0 |  1 |  0 |
	
	1*2^5 + 1*2^1 + 1*2^(-3) + 1*2^(-7) = 32 + 2 + 0.125 + 0.0078125 = 34.1328125

The number can be treated like an integer for the most part.


### Enumerations and constants ###
The game has a number of hard-coded constants for gameplay.

| Name       | Type    | Value        | Description                           |
|------------|---------|--------------|---------------------------------------|
| FLOATTILE  | Float   | 65536.0f     | ???                                   |
| TILEGLOBAL | integer | 0x10000      | ???                                   |
| HALFTILE   | integer | 0x08000      | 0.5 as fixed-point decimal            |
| MINDIST    | integer | 0x05800      | ???                                   |
| STEP       | float   | 0.0078125f   | How many degrees are one step         |
| STEPRAD    | float   | 0.000136354f | How many radians are one step         |
| MAX_GUARDS | integer | 255          | Maximum number of enemies in the game |
| SPDPATROL  | integer | 512          | Patrolling speed of humans            |
| SPDDOG     | integer | 1500         | Patrolling speed of dogs              | 
These are the enumerations defined in the code:

	quadrant    = {first, second, third, fourth}
	direction_8 = {east, north_east, north, north_west, west, south_west, south, south_east}
	direction_4 = {east,             north,             west,             south            }

All enumerations are mapped to integer values as defined in the C standard: the first element has value 0 and ever successive element has a value +1 greater than the previous one. In the following enumeration elements will be treated as equivalent to integers.

### Functions and macros ###
There are a number of functions and macros defined. The first batch is standard stuff:

	max(x, y) : maximum of two numbers
	abs(x)    : absolute value of a number

The following are converting between world-space and tile-space; to understand them we need to know that positions are stored as 32-bit integers representing fixed-point decimals. Shifting a number by `TILESHIFT` (=16) left turns an integer into a decimal and shifting right turns a decimal into an integer.

	tile_to_pos(a) : converters tile coordinate to world coordinate
	                 Make `a` into fixed-point, add `HALFTILE`
	pos_to_tile(a) : converts world coordinate to tile coordinate
	                 Make `a` into an integer
	
	pos_to_tile_f(a) : Converts world coordinate to floating-point tile coordinate
	                   divide `a` by `FLOATTILE`

### Angles & trigonometry ###
The limited precision offered by fixed-point arithmetic forced the developers to work around it. Angles are given in *steps* and can be converted to degree and radians. See the table of constatns for the conversion ratios. Here is the list of pre-defined angles in steps:

| Degrees | Steps |
|---------|-------|
|     5   |     0 |
|     1   |   128 |
|     6   |   768 |
|    15   |  1920 |
|    22.5 |  2880 |
|    30   |  3840 |
|    45   |  5760 |
|    67.5 |  8640 |
|    90   | 11520 |
|   112.5 | 14400 |
|   135   | 17280 |
|   157.5 | 20160 |
|   180   | 32040 |
|   202.5 | 25920 |
|   225   | 28800 |
|   247.5 | 31680 |
|   270   | 34560 |
|   292.5 | 37440 |
|   315   | 40320 |
|   337.5 | 43200 |
|   360   | 46080 |

All of these numbers could be computed at runtime from one base value, but they were manually pre-computed and hard-coded. Conversion between steps and angles works as follows:

	step_to_radian(a) = (`a` * PI) / `angle_180`
	radian_to_step(a) = (`a` * `angle_180`) / PI
	
	step_to_degree(a)   = (float)(a) / angle_1
	step_to_degree_f(a) = (a) / (float)angle_1
	degree_to_step(a)   = (a) * angle_1

The first cast prevents precision loss during division, the second cast makes the result of the division itself a floating-point number.

After defining these discrete angles we build tables of trigonometric values. The sine- cosine and tangens table simply hold the respective values for each angle. Finally we have a number of angle-related functions:

	normalize_angle(a) : convert any integer to a number between 0 and 360, in steps

To convert an angle to a direction we use the *floor*: an angle always corresponds to the nearest direction that's below an angle. For instance, an 89° angle would correspond to north-east, because it's rounded down to 45°.


Levels
------


Actors / Entities
-----------------
Actors, or entities as they can also be referred to in the code, are any in-game entities that can move around in the world. They include enemies as well as projectiles like fireballs or rockets and even BJ himself, but not static objects like weapons, food, chairs or stone columns. An actor's behaviour is modelled using a finite-state machine where each state holds information on what sprite to display, how long the state lasts, what state to transition to.

### The actor structure ###
An actor is define as a structure with the following members:

| Type        | Name           | Description                        |
|-------------|----------------|------------------------------------|
| integer     | position_x     | Horizontal position on the map     |
| integer     | position_y     | Vertical position on the map       |
| integer     | angle          | Angle the actor is facing          |
| integer     | type           | Class of the actor (e.g. guard)    |
| integer     | current_health | Current health of the actor        |
| integer     | maximum_health | Maximum health of the actor        |
| integer     | speed          | Walking speed                      |
| integer     | tic_count      | Timer driving the actions          |
| integer     | reaction       | Reaction time for noticing player? |
| integer     | distance;      | ???                                |
| character   | tile_x         | Tile the actor is standing on      |
| character   | tile_y         | Tile the actor is standing on      |
| character   | area_number    | Area on the map                    |
| integer     |	waitfordoor_x  | // waiting on this door if non 0   |
| integer     | waitfordoor_y  |                                    |
| actor_flags | flags 	       | Various flags for game rules       |
| actor_state | state          | Currents state                     |
| dir8type    | direction      | Direction to move into             |
| integer     | sprite         | Sprite to display                  |

The type `actor_flags` is a combination of various options which can be either on or off.

| Option      | Meaning |
|-------------|---------|
| Shootable   | ?       |
| Bonus       | ?       |
| Nevermark   | ?       |
| Visable     | ?       |
| Attackmode  | ?       |
| Firstattack | ?       |
| Ambush      | ?       |
| Nonmark     | ?       |

#### Starting hit points ####
The starting hit points of an actor depend on the chosen game difficuly. The list can be found in the appendix, since it would be too large for this section. (//TODO)


### Actor states ###
Each actor state uses the same basic state structure:

| Type      | Name        | Description                                            |
|-----------|-------------|--------------------------------------------------------|
| `boolean` | can_rotate  | `true` if actor has unique sprites for every rotation  |
| `int`     | base_sprite | Base sprite for when facing the player                 |
| `int`     | timeout     | Duration of the state until transiotiong to next state |
| `think`   | thought     | Function to call every frame during this state         |
| `think`   | action      | Function to call when changing state                   |
| `state`   | next_state  | Next state to transition to naturally                  |

The first member tells us wheter the actor has different sprites for rotation or if it is always facing the player; for example, guards have different directions for walking, allowing the player to sneak behind them, but they always face the player when they are shooting or when they are dying.

The second member tells us the index of the base sprite, the image to display when the actor is facing the player. For non-rotateable states this is the sprite to always display, but for ratateable states the right sprite has to be found using the base sprite and adding an appropiate offset to get the index of the proper sprite. The offset depends on the rotation of the actor relative to the player.

The `think` type is a function pointer to a function that takes one actor as its argument, usually the actor calling it, and returns nothing:

	typedef void (*think_t)( entity_t *self )

We can see that these states allow the actors to naturally transition from one state into another solely based on time passed. A patrolling enemy will cycle between patrolling states on its own as long as it doesn't become aware of the player, an enemy in pain will naturally transition to shooting and a dying enemy will automatically be dead once the dying animation has finished playing. The exact actor states are hard-coded and can be found within the *wolf_act_stat.h* file of the original source. There can be several states with simila function, like several walking states, they are driving the animation frames.

#### Groups of states ####
States can be split into the follwing groups:

- **Standing still** The actor is just standing in one spot and waiting
- **Patrolling** The actor is moving along a pre-defined part and can open doors if needed. Dogs cannot stand still and must always walk.
- **In pain** Temporarily paralysed after getting shot at
- **Attacking** Shooting for humans and biting for dogs
- **Chasing** Actively pursuing the player and occasionally stopping to shoot
- **Dying** In the process of dying
- **Dead** Having died
- **Removed** ???

Each of these groups consists of several actual states, with the exception of the standing- and dead state since there is only one way of standing still or being dead. If a state is unused it is still defined, but its members are useless junk data and the sprite is the "demo" sprite. Each state can only display one sprite, so in order to cycle through animation frames the states within one group must be cycled through. In the case of the brown guard there are three shooting frames, so the guard cycles through the first three of his shooting states with the remaining shooting states being unused. There also appear to be special states for some actors, but those are just the above states re-purposed.

#### Changing state ####
To change the state of an actor set its state to the target state. If the state is the `remove` state set the `tic_count` to `0`, otherwise set it to the `timeout` of the target state.

	prerequisites: `actor`       = existing actor
	               `target`      = target state
	               `state_table` = maps actor and state to concrete state object
	
	side effects: will change the `state` and `tic_count` of `actor`
	
	1) Set state of `actor` to `target`
	2) If `target` == `remove`
		2.1) Set tic_count of `actor` to 0
	3) Else
		3.1) Set tic_count of `actor` to timeout of `state_table`(`actor`, `target`)

#### Actor routine ####
The following routine if called every frame on every actor when processing actors (see below). The variable `ticks` measures the number of ticks that have passed since the last frame; for a 30 FPS game that wuld be two ticks.


	prerequisites: `actor` = the actor to run the routine on
	               `tics`  = ticks passed since last time
	
	side effects: - might change the state of `actor`
	              - might call the `thought` and `action` of the state
	
	return value: boolean, false if `actor` ends up in the `remove` state
	
	1) If `tic_count` of `actor` != 0
		1.1) Subtract `ticks` from `tic_count` of `actor`
		1.2) While `tick_count` of `actor` <= 0
			1.2.1) Set `action` to the `action` of `actor`
			1.2.2) If `action` is not NULL
				1.2.2.1) Perform `action`
				1.2.2.2) If `state` of `actor` is `remove`
					1.2.2.2.1) Return false
			1.2.3) Transition to next state
			1.2.4) If the state is `remove`
				1.2.4.1) Return false
			1.2.5) If `timeout` of the state is 0
				1.2.5.1) Set `tic_count` of `actor` to 0
				1.2.5.2) Break out of the loop
			1.2.6) Add `timeout` of the state to the `tic_count` of `actor`
	2) Set `thought` to the `thought` of `actor`
	3) If `thought` is not NULL
		3.1) Perform `think`
		3.2) If the state of `actor` is `remove`
			3.2.1) Return false
	4) Return true


The routine has two major parts. In the first part we subtract the time passed from the actor's tick count. If the count drops to 0 or below we have to call the actor's action and change the state. We have to do this for every state that has passed since the last run of the routine.

This routine is not perfect, if the game speed drops too low the subtracted ticks might skip too many calls of the actor's *think* function.

#### Removing an actor ####
To remove an actor remove it from the global list of actors. This will make any functions that iterates over actors skip it, but the actor will still remain as a corpse sprite in the game.

#### Processing actors ####
Pseudocode:

	1) For each living (i.e. not dead) actor do the following
		1.1) Run the actor routine on the current actor
		1.2) If the routine returned false
			1.2.1) Remove the actor and skip to the next actor
		1.3) Adjust the position and angle of the actor's sprite
		1.4) If the actor state can rotate
			1.4.1) Add the rotation to the index of the base sprite
		1.5) Display the sprite

Rotating a sprite means taking the actor's angle and computing the closest direction. Each direction can be mapped to an integer number and this number is added to the index of the base sprite texture (the one facing the player). The mapping is as follows

	r_add8dir[ 9 ] = { 4, 7, 6, 5, 0, 1, 2, 3, 0 };  // for rockets and hrockets
	a_add8dir[ 9 ] = { 4, 5, 6, 7, 0, 1, 2, 3, 0 };  // for every other actor

The index of the direction to use is the direction of the angle difference between the player and the actor. This means we first compute the absolute difference in angles between actor and player and use that angle to get an eight-way direction. This direction is the index of the number to add.

#### Creating a new actor ####
Creating a new actor is the invers of removing it. Instantiate a new empty actor and add it to the list of actors. Its members will be initialised by the function calling this.

#### Spawning actors ####
Spawning actors is split into a number of similar but not exatly same functions. There are standing actors, patrolling actors, dead actors, bosses and ghosts. All the spawning functions call one general spawning function.

In my opinion these are too many special cases that should be resolved using a sort of table and only one spawning function.

##### Spawn general actor #####
This function is called by other functions to spawn an actor in the world. Pseudocode:

	prerequisites: `class` = actor class of the new actor
	               `x`     = tile X-coordinate of the actor
	               `y`     = tile Y-coordinate of the actor
	               `dir`   = 4-way direction for the actor to face
	               `level` = the level to spawn in
	
	1) Create a new actor as `actor`
	2) Convert `x` and `y` to to world positions and set them as the actor position
	3) Set `angle` and `direction` of `actor` to `dir`
	4) Set `area_number` to area of tile the actor is standing on
	5) If `area_number` < 0
		5.1) Set`area_number` to 0 
	6) Set `type` of the actor to `class`
	7) Set `health` of the actor from the health table (see appendix)
	8) Set `sprite` of the actor to a newly created sprite

##### Spawning standing actor #####
This function spawns a regular still-standing actor. The actor can be either on guard or in ambush mode (deaf) Pseudocode:

	prerequisites: `class` = actor class of the new actor
	               `x`     = tile X-coordinate of the actor
	               `y`     = tile Y-coordinate of the actor
	               `dir`   = 4-way direction for the actor to face
	               `level` = the level to spawn in
	
	1) Spawn a new actor as `actor`
	2) Set `state` of the actor to `stand` and `speed` to `SPDPATROL`
	3) If `timeout` of the state for this actor class and state class `stand` != 0
		3.1) Set `tic_count` of the actor to `timeout`+1
	4) Else
		4.1) Set `tic_count` of the actor to 0
	5) Add the Shootable flag to the actor
	6) If the actor is standing on an ambush tile
		6.1) Add the Ambush flag to the actor
	7) Increment level's enemy count

##### Spawning patrolling actor #####
This function spawns a patrolling actor, dogs always patrol. Pseudocode:

	prerequisites: `class` = actor class of the new actor
	               `x`     = tile X-coordinate of the actor
	               `y`     = tile Y-coordinate of the actor
	               `dir`   = 4-way direction for the actor to face
	               `level` = the level to spawn in
	
	1) Spawn a new actor as `actor`
	2) Set `state` of the actor to `path1` and `speed` to `SPDPATROL`
	3) Set `speed` of the actor to SPDPATROL, or SPDDOG if the actor is a dog
	4) Set `distance` of the actor to TILEGLOBAL
	5) If the `timeout` of the state from the state table != 0
		5.1) Set `tic_count` of the actor to the `timeout`+1
	6) Else
		5.1) Set `tic_count` of the actor to 0
	7) Add the Shootable flag to the actor
	8) Increment level's enemy count


##### Spawning dead actor #####
Dead actors are special in that they have no direction to look at. Pseudocode:

	prerequisites: `class` = actor class of the new actor
	               `x`     = tile X-coordinate of the actor
	               `y`     = tile Y-coordinate of the actor
	
	1) Spawn a new actor as `actor` with no direction
	2) Set `state` of the actor to `dead`
	3) Set health and `speed` of the actor to 0
	5) If the `timeout` of the state from the state table != 0
		5.1) Set `tic_count` of the actor to the `timeout`+1
	6) Else
		5.1) Set `tic_count` of the actor to 0


##### Spawning boss actor #####
The direction of bosses depend on the particular boss. Pseudocode:

	prerequisites: `class` = actor class of the new actor
	               `x`     = tile X-coordinate of the actor
	               `y`     = tile Y-coordinate of the actor
	
	 1) Make 4-way direction variable `dir`
	 2) Value of dir is
	 	2.1) South for: Hans, Schabbs, Fettgesicht and Hitler
	 	2.2) North for: Fake Hitler, Gretel and Giftmacher
	 	2.3) No direction for everything else
	 3) Spawn a new actor as `actor` with direction `dir`
	 4) Set the state of the actor to `path_1` for a spectre and `stand` for everyone else
	 5) Set `speed` of the actor to SPDPATROL
	 6) Set `health` of the actor from the starting health table (redundant?)
	 7) If the `timeout` of the state from the state table != 0
	 	7.1) Set `tic_count` of the actor to the `timeout`+1
	 8) Else
	 	8.1) Set `tic_count` of the actor to 0
	 9) Add the Shootable and Ambush flag to the actor
	10) Increment level's enemy count
	

##### Spawning ghost actor #####
This function spawns Pac-Man ghosts. Pseudocode

	prerequisites: `class` = actor class of the new actor
	               `x`     = tile X-coordinate of the actor
	               `y`     = tile Y-coordinate of the actor
	
	1) Spawn a new actor as `actor` with no direction
	2) Set `state` of the actor to `chase1`
	3) Set `speed` of the actor to SPDPATROL*3
	4) Set `health` of the actor from the starting health table (redundant?)
	5) If the `timeout` of the state from the state table != 0
		7.1) Set `tic_count` of the actor to the `timeout`+1
	6) Else
		8.1) Set `tic_count` of the actor to 0
	7) Add the Ambush flag to the actor
	8) Increment level's enemy count

### Actor AI ###

#### General AI routine ####
These AI routines are not called directly, but they are called by other functions

##### Check Sight #####
This routine scans the line of sight of the actor for the presence of the player. Pseudocode:

	constants: `MINSIGHT` = 1.1; below this distance the player is always noticed
	
	1) If the actor does not have the Ambush flag set and the player is not in the same area
		1.1) Return false
	2) If the difference of the actor's and player's position on both coordinates is less than `MINSIGHT`
		2.1) Return true
	3) If the player is not in front of the actor return false
	   We only compare the direction of the player, not if the actor can actually see the player
	4) Return the result of the Check Line function using the actor and the player
	   The function is discussed in the level section

First we exclude the cases that are easy to verify. Then we exclude the cases where the player is behind the actor, e.g. if the actor is facing south and the player is north of the actor. Finally, we check if the line between the actor and the player is unobstructed; the player could be hiding around a corner or behind a door like in the title screen.

##### First Sighting #####
This routine puts the actor into an attack state and makes it face the player. Pseudocode:

	1) Play a sound and multiply the actor's `speed` by a factor
	   Both depend on the actor's class, there is a table below
	2) If the actor's `waitfordoor_x` != 0
		2.1) Set the actor's `waitfordoor_x` and `waitfordoor_y` to 0
	3) Change the actor's state to Chase1
	4) Set actor's direction to no direction
	5) Set the actor's Attackmode and Firstattack flags on

If the actor was waiting for a door to open while is spotted the player the waiting is canacelled since the actor is now primarily concerned with killing the player, not opening a door. Here is the table with the sound effects and speed factors for the individual actor classes.

| Actor class | Sound number | Factor          |
|-------------|--------------|-----------------|
| Guard       |            1 |             * 3 |
| Officer     |           71 |             * 5 |
| SS          |           15 |             * 4 |
| Dog         |            2 |             * 2 |
| Hans        |           71 | = SPDPATROL * 3 |
| Schabbs     |           65 |             * 3 |
| Fake        |           54 |             * 3 |
| Mecha       |           40 |             * 3 |
| Hitler      |           40 |             * 5 |
| Mutant      |              |             * 3 |
| Blinky      |              |             * 2 |
| Clyde       |              |             * 2 |
| Pinky       |              |             * 2 |
| Inky        |              |             * 2 |
| Gretel      |          112 |             * 3 |
| Gift        |           96 |             * 3 |
| Fat         |          102 |             * 3 |
|             |              |                 |
| Officer     |           43 |                 |
| Spectre     |            3 |          =  800 |
| Angel       |           95 |          = 1536 |
| Trans       |           66 |          = 1536 |
| Uber        |              |          = 3000 |
| Will        |           73 |          = 2048 |
| Death       |           85 |          = 2048 |

The officer has a different sound for Spear of Destiny, but the same speed. Speeds prepended with '=' are set to a fixed value. The ghosts and the (uber)mutant have no sound to play.

##### Find Target #####
This routine is scanning the surroundings of the actor for the player. After the player has been spotted the actor will act surprised for a while and actios will be delayed; this is achieved by the actor's `reaction` member. Pseudocode

	returns: true if the player was detected, false otherwise
	
	side effects: Changes the `react` of the actor
	              Might change the actor's Ambush flag
	
	1) If `reaction` of the actor > 0
		1.1) Subtract ticks since last frame from `reaction`
		1.2) If `reaction` > 0
			1.2.1) Return false
		1.3) Set `reaction` to 0
	2) Else
		2.1) Set `reaction` to 0
		2.2) If the player has the Notarget flag set ("notarget" cheat)
			2.2.1) Return false
		2.3) If the actor does not have the ambush flag and the player is not in the same area
			2.3.1) Return false
		2.4) If Check Sight returned false //failed to see, attempt to hear
			2.4.1) If the actor has the Ambush flag set or if the player hasn't made any noise
				2.4.1.1) Return false
		2.5) Remove the Ambush flag from the actor
		2.6) Set the actor's `reaction` depending on the actor's class
			2.6.1) For guards to (1 + Random/4)
			2.6.2) For officers to 2
			2.6.3) For SS and mutants to (1 + Random/6)
			2.6.4) For dogs to (1 + Random/8)
			2.6.5) For everyone else to 1
		2.7) Return false
	3) Run the First Sighting routine on the actor
	4) Return true

This function works in two ways: If the player hasn't been spotted it will keep looking. Once the player has been spotted a reaction delay will be initialised. As long as that delay persists the function will just keep decrementing it. Only after the reaction delay has passed will the function be called where the actor does actually react.

Note that once an actor has spotted the player it will eventually react, there is no way to quickly run into hiding or that the actor will forget about the player.

####2.6Special AI routines ####
Thes2.6 AI routines are called directly

##### Stand #####


Appendix
========

Tables for actors
-----------------

### List of actor classes and starting hit points ###
The following is a list of all types of actors defined in Wolfenstein 3D and their description, as well as their starting hit points for every dificulty. They are ordered by their appearance in the game.

| Name    | Description                       | Baby | Easy | Normal | Hard |
|---------|-----------------------------------|-----:|-----:|-------:|-----:|
| Guard   | Brown guards                      |   25 |   25 |     25 |   25 |
| Officer | White soldiers                    |   50 |   50 |     50 |   50 |
| SS      | Blue soldiers                     |  100 |  100 |    100 |  100 |
| Dog     | Shepherd dogs                     |    1 |    1 |      1 |    1 |
| Hans    | Hans Grosse                       |  850 |  950 |   1050 | 1200 |
| Schabbs | Dr. Schabbs                       |  850 |  950 |   1550 | 2400 |
| Fake    | Fake Hitler                       |  200 |  300 |    400 |  500 |
| Mecha   | Mecha Hitler                      |  800 |  950 |   1050 | 1200 |
| Hitler  | Real Hitler                       |  500 |  700 |    800 |  900 |
| Mutant  | Mutant soldier                    |   45 |   55 |     55 |   65 |
| Blinky  | Red Pac-Man ghost                 |   25 |   25 |     25 |   25 |
| Clyde   | Orange Pac-Man ghost              |   25 |   25 |     25 |   25 |
| Pinky   | Pink Pac-Man ghost                |   25 |   25 |     25 |   25 |
| Inky    | Cyan Pac-Man ghost                |   25 |   25 |     25 |   25 |
| Gretel  | Gretel Grosse                     |  850 |  950 |   1050 | 1200 |
| Gift    | Otto Gifmacher                    |  850 |  950 |   1050 | 1200 |
| Fat     | General Fettgesicht               |  850 |  950 |   1050 | 1200 |
|         |                                   |      |      |        |      |
| Needle  | Syringe thrown by Schabbs         |    0 |    0 |      0 |    0 |
| Fire    | Fireball from Fake Hitler         |    0 |    0 |      0 |    0 |
| Rocket  | Some bosses have rocket launchers |    0 |    0 |      0 |    0 |
| Smoke   | Smoke from rockets                |    0 |    0 |      0 |    0 |
|         |                                   |      |      |        |      |
| BJ      | BJ doing the victory jump         |  100 |  100 |    100 |  100 |

Spear of destiny adds the following actors as well:

| Name    | Description      | Baby | Easy | Normal | Hard |
|---------|------------------|-----:|-----:|-------:|-----:|
| Spark   | ???              |    0 |    0 |      0 |    0 |
| hrocket | ???              |    0 |    0 |      0 |    0 |
| hsmoke  | ???              |    0 |    0 |      0 |    0 |
|         |                  |      |      |        |      |
| Spectre | Spectre enemy    |   10 |   10 |     15 |   25 |
| Angel   | Angle of Death   | 1550 | 1550 |   1650 | 2000 |
| Trans   | Trans Grosse     |  950 |  950 |   1050 | 1200 |
| Uber    | Ubermutant       | 1150 | 1150 |   1250 | 1400 |
| Will    | Barnacle Wilhelm | 1050 | 1050 |   1150 | 1300 |
| Death   | Death Knight     | 1350 | 1350 |   1450 | 1600 |

The mission packs introduce the following enemies:

| Name  | Description           | Baby | Easy | Normal | Hard |
|-------|-----------------------|-----:|-----:|-------:|-----:|
| Bat   | Bats, replace mutants |   10 |   10 |     15 |   25 |
| Willi | Sumbarine Willi       | 1550 | 1550 |   1650 | 2000 |
| Quark | Dr. Quarkblitz        |  950 |  950 |   1050 | 1200 |
| Axe   | The Axe               | 1150 | 1150 |   1250 | 1400 |
| Robot | The Robot             | 1050 | 1050 |   1150 | 1300 |
| Devil | Decil Incarnate       | 1350 | 1350 |   1450 | 1600 |

Due to the hardcoded nature of the Wolfenstein 3D engine all the enemies in the mission packs are just re-skins of enemies from Spear of Destiny, to their starting hit points are the same.

### List of state classes ###
The following is a list of all types of actor states in the order they are defined in the original source. Sticking to that order is not strictly necessary, but one has to make sure to keep the mapping scorrect.

##### Standing still #####
| State | Index |
|-------|-------|
| stand |     0 |
	
##### Patrolling #####
| State  | Index |
|--------|-------|
| path1  |     1 |
| path1s |     2 |
| path2  |     3 |
| path3  |     4 |
| path3s |     5 |
| path4  |     6 |
	
##### Paralysed in pain #####
| State | Index |
|-------|-------|
| pain  |     7 |
| pain1 |     8 |
	
##### Shooting #####
| State  | Index |
|--------|-------|
| shoot1 |     9 |
| shoot2 |    10 |
| shoot3 |    11 |
| shoot4 |    12 |
| shoot5 |    13 |
| shoot6 |    14 |
| shoot7 |    15 |
| shoot8 |    16 |
| shoot9 |    17 |
	
##### Chasing the player #####
| State   | Index |
|---------|-------|
| chase1  |    18 |
| chase1s |    19 |
| chase2  |    20 |
| chase3  |    21 |
| chase3s |    22 |
| chase4  |    23 |
	
##### Dying #####
| State | Index |
|-------|-------|
| die1  |    24 |
| die2  |    25 |
| die3  |    26 |
| die4  |    27 |
| die5  |    28 |
| die6  |    29 |
| die7  |    30 |
| die8  |    31 |
| die9  |    32 |

##### Being dead #####
| State  | Index |
|--------|-------|
| dead   |    33 |
| remove |    34 |

A list of all actor states for each actor would be too large for this document, so it will have its own file. (//TODO)

### Table of actor states ###
The following tables list the states for every actor class and state class. If a field is empty, then there is nothing defined for it. In the case of a function pointer it means the pointer is `NULL` and in the case of a sprite the sprite is just some useless junk sprite.

#### Guard ####
| State       | Can Rotate | Base Sprite    | Timeout | Thought | Action       | Next State |
|-------------|------------|----------------|--------:|---------|--------------|------------|
| **stand**   | true       | SPR_GRD_S_1    |       0 | Stand   |              | stand      |
|             |            |                |         |         |              |            |
| **path1**   | true       | SPR_GRD_W1_1   |      20 | Path    |              | path1s     |
| **path1s**  | true       | SPR_GRD_W1_1   |       5 |         |              | path2      |
| **path2**   | true       | SPR_GRD_W2_1   |      15 | Path    |              | path3      |
| **path3**   | true       | SPR_GRD_W3_1   |      20 | Path    |              | path3s     |
| **path3s**  | true       | SPR_GRD_W3_1   |       5 |         |              | path4      |
| **path4**   | true       | SPR_GRD_W4_1   |      15 | Path    |              | path1      |
|             |            |                |         |         |              |            |
| **pain**    | false      | SPR_GRD_PAIN_1 |      10 |         |              | chase1     |
| **pain1**   | false      | SPR_GRD_PAIN_2 |      10 |         |              | chase1     |
|             |            |                |         |         |              |            |
| **shoot1**  | false      | SPR_GRD_SHOOT1 |      20 |         |              | shoot2     |
| **shoot2**  | false      | SPR_GRD_SHOOT2 |      20 |         | Shoot        | shoot3     |
| **shoot3**  | false      | SPR_GRD_SHOOT3 |      20 |         |              | chase1     |
| **shoot4**  | false      |                |       0 |         |              | chase1     |
| **shoot5**  | false      |                |       0 |         |              | chase1     |
| **shoot6**  | false      |                |       0 |         |              | chase1     |
| **shoot7**  | false      |                |       0 |         |              | chase1     |
| **shoot8**  | false      |                |       0 |         |              | chase1     |
| **shoot9**  | false      |                |       0 |         |              | chase1     |
|             |            |                |         |         |              |            |
| **chase1**  | true       | SPR_GRD_W1_1   |      10 | Chase   |              | chase1s    |
| **chase1s** | true       | SPR_GRD_W1_1   |       3 |         |              | chase2     |
| **chase2**  | true       | SPR_GRD_W2_1   |       8 | Chase   |              | chase3     |
| **chase3**  | true       | SPR_GRD_W3_1   |      10 | Chase   |              | chase3s    |
| **chase3s** | true       | SPR_GRD_W3_1   |       3 |         |              | chase4     |
| **chase4**  | true       | SPR_GRD_W4_1   |       8 | Chase   |              | chase1     |
|             |            |                |         |         |              |            |
| **die1**    | false      | SPR_GRD_DIE_1  |      15 |         | Death Scream | die2       |
| **die2**    | false      | SPR_GRD_DIE_2  |      15 |         |              | die3       |
| **die3**    | false      | SPR_GRD_DIE_3  |      15 |         |              | dead       |
| **die4**    | false      |                |       0 |         |              | dead       |
| **die5**    | false      |                |       0 |         |              | dead       |
| **die6**    | false      |                |       0 |         |              | dead       |
| **die7**    | false      |                |       0 |         |              | dead       |
| **die8**    | false      |                |       0 |         |              | dead       |
| **die9**    | false      |                |       0 |         |              | dead       |
|             |            |                |         |         |              |            |
| **dead**    | false      | SPR_GRD_DEAD   |       0 |         |              | dead       |


#### Officer ####
| State       | Can Rotate | Base Sprite    | Timeout | Thought | Action       | Next State |
|-------------|------------|----------------|--------:|---------|--------------|------------|
| **stand**   | true       | SPR_OFC_S_1    |       0 | Stand   |              | stand      |
|             |            |                |         |         |              |            |
| **path1**   | true       | SPR_OFC_W1_1   |      20 | Path    |              | path1s     |
| **path1s**  | true       | SPR_OFC_W1_1   |       5 |         |              | path2      |
| **path2**   | true       | SPR_OFC_W2_1   |      15 | Path    |              | path3      |
| **path3**   | true       | SPR_OFC_W3_1   |      20 | Path    |              | path3s     |
| **path3s**  | true       | SPR_OFC_W3_1   |       5 |         |              | path4      |
| **path4**   | true       | SPR_OFC_W4_1   |      15 | Path    |              | path1      |
|             |            |                |         |         |              |            |
| **pain**    | false      | SPR_GRD_PAIN_1 |      10 |         |              | chase1     |
| **pain1**   | false      | SPR_GRD_PAIN_2 |      10 |         |              | chase1     |
|             |            |                |         |         |              |            |
| **shoot1**  | false      | SPR_GRD_SHOOT1 |       6 |         |              | shoot2     |
| **shoot2**  | false      | SPR_GRD_SHOOT2 |      20 |         | Shoot        | shoot3     |
| **shoot3**  | false      | SPR_GRD_SHOOT3 |      10 |         |              | chase1     |
| **shoot4**  | false      |                |       0 |         |              | chase1     |
| **shoot5**  | false      |                |       0 |         |              | chase1     |
| **shoot6**  | false      |                |       0 |         |              | chase1     |
| **shoot7**  | false      |                |       0 |         |              | chase1     |
| **shoot8**  | false      |                |       0 |         |              | chase1     |
| **shoot9**  | false      |                |       0 |         |              | chase1     |
|             |            |                |         |         |              |            |
| **chase1**  | true       | SPR_OFC_W1_1   |      10 | Chase   |              | chase1s    |
| **chase1s** | true       | SPR_OFC_W1_1   |       3 |         |              | chase2     |
| **chase2**  | true       | SPR_OFC_W2_1   |       8 | Chase   |              | chase3     |
| **chase3**  | true       | SPR_OFC_W3_1   |      10 | Chase   |              | chase3s    |
| **chase3s** | true       | SPR_OFC_W3_1   |       3 |         |              | chase4     |
| **chase4**  | true       | SPR_OFC_W4_1   |       8 | Chase   |              | chase1     |
|             |            |                |         |         |              |            |
| **die1**    | false      | SPR_OFC_DIE_1  |      11 |         | Death Scream | die2       |
| **die2**    | false      | SPR_OFC_DIE_2  |      11 |         |              | die3       |
| **die3**    | false      | SPR_OFC_DIE_3  |      11 |         |              | dead       |
| **die4**    | false      |                |       0 |         |              | dead       |
| **die5**    | false      |                |       0 |         |              | dead       |
| **die6**    | false      |                |       0 |         |              | dead       |
| **die7**    | false      |                |       0 |         |              | dead       |
| **die8**    | false      |                |       0 |         |              | dead       |
| **die9**    | false      |                |       0 |         |              | dead       |
|             |            |                |         |         |              |            |
| **dead**    | false      | SPR_OFC_DEAD   |       0 |         |              | dead       |


#### SS ####
| State       | Can Rotate | Base Sprite   | Timeout | Thought | Action       | Next State |
|-------------|------------|---------------|--------:|---------|--------------|------------|
| **stand**   | true       | SPR_SS_S_1    |       0 | Stand   |              | stand      |
|             |            |               |         |         |              |            |
| **path1**   | true       | SPR_SS_W1_1   |      20 | Path    |              | path1s     |
| **path1s**  | true       | SPR_SS_W1_1   |       5 |         |              | path2      |
| **path2**   | true       | SPR_SS_W2_1   |      15 | Path    |              | path3      |
| **path3**   | true       | SPR_SS_W3_1   |      20 | Path    |              | path3s     |
| **path3s**  | true       | SPR_SS_W3_1   |       5 |         |              | path4      |
| **path4**   | true       | SPR_SS_W4_1   |      15 | Path    |              | path1      |
|             |            |               |         |         |              |            |
| **pain**    | false      | SPR_SS_PAIN_1 |      10 |         |              | chase1     |
| **pain1**   | false      | SPR_SS_PAIN_2 |      10 |         |              | chase1     |
|             |            |               |         |         |              |            |
| **shoot1**  | false      | SPR_SS_SHOOT1 |      20 |         |              | shoot2     |
| **shoot2**  | false      | SPR_SS_SHOOT2 |      20 |         | Shoot        | shoot3     |
| **shoot3**  | false      | SPR_SS_SHOOT3 |      10 |         |              | chase1     |
| **shoot4**  | false      | SPR_SS_SHOOT2 |      10 |         | Shoot        | chase1     |
| **shoot5**  | false      | SPR_SS_SHOOT3 |      10 |         |              | chase1     |
| **shoot6**  | false      | SPR_SS_SHOOT2 |      10 |         | Shoot        | chase1     |
| **shoot7**  | false      | SPR_SS_SHOOT3 |      10 |         |              | chase1     |
| **shoot8**  | false      | SPR_SS_SHOOT2 |      10 |         | Shoot        | chase1     |
| **shoot9**  | false      | SPR_SS_SHOOT3 |      10 |         |              | chase1     |
|             |            |               |         |         |              |            |
| **chase1**  | true       | SPR_SS_W1_1   |      10 | Chase   |              | chase1s    |
| **chase1s** | true       | SPR_SS_W1_1   |       3 |         |              | chase2     |
| **chase2**  | true       | SPR_SS_W2_1   |       8 | Chase   |              | chase3     |
| **chase3**  | true       | SPR_SS_W3_1   |      10 | Chase   |              | chase3s    |
| **chase3s** | true       | SPR_SS_W3_1   |       3 |         |              | chase4     |
| **chase4**  | true       | SPR_SS_W4_1   |       8 | Chase   |              | chase1     |
|             |            |               |         |         |              |            |
| **die1**    | false      | SPR_SS_DIE_1  |      15 |         | Death Scream | die2       |
| **die2**    | false      | SPR_SS_DIE_2  |      15 |         |              | die3       |
| **die3**    | false      | SPR_SS_DIE_3  |      15 |         |              | dead       |
| **die4**    | false      |               |       0 |         |              | dead       |
| **die5**    | false      |               |       0 |         |              | dead       |
| **die6**    | false      |               |       0 |         |              | dead       |
| **die7**    | false      |               |       0 |         |              | dead       |
| **die8**    | false      |               |       0 |         |              | dead       |
| **die9**    | false      |               |       0 |         |              | dead       |
|             |            |               |         |         |              |            |
| **dead**    | false      | SPR_SS_DEAD   |       0 |         |              | dead       |


#### Dog ####
Dogs have no pain because they have only one hit point.

| State       | Can Rotate | Base Sprite   | Timeout | Thought | Action       | Next State |
|-------------|------------|---------------|--------:|---------|--------------|------------|
| **stand**   | false      |               |       0 |         |              | stand      |
|             |            |               |         |         |              |            |
| **path1**   | true       | SPR_DOG_W1_1  |      20 | Path    |              | path1s     |
| **path1s**  | true       | SPR_DOG_W1_1  |       5 |         |              | path2      |
| **path2**   | true       | SPR_DOG_W2_1  |      15 | Path    |              | path3      |
| **path3**   | true       | SPR_DOG_W3_1  |      20 | Path    |              | path3s     |
| **path3s**  | true       | SPR_DOG_W3_1  |       5 |         |              | path4      |
| **path4**   | true       | SPR_DOG_W4_1  |      15 | Path    |              | path1      |
|             |            |               |         |         |              |            |
| **pain**    | false      |               |      10 |         |              | chase1     |
| **pain1**   | false      |               |      10 |         |              | chase1     |
|             |            |               |         |         |              |            |
| **shoot1**  | false      | SPR_DOG_JUMP1 |      20 |         |              | shoot2     |
| **shoot2**  | false      | SPR_DOG_JUMP2 |      20 |         | Shoot        | shoot3     |
| **shoot3**  | false      | SPR_DOG_JUMP3 |      10 |         |              | chase1     |
| **shoot4**  | false      | SPR_DOG_JUMP2 |      10 |         | Shoot        | chase1     |
| **shoot5**  | false      | SPR_DOG_W1_1  |      10 |         |              | chase1     |
| **shoot6**  | false      |               |      10 |         | Shoot        | chase1     |
| **shoot7**  | false      |               |      10 |         |              | chase1     |
| **shoot8**  | false      |               |      10 |         | Shoot        | chase1     |
| **shoot9**  | false      |               |      10 |         |              | chase1     |
|             |            |               |         |         |              |            |
| **chase1**  | true       | SPR_DOG_W1_1  |      10 | Chase   |              | chase1s    |
| **chase1s** | true       | SPR_DOG_W1_1  |       3 |         |              | chase2     |
| **chase2**  | true       | SPR_DOG_W2_1  |       8 | Chase   |              | chase3     |
| **chase3**  | true       | SPR_DOG_W3_1  |      10 | Chase   |              | chase3s    |
| **chase3s** | true       | SPR_DOG_W3_1  |       3 |         |              | chase4     |
| **chase4**  | true       | SPR_DOG_W4_1  |       8 | Chase   |              | chase1     |
|             |            |               |         |         |              |            |
| **die1**    | false      | SPR_DOG_DIE_1 |      15 |         | Death Scream | die2       |
| **die2**    | false      | SPR_DOG_DIE_2 |      15 |         |              | die3       |
| **die3**    | false      | SPR_DOG_DIE_3 |      15 |         |              | dead       |
| **die4**    | false      |               |       0 |         |              | dead       |
| **die5**    | false      |               |       0 |         |              | dead       |
| **die6**    | false      |               |       0 |         |              | dead       |
| **die7**    | false      |               |       0 |         |              | dead       |
| **die8**    | false      |               |       0 |         |              | dead       |
| **die9**    | false      |               |       0 |         |              | dead       |
|             |            |               |         |         |              |            |
| **dead**    | false      | SPR_DOG_DEAD  |       0 |         |              | dead       |


#### Hans Grosse ####
Hans has no pain and no patrol.

| State       | Can Rotate | Base Sprite     | Timeout | Thought | Action          | Next State |
|-------------|------------|-----------------|--------:|---------|-----------------|------------|
| **stand**   | true       | SPR_BOSS_W1     |       0 | Stand   |                 | stand      |
|             |            |                 |         |         |                 |            |
| **path1**   | true       |                 |       0 | Path    |                 | path1s     |
| **path1s**  | true       |                 |       0 |         |                 | path2      |
| **path2**   | true       |                 |       0 | Path    |                 | path3      |
| **path3**   | true       |                 |       0 | Path    |                 | path3s     |
| **path3s**  | true       |                 |       0 |         |                 | path4      |
| **path4**   | true       |                 |       0 | Path    |                 | path1      |
|             |            |                 |         |         |                 |            |
| **pain**    | false      |                 |       0 |         |                 | chase1     |
| **pain1**   | false      |                 |       0 |         |                 | chase1     |
|             |            |                 |         |         |                 |            |
| **shoot1**  | false      | SPR_BOSS_SHOOT1 |      30 |         |                 | shoot2     |
| **shoot2**  | false      | SPR_BOSS_SHOOT2 |      10 |         | Shoot           | shoot3     |
| **shoot3**  | false      | SPR_BOSS_SHOOT3 |      10 |         | Shoot           | chase4     |
| **shoot4**  | false      | SPR_BOSS_SHOOT2 |      10 |         | Shoot           | chase5     |
| **shoot5**  | false      | SPR_BOSS_SHOOT3 |      10 |         | Shoot           | chase6     |
| **shoot6**  | false      | SPR_BOSS_SHOOT2 |      10 |         | Shoot           | chase7     |
| **shoot7**  | false      | SPR_BOSS_SHOOT3 |      10 |         | Shoot           | chase8     |
| **shoot8**  | false      | SPR_BOSS_SHOOT1 |      10 |         |                 | chase1     |
| **shoot9**  | false      |                 |       0 |         |                 | chase1     |
|             |            |                 |         |         |                 |            |
| **chase1**  | true       | SPR_BOSS_W1     |      10 | Chase   |                 | chase1s    |
| **chase1s** | true       | SPR_BOSS_W1     |       3 |         |                 | chase2     |
| **chase2**  | true       | SPR_BOSS_W2     |       8 | Chase   |                 | chase3     |
| **chase3**  | true       | SPR_BOSS_W3     |      10 | Chase   |                 | chase3s    |
| **chase3s** | true       | SPR_BOSS_W3     |       3 |         |                 | chase4     |
| **chase4**  | true       | SPR_BOSS_W4     |       8 | Chase   |                 | chase1     |
|             |            |                 |         |         |                 |            |
| **die1**    | false      | SPR_BOSS_DIE_1  |      15 |         | Death Scream    | die2       |
| **die2**    | false      | SPR_BOSS_DIE_2  |      15 |         |                 | die3       |
| **die3**    | false      | SPR_BOSS_DIE_3  |      15 |         |                 | dead       |
| **die4**    | false      |                 |       0 |         |                 | dead       |
| **die5**    | false      |                 |       0 |         |                 | dead       |
| **die6**    | false      |                 |       0 |         |                 | dead       |
| **die7**    | false      |                 |       0 |         |                 | dead       |
| **die8**    | false      |                 |       0 |         |                 | dead       |
| **die9**    | false      |                 |       0 |         |                 | dead       |
|             |            |                 |         |         |                 |            |
| **dead**    | false      | SPR_BOSS_DEAD   |       0 |         | Start Death Cam | dead       |


#### Dr. Schabbs ####
| State       | Can Rotate | Base Sprite            | Timeout | Thought    | Action          | Next State |
|-------------|------------|------------------------|--------:|------------|-----------------|------------|
| **stand**   | false      | SPR_SCHABB_W1          |       0 | Stand      |                 | stand      |
|             |            |                        |         |            |                 |            |
| **path1**   | false      |                        |       0 |            |                 | path1s     |
| **path1s**  | false      |                        |       0 |            |                 | path2      |
| **path2**   | false      |                        |       0 |            |                 | path3      |
| **path3**   | false      |                        |       0 |            |                 | path3s     |
| **path3s**  | false      |                        |       0 |            |                 | path4      |
| **path4**   | false      |                        |       0 |            |                 | path1      |
|             |            |                        |         |            |                 |            |
| **pain**    | false      |                        |       0 |            |                 | chase1     |
| **pain1**   | false      |                        |       0 |            |                 | chase1     |
|             |            |                        |         |            |                 |            |
| **shoot1**  | false      | SPR_SCHABB_SHOOT1      |      30 |            |                 | shoot2     |
| **shoot2**  | false      | SPR_SCHABB_SHOOT2      |      10 |            | Launch          | chase1     |
| **shoot3**  | false      |                        |      10 |            |                 | chase1     |
| **shoot4**  | false      |                        |      10 |            |                 | chase1     |
| **shoot5**  | false      |                        |      10 |            |                 | chase1     |
| **shoot6**  | false      |                        |      10 |            |                 | chase1     |
| **shoot7**  | false      |                        |      10 |            |                 | chase1     |
| **shoot8**  | false      |                        |      10 |            |                 | chase1     |
| **shoot9**  | false      |                        |       0 |            |                 | chase1     |
|             |            |                        |         |            |                 |            |
| **chase1**  | false      | SPR_SCHABB_W1          |      10 | Boss Chase |                 | chase1s    |
| **chase1s** | false      | SPR_SCHABB_W1          |       3 |            |                 | chase2     |
| **chase2**  | false      | SPR_SCHABB_W2          |       8 | Boss Chase |                 | chase3     |
| **chase3**  | false      | SPR_SCHABB_W3          |      10 | Boss Chase |                 | chase3s    |
| **chase3s** | false      | SPR_SCHABB_W3          |       3 |            |                 | chase4     |
| **chase4**  | false      | SPR_SCHABB_W4          |       8 | Boss Chase |                 | chase1     |
|             |            |                        |         |            |                 |            |
| **die1**    | false      | SPR_SCHABB_SCHABB_W1   |      10 |            | Death Scream    | die2       |
| **die2**    | false      | SPR_SCHABB_SCHABB_W1   |      10 |            |                 | die3       |
| **die3**    | false      | SPR_SCHABB_SCHABB_DIE1 |      10 |            |                 | die4       |
| **die4**    | false      | SPR_SCHABB_SCHABB_DIE2 |      10 |            |                 | die5       |
| **die5**    | false      | SPR_SCHABB_SCHABB_DIE3 |      10 |            |                 | die6       |
| **die6**    | false      |                        |       0 |            |                 | dead       |
| **die7**    | false      |                        |       0 |            |                 | dead       |
| **die8**    | false      |                        |       0 |            |                 | dead       |
| **die9**    | false      |                        |       0 |            |                 | dead       |
|             |            |                        |         |            |                 |            |
| **dead**    | false      | SPR_SCHABB_DEAD        |       0 |            | Start Death Cam | dead       |


#### Fake Hitler ####
| State       | Can Rotate | Base Sprite    | Timeout | Thought    | Action       | Next State |
|-------------|------------|----------------|--------:|------------|--------------|------------|
| **stand**   | false      | SPR_FAKE_W1    |       0 | Stand      |              | stand      |
|             |            |                |         |            |              |            |
| **path1**   | false      |                |       0 |            |              | path1s     |
| **path1s**  | false      |                |       0 |            |              | path2      |
| **path2**   | false      |                |       0 |            |              | path3      |
| **path3**   | false      |                |       0 |            |              | path3s     |
| **path3s**  | false      |                |       0 |            |              | path4      |
| **path4**   | false      |                |       0 |            |              | path1      |
|             |            |                |         |            |              |            |
| **pain**    | false      |                |       0 |            |              | chase1     |
| **pain1**   | false      |                |       0 |            |              | chase1     |
|             |            |                |         |            |              |            |
| **shoot1**  | false      | SPR_FAKE_SHOOT |       8 |            | Launch       | shoot2     |
| **shoot2**  | false      | SPR_FAKE_SHOOT |       8 |            | Launch       | shoot3     |
| **shoot3**  | false      | SPR_FAKE_SHOOT |       8 |            | Launch       | shoot4     |
| **shoot4**  | false      | SPR_FAKE_SHOOT |       8 |            | Launch       | shoot5     |
| **shoot5**  | false      | SPR_FAKE_SHOOT |       8 |            | Launch       | shoot6     |
| **shoot6**  | false      | SPR_FAKE_SHOOT |       8 |            | Launch       | shoot7     |
| **shoot7**  | false      | SPR_FAKE_SHOOT |       8 |            | Launch       | shoot8     |
| **shoot8**  | false      | SPR_FAKE_SHOOT |       8 |            | Launch       | shoot9     |
| **shoot9**  | false      | SPR_FAKE_SHOOT |       8 |            |              | chase1     |
|             |            |                |         |            |              |            |
| **chase1**  | false      | SPR_FAKE_W1    |      10 | Fake       |              | chase1s    |
| **chase1s** | false      | SPR_FAKE_W1    |       3 |            |              | chase2     |
| **chase2**  | false      | SPR_FAKE_W2    |       8 | Fake       |              | chase3     |
| **chase3**  | false      | SPR_FAKE_W3    |      10 | Fake       |              | chase3s    |
| **chase3s** | false      | SPR_FAKE_W3    |       3 |            |              | chase4     |
| **chase4**  | false      | SPR_FAKE_W4    |       8 | Fake       |              | chase1     |
|             |            |                |         |            |              |            |
| **die1**    | false      | SPR_FAKE_DIE1  |      10 |            | Death Scream | die2       |
| **die2**    | false      | SPR_FAKE_DIE2  |      10 |            |              | die3       |
| **die3**    | false      | SPR_FAKE_DIE3  |      10 |            |              | die4       |
| **die4**    | false      | SPR_FAKE_DIE4  |      10 |            |              | die5       |
| **die5**    | false      | SPR_FAKE_DIE5  |      10 |            |              | dead       |
| **die6**    | false      |                |       0 |            |              | dead       |
| **die7**    | false      |                |       0 |            |              | dead       |
| **die8**    | false      |                |       0 |            |              | dead       |
| **die9**    | false      |                |       0 |            |              | dead       |
|             |            |                |         |            |              |            |
| **dead**    | false      | SPR_FAKE_DEAD  |       0 |            |              | dead       |


#### Mecha Hitler ####
| State       | Can Rotate | Base Sprite      | Timeout | Thought    | Action       | Next State |
|-------------|------------|------------------|--------:|------------|--------------|------------|
| **stand**   | false      | SPR_MECHA_W1     |       0 | Stand      |              | stand      |
|             |            |                  |         |            |              |            |
| **path1**   | false      |                  |       0 |            |              | path1s     |
| **path1s**  | false      |                  |       0 |            |              | path2      |
| **path2**   | false      |                  |       0 |            |              | path3      |
| **path3**   | false      |                  |       0 |            |              | path3s     |
| **path3s**  | false      |                  |       0 |            |              | path4      |
| **path4**   | false      |                  |       0 |            |              | path1      |
|             |            |                  |         |            |              |            |
| **pain**    | false      |                  |       0 |            |              | chase1     |
| **pain1**   | false      |                  |       0 |            |              | chase1     |
|             |            |                  |         |            |              |            |
| **shoot1**  | false      | SPR_MECHA_SHOOT1 |      30 |            |              | shoot2     |
| **shoot2**  | false      | SPR_MECHA_SHOOT2 |      10 |            | Shoot        | shoot3     |
| **shoot3**  | false      | SPR_MECHA_SHOOT3 |      10 |            | Shoot        | shoot4     |
| **shoot4**  | false      | SPR_MECHA_SHOOT2 |      10 |            | Shoot        | shoot5     |
| **shoot5**  | false      | SPR_MECHA_SHOOT3 |      10 |            | Shoot        | shoot6     |
| **shoot6**  | false      | SPR_MECHA_SHOOT2 |      10 |            | Shoot        | shoot7     |
| **shoot7**  | false      |                  |       0 |            |              | shoot8     |
| **shoot8**  | false      |                  |       0 |            |              | shoot9     |
| **shoot9**  | false      |                  |       0 |            |              | chase1     |
|             |            |                  |         |            |              |            |
| **chase1**  | false      | SPR_MECHA_W1     |      10 | Chase      | Mecha Sound  | chase1s    |
| **chase1s** | false      | SPR_MECHA_W1     |       6 |            |              | chase2     |
| **chase2**  | false      | SPR_MECHA_W2     |       8 | Chase      |              | chase3     |
| **chase3**  | false      | SPR_MECHA_W3     |      10 | Chase      | Mecha Sound  | chase3s    |
| **chase3s** | false      | SPR_MECHA_W3     |       6 |            |              | chase4     |
| **chase4**  | false      | SPR_MECHA_W4     |       8 | Chase      |              | chase1     |
|             |            |                  |         |            |              |            |
| **die1**    | false      | SPR_MECHA_DIE1   |      10 |            | Death Scream | die2       |
| **die2**    | false      | SPR_MECHA_DIE2   |      10 |            |              | die3       |
| **die3**    | false      | SPR_MECHA_DIE3   |      10 |            | Hitler Morph | dead       |
| **die4**    | false      |                  |       0 |            |              | dead       |
| **die5**    | false      |                  |       0 |            |              | dead       |
| **die6**    | false      |                  |       0 |            |              | dead       |
| **die7**    | false      |                  |       0 |            |              | dead       |
| **die8**    | false      |                  |       0 |            |              | dead       |
| **die9**    | false      |                  |       0 |            |              | dead       |
|             |            |                  |         |            |              |            |
| **dead**    | false      | SPR_MECHA_DEAD   |       0 |            |              | dead       |


#### Adolf Hitler ####
| State       | Can Rotate | Base Sprite       | Timeout | Thought    | Action          | Next State |
|-------------|------------|-------------------|--------:|------------|-----------------|------------|
| **stand**   | false      |                   |       0 | Stand      |                 | stand      |
|             |            |                   |         |            |                 |            |
| **path1**   | false      |                   |       0 |            |                 | path1s     |
| **path1s**  | false      |                   |       0 |            |                 | path2      |
| **path2**   | false      |                   |       0 |            |                 | path3      |
| **path3**   | false      |                   |       0 |            |                 | path3s     |
| **path3s**  | false      |                   |       0 |            |                 | path4      |
| **path4**   | false      |                   |       0 |            |                 | path1      |
|             |            |                   |         |            |                 |            |
| **pain**    | false      |                   |       0 |            |                 | chase1     |
| **pain1**   | false      |                   |       0 |            |                 | chase1     |
|             |            |                   |         |            |                 |            |
| **shoot1**  | false      | SPR_HITLER_SHOOT1 |      30 |            |                 | shoot2     |
| **shoot2**  | false      | SPR_HITLER_SHOOT2 |      10 |            | Shoot           | shoot3     |
| **shoot3**  | false      | SPR_HITLER_SHOOT3 |      10 |            | Shoot           | shoot4     |
| **shoot4**  | false      | SPR_HITLER_SHOOT2 |      10 |            | Shoot           | shoot5     |
| **shoot5**  | false      | SPR_HITLER_SHOOT3 |      10 |            | Shoot           | shoot6     |
| **shoot6**  | false      | SPR_HITLER_SHOOT2 |      10 |            | Shoot           | shoot7     |
| **shoot7**  | false      |                   |       0 |            |                 | shoot8     |
| **shoot8**  | false      |                   |       0 |            |                 | shoot9     |
| **shoot9**  | false      |                   |       0 |            |                 | chase1     |
|             |            |                   |         |            |                 |            |
| **chase1**  | false      | SPR_HITLER_W1     |       6 | Chase      |                 | chase1s    |
| **chase1s** | false      | SPR_HITLER_W1     |       4 |            |                 | chase2     |
| **chase2**  | false      | SPR_HITLER_W2     |       2 | Chase      |                 | chase3     |
| **chase3**  | false      | SPR_HITLER_W3     |       6 | Chase      |                 | chase3s    |
| **chase3s** | false      | SPR_HITLER_W3     |       4 |            |                 | chase4     |
| **chase4**  | false      | SPR_HITLER_W4     |       2 | Chase      |                 | chase1     |
|             |            |                   |         |            |                 |            |
| **die1**    | false      | SPR_HITLER_W1     |       1 |            | Death Scream    | die2       |
| **die2**    | false      | SPR_HITLER_W1     |      10 |            |                 | die3       |
| **die3**    | false      | SPR_HITLER_DIE1   |      10 |            | Hitler Morph    | die4       |
| **die4**    | false      | SPR_HITLER_DIE2   |      10 |            |                 | die5       |
| **die5**    | false      | SPR_HITLER_DIE3   |      10 |            |                 | die6       |
| **die6**    | false      | SPR_HITLER_DIE4   |      10 |            |                 | die7       |
| **die7**    | false      | SPR_HITLER_DIE5   |      10 |            |                 | die8       |
| **die8**    | false      | SPR_HITLER_DIE6   |      10 |            |                 | die9       |
| **die9**    | false      | SPR_HITLER_DIE7   |      10 |            |                 | dead       |
|             |            |                   |         |            |                 |            |
| **dead**    | false      | SPR_HITLER_DEAD   |       0 |            | Start Death Cam | dead       |


#### Mutant ####
| State       | Can Rotate | Base Sprite    | Timeout | Thought    | Action       | Next State |
|-------------|------------|----------------|--------:|------------|--------------|------------|
| **stand**   | true       | SPR_MUT_S_1    |       0 | Stand      |              | stand      |
|             |            |                |         |            |              |            |
| **path1**   | true       | SPR_MUT_W1_1   |      20 | Path       |              | path1s     |
| **path1s**  | true       | SPR_MUT_W1_1   |       5 |            |              | path2      |
| **path2**   | true       | SPR_MUT_W2_1   |      15 | Path       |              | path3      |
| **path3**   | true       | SPR_MUT_W3_1   |      20 | Path       |              | path3s     |
| **path3s**  | true       | SPR_MUT_W3_1   |       5 |            |              | path4      |
| **path4**   | true       | SPR_MUT_W4_1   |      15 | Path       |              | path1      |
|             |            |                |         |            |              |            |
| **pain**    | false      | SPR_MUT_PAIN_1 |      10 |            |              | chase1     |
| **pain1**   | false      | SPR_MUT_PAIN_2 |      10 |            |              | chase1     |
|             |            |                |         |            |              |            |
| **shoot1**  | false      | SPR_MUT_SHOOT1 |       6 |            |              | shoot2     |
| **shoot2**  | false      | SPR_MUT_SHOOT2 |      20 |            | Shoot        | shoot3     |
| **shoot3**  | false      | SPR_MUT_SHOOT3 |      10 |            | Shoot        | shoot4     |
| **shoot4**  | false      | SPR_MUT_SHOOT4 |      20 |            | Shoot        | shoot5     |
| **shoot5**  | false      |                |       0 |            | Shoot        | shoot6     |
| **shoot6**  | false      |                |       0 |            | Shoot        | chase1     |
| **shoot7**  | false      |                |       0 |            |              | chase1     |
| **shoot8**  | false      |                |       0 |            |              | chase1     |
| **shoot9**  | false      |                |       0 |            |              | chase1     |
|             |            |                |         |            |              |            |
| **chase1**  | true       | SPR_MUT_W1_1   |      10 | Chase      |              | chase1s    |
| **chase1s** | true       | SPR_MUT_W1_1   |       3 |            |              | chase2     |
| **chase2**  | true       | SPR_MUT_W2_1   |       8 | Chase      |              | chase3     |
| **chase3**  | true       | SPR_MUT_W3_1   |      10 | Chase      |              | chase3s    |
| **chase3s** | true       | SPR_MUT_W3_1   |       3 |            |              | chase4     |
| **chase4**  | true       | SPR_MUT_W4_1   |       8 | Chase      |              | chase1     |
|             |            |                |         |            |              |            |
| **die1**    | false      | SPR_MUT_DIE_1  |       7 |            | Death Scream | die2       |
| **die2**    | false      | SPR_MUT_DIE_2  |       7 |            |              | die3       |
| **die3**    | false      | SPR_MUT_DIE_3  |       7 |            |              | die4       |
| **die4**    | false      | SPR_MUT_DIE_4  |       7 |            |              | dead       |
| **die5**    | false      |                |       0 |            |              | dead       |
| **die6**    | false      |                |       0 |            |              | dead       |
| **die7**    | false      |                |       0 |            |              | dead       |
| **die8**    | false      |                |       0 |            |              | dead       |
| **die9**    | false      |                |       0 |            |              | dead       |
|             |            |                |         |            |              |            |
| **dead**    | false      | SPR_MUT_DEAD   |       0 |            |              | dead       |


#### Blinky ####
| State       | Can Rotate | Base Sprite   | Timeout | Thought | Action | Next State |
|-------------|------------|---------------|--------:|---------|--------|------------|
| **stand**   | false      |               |       0 |         |        | stand      |
|             |            |               |         |         |        |            |
| **path1**   | false      |               |       0 |         |        | path1s     |
| **path1s**  | false      |               |       0 |         |        | path2      |
| **path2**   | false      |               |       0 |         |        | path3      |
| **path3**   | false      |               |       0 |         |        | path3s     |
| **path3s**  | false      |               |       0 |         |        | path4      |
| **path4**   | false      |               |       0 |         |        | path1      |
|             |            |               |         |         |        |            |
| **pain**    | false      |               |       0 |         |        | chase1     |
| **pain1**   | false      |               |       0 |         |        | chase1     |
|             |            |               |         |         |        |            |
| **shoot1**  | false      |               |       0 |         |        | shoot2     |
| **shoot2**  | false      |               |       0 |         |        | shoot3     |
| **shoot3**  | false      |               |       0 |         |        | shoot4     |
| **shoot4**  | false      |               |       0 |         |        | shoot5     |
| **shoot5**  | false      |               |       0 |         |        | shoot6     |
| **shoot6**  | false      |               |       0 |         |        | chase1     |
| **shoot7**  | false      |               |       0 |         |        | shoot8     |
| **shoot8**  | false      |               |       0 |         |        | shoot9     |
| **shoot9**  | false      |               |       0 |         |        | chase1     |
|             |            |               |         |         |        |            |
| **chase1**  | false      | SPR_BLINKY_W1 |      10 | Ghosts  |        | chase2     |
| **chase1s** | false      |               |       0 |         |        | chase2     |
| **chase2**  | false      | SPR_BLINKY_W2 |      10 | Ghosts  |        | chase1     |
| **chase3**  | false      |               |       0 |         |        | chase3s    |
| **chase3s** | false      |               |       0 |         |        | chase4     |
| **chase4**  | false      |               |       0 |         |        | chase1     |
|             |            |               |         |         |        |            |
| **die1**    | false      |               |      10 |         |        | die2       |
| **die2**    | false      |               |      10 |         |        | die3       |
| **die3**    | false      |               |      10 |         |        | dead       |
| **die4**    | false      |               |       0 |         |        | dead       |
| **die5**    | false      |               |       0 |         |        | dead       |
| **die6**    | false      |               |       0 |         |        | dead       |
| **die7**    | false      |               |       0 |         |        | dead       |
| **die8**    | false      |               |       0 |         |        | dead       |
| **die9**    | false      |               |       0 |         |        | dead       |
|             |            |               |         |         |        |            |
| **dead**    | false      |               |       0 |         |        | dead       |


#### Clyde ####
| State       | Can Rotate | Base Sprite  | Timeout | Thought | Action | Next State |
|-------------|------------|--------------|--------:|---------|--------|------------|
| **stand**   | false      |              |       0 |         |        | stand      |
|             |            |              |         |         |        |            |
| **path1**   | false      |              |       0 |         |        | path1s     |
| **path1s**  | false      |              |       0 |         |        | path2      |
| **path2**   | false      |              |       0 |         |        | path3      |
| **path3**   | false      |              |       0 |         |        | path3s     |
| **path3s**  | false      |              |       0 |         |        | path4      |
| **path4**   | false      |              |       0 |         |        | path1      |
|             |            |              |         |         |        |            |
| **pain**    | false      |              |       0 |         |        | chase1     |
| **pain1**   | false      |              |       0 |         |        | chase1     |
|             |            |              |         |         |        |            |
| **shoot1**  | false      |              |       0 |         |        | shoot2     |
| **shoot2**  | false      |              |       0 |         |        | shoot3     |
| **shoot3**  | false      |              |       0 |         |        | shoot4     |
| **shoot4**  | false      |              |       0 |         |        | shoot5     |
| **shoot5**  | false      |              |       0 |         |        | shoot6     |
| **shoot6**  | false      |              |       0 |         |        | chase1     |
| **shoot7**  | false      |              |       0 |         |        | shoot8     |
| **shoot8**  | false      |              |       0 |         |        | shoot9     |
| **shoot9**  | false      |              |       0 |         |        | chase1     |
|             |            |              |         |         |        |            |
| **chase1**  | false      | SPR_CLYDE_W1 |      10 | Ghosts  |        | chase2     |
| **chase1s** | false      |              |       0 |         |        | chase2     |
| **chase2**  | false      | SPR_CLYDE_W2 |      10 | Ghosts  |        | chase1     |
| **chase3**  | false      |              |       0 |         |        | chase3s    |
| **chase3s** | false      |              |       0 |         |        | chase4     |
| **chase4**  | false      |              |       0 |         |        | chase1     |
|             |            |              |         |         |        |            |
| **die1**    | false      |              |      10 |         |        | die2       |
| **die2**    | false      |              |      10 |         |        | die3       |
| **die3**    | false      |              |      10 |         |        | dead       |
| **die4**    | false      |              |       0 |         |        | dead       |
| **die5**    | false      |              |       0 |         |        | dead       |
| **die6**    | false      |              |       0 |         |        | dead       |
| **die7**    | false      |              |       0 |         |        | dead       |
| **die8**    | false      |              |       0 |         |        | dead       |
| **die9**    | false      |              |       0 |         |        | dead       |
|             |            |              |         |         |        |            |
| **dead**    | false      |              |       0 |         |        | dead       |


#### Pinky ####
| State       | Can Rotate | Base Sprite  | Timeout | Thought | Action | Next State |
|-------------|------------|--------------|--------:|---------|--------|------------|
| **stand**   | false      |              |       0 |         |        | stand      |
|             |            |              |         |         |        |            |
| **path1**   | false      |              |       0 |         |        | path1s     |
| **path1s**  | false      |              |       0 |         |        | path2      |
| **path2**   | false      |              |       0 |         |        | path3      |
| **path3**   | false      |              |       0 |         |        | path3s     |
| **path3s**  | false      |              |       0 |         |        | path4      |
| **path4**   | false      |              |       0 |         |        | path1      |
|             |            |              |         |         |        |            |
| **pain**    | false      |              |       0 |         |        | chase1     |
| **pain1**   | false      |              |       0 |         |        | chase1     |
|             |            |              |         |         |        |            |
| **shoot1**  | false      |              |       0 |         |        | shoot2     |
| **shoot2**  | false      |              |       0 |         |        | shoot3     |
| **shoot3**  | false      |              |       0 |         |        | shoot4     |
| **shoot4**  | false      |              |       0 |         |        | shoot5     |
| **shoot5**  | false      |              |       0 |         |        | shoot6     |
| **shoot6**  | false      |              |       0 |         |        | chase1     |
| **shoot7**  | false      |              |       0 |         |        | shoot8     |
| **shoot8**  | false      |              |       0 |         |        | shoot9     |
| **shoot9**  | false      |              |       0 |         |        | chase1     |
|             |            |              |         |         |        |            |
| **chase1**  | false      | SPR_PINKY_W1 |      10 | Ghosts  |        | chase2     |
| **chase1s** | false      |              |       0 |         |        | chase2     |
| **chase2**  | false      | SPR_PINKY_W2 |      10 | Ghosts  |        | chase1     |
| **chase3**  | false      |              |       0 |         |        | chase3s    |
| **chase3s** | false      |              |       0 |         |        | chase4     |
| **chase4**  | false      |              |       0 |         |        | chase1     |
|             |            |              |         |         |        |            |
| **die1**    | false      |              |      10 |         |        | die2       |
| **die2**    | false      |              |      10 |         |        | die3       |
| **die3**    | false      |              |      10 |         |        | dead       |
| **die4**    | false      |              |       0 |         |        | dead       |
| **die5**    | false      |              |       0 |         |        | dead       |
| **die6**    | false      |              |       0 |         |        | dead       |
| **die7**    | false      |              |       0 |         |        | dead       |
| **die8**    | false      |              |       0 |         |        | dead       |
| **die9**    | false      |              |       0 |         |        | dead       |
|             |            |              |         |         |        |            |
| **dead**    | false      |              |       0 |         |        | dead       |


#### Inky ####
| State       | Can Rotate | Base Sprite | Timeout | Thought | Action | Next State |
|-------------|------------|-------------|--------:|---------|--------|------------|
| **stand**   | false      |             |       0 |         |        | stand      |
|             |            |             |         |         |        |            |
| **path1**   | false      |             |       0 |         |        | path1s     |
| **path1s**  | false      |             |       0 |         |        | path2      |
| **path2**   | false      |             |       0 |         |        | path3      |
| **path3**   | false      |             |       0 |         |        | path3s     |
| **path3s**  | false      |             |       0 |         |        | path4      |
| **path4**   | false      |             |       0 |         |        | path1      |
|             |            |             |         |         |        |            |
| **pain**    | false      |             |       0 |         |        | chase1     |
| **pain1**   | false      |             |       0 |         |        | chase1     |
|             |            |             |         |         |        |            |
| **shoot1**  | false      |             |       0 |         |        | shoot2     |
| **shoot2**  | false      |             |       0 |         |        | shoot3     |
| **shoot3**  | false      |             |       0 |         |        | shoot4     |
| **shoot4**  | false      |             |       0 |         |        | shoot5     |
| **shoot5**  | false      |             |       0 |         |        | shoot6     |
| **shoot6**  | false      |             |       0 |         |        | chase1     |
| **shoot7**  | false      |             |       0 |         |        | shoot8     |
| **shoot8**  | false      |             |       0 |         |        | shoot9     |
| **shoot9**  | false      |             |       0 |         |        | chase1     |
|             |            |             |         |         |        |            |
| **chase1**  | false      | SPR_INKY_W1 |      10 | Ghosts  |        | chase2     |
| **chase1s** | false      |             |       0 |         |        | chase2     |
| **chase2**  | false      | SPR_INKY_W2 |      10 | Ghosts  |        | chase1     |
| **chase3**  | false      |             |       0 |         |        | chase3s    |
| **chase3s** | false      |             |       0 |         |        | chase4     |
| **chase4**  | false      |             |       0 |         |        | chase1     |
|             |            |             |         |         |        |            |
| **die1**    | false      |             |      10 |         |        | die2       |
| **die2**    | false      |             |      10 |         |        | die3       |
| **die3**    | false      |             |      10 |         |        | dead       |
| **die4**    | false      |             |       0 |         |        | dead       |
| **die5**    | false      |             |       0 |         |        | dead       |
| **die6**    | false      |             |       0 |         |        | dead       |
| **die7**    | false      |             |       0 |         |        | dead       |
| **die8**    | false      |             |       0 |         |        | dead       |
| **die9**    | false      |             |       0 |         |        | dead       |
|             |            |             |         |         |        |            |
| **dead**    | false      |             |       0 |         |        | dead       |


Known bugs and limitations
--------------------------

1. A map needs at least 1 enemy, 1 piece of treasure and 1 secret door, or else the game will crash. This is the result of the game trying to calculate the percentage the player has picked up and ending up dividing by zero.

### Distributions of the game and magic numbers ###
Different versions of the game assign different key numbers to the graphics. Each graphic can be identified using an integer "key" magic number and that number depended on what the graphics artists produced. This means the programmers would have had to keep a list of key numbers and always change their code when the graphics changed. Instead the software used by the artists produced alongside the data files also a header file that mapped each number to a macro, so the developers could simply use the macro and the generated header would map the macro to the correct number.

The problem with this is that each build was only suitable for a particular distribution of the game, like shareware, registered, Japanese or Spear of Destiny. The simple solution is to use `#ifdef` directives and set the version at compile time, which is an acceptable solution when building for a particular release, but ill-suited for a source port that needs to be as compatible as possible. I propose the following solution that moves the version detection from compile-time to run-time.

There will be one global header file that has a universal mapping that assigns a number to any image that might exist in any distribution. It is not necessary for it to be compatible to any existing distribution. The programmers will use these global macros then. For each distribution there will be a mapping that maps the universal macro to the distribution's corresponding key number. At run-time when the program starts determine the distribution and assign a global mapping variable to be the mapping for that distribution. The mapping could for example be done using an array where the universal macro is the index and the distribution's key the value. Trying to access an image that does not exist in that particular distribution could be mapped to an invalid number such as -1.

References
----------
The following sources were used for reference and to guide me in the right direction:

- [Wolfenstein 3D source code][1](https://github.com/id-Software/wolf3d)
- [Chocolate Wolfenstein 3D source code][2](https://github.com/fabiensanglard/Chocolate-Wolfenstein-3D)
- [Wolfensein 3D on Modding Wiki][3](http://www.shikadi.net/moddingwiki/Wolfenstein_3-D)
- [Some guy's abandoned attempt to understand the game data][4](http://devinsmith.net/backups/bruce/wolf3d.html)
