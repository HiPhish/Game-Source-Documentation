.. default-role:: code

###################################
Wolfenstein 3D code design document
###################################

Format documentation of asset files in Wolenstein 3D.

.. contents:: Table of Contents
   :depth: 3

--------------------------------------------------------------------------------

Files overview
##############

The assets of Wolfensein 3D are stored in multiple files, unlike more modern
games such as Doom. These files are containers that contain different assets
glued together, and every file usually contains a "header" of sorts that helps
in locating the actual asset we seek.

The files are named the same in all distributions of the game, but their file
extension varies. For simplicity the file extensions will be omitted in this
document when not relevant The following table offers an overview:

=========  ================================
Extension  Version                         
=========  ================================
``WL1``    Shareware                       
``WL3``    Early three-episode full version
``WL6``    Six-episode full version        
``WJ1``    Japanese shareware?             
``WJ6``    Japanese full version?          
``SOD``    Spear of Destiny?               
``SDM``    ???                             
``SD1``    SoD M2: Return to Danger?       
``SD2``    SoD M3: Ultimate challenge      
``SD3``    ???                             
=========  ================================

The files can be grouped roughly in three categories: *graphics*, *audio* and
*maps*. The game assets for WS3D are stored in various files with the same
extension, which is depending on the version of the game. The assets are
distributed as follows:

Graphics
	VGADICT, VGAHEAD, VGAGRAPH
Audio
	AUDIOHED, AUDIOT
Maps
	MAPHEAD, GAMEMAPS

The header files contain information about the structure of the actual asset
files



Conventions and nomenclature
############################

Decimal integer numbers are given as regular numbers, binary numbers are
prefixed with `0b`, octal numbers with `0o` *o* and hexadecimal numbers are
prefixed with `0x`. Example:

.. math::
	13 = 0b1011 = 0o15 = 0xD

All continuous numbers are written in the standard big-endian notation used in
the English language. This means the left-most digit is the most significant
one and a number like `0b1011` is computed as

.. math::
    (1 * 2^3 + 0 * 2^2 + 1 * 2^1 + 1 * 2^0).

Multibyte numbers are given in little-endian notation and the bytes are
seperated by whitespace charakters, thus the multibyte number `0x1A 0x3C`
corresponds to the hexadecimal number `0x3C1A` or decimal `15386`. If the
endianness needs to be explicitly stated it will be noted as `(BE)` for
big-endian and as `(LE)` for little-endian.

Notes are written as ``:NOTE:``, tasks are written as ``:TODO:`` and bugs in the
original implementation are written as ``:BUG:``. This allows using the editor's
search function to quickly jump to these points.


Pseudocode
==========

Pseudocode is based on how C works, so if an integer is added to or subtracted
from a pointer it means the pointer is advanced by the amount of bytes its
value takes up. For instance, if `p` is a pointer to a word and a word is two
bytes in size, then `p + 2` is a pointer that points four bytes (two words)
further away from `p`. Variables in pseudocode are considered immutable, every
arithmetic operation returns a copy. In the above example the pointer `p` would
not advance, instead `p + 2` would be a new pointer. To mutate a variable use a
verb such as *increment* or *advance*.

Pointers and arrays are used more or less interchangably in the pseudocode as
well. Usually when a pointer is described as a "sequence" of something it can
be an array as well. Similarly, the square bracked notation `pointer[i]` means
"the value of `pointer + i`". Whether the actual implementation uses pointers,
arrays, vectors, linked lists of whatever is irrelevant.


Data types
==========

The data types in this document are similar to the types used in the original
source. A *byte* is eight bits in size, a *word* is a sequence of two bytes. All
multibyte-numbers are stored in little-endian format in the asset files. A
*character* or *char* is synonymous to a byte and encodes an ASCII character.

Sometimes data types will be used interchangably, like *byte*, *unint8* of
*char*, depending on what fits the context better. When reading or writing data
it makes more sense to talk about bytes and words, whereas *uint8* or *uint16*
are more suited when the numerical value is relevant.


2D or 3D
========
Despite its name Wolfenstein 3D is not a true 3D game; the game's data and
simulation all happen in a flat 2D space on a strict grid, while the rendering
appears to take place in a 3D world to the player. In this document the game
world (called "world space") will be treated as if it was actually a three-
dimensional space, since that is what the player is experiencing. On the other
hand, simulation space, where all the game's actual mechanics are implemented,
will be treated like a two-dimensional plane, since that is the way the game
works.



Compression Algorithms
######################

The following desctiptions describe the algorithms in general, regardless of how
the game uses them.


RLEW compression
================

A variant of RLE (Run Length Encoding) that uses words instead of bytes as the
underlying unit. Repeating words are stored as a word triplet `(tag, count,
word)` where `tag` is a constant word used to identify the triplet, `count` is
how many times to copy the word and `word` is the word to copy. Aside from
these triplets there are also uncompressed words that are copied verbatim. Here
is the pseudocode:

--------------------------------------------------------------------------------

:Prerequisites:
 - `source`      : pointer to the start of the compressed input stream
 - `destination` : pointer to the start of the decompressed output stream
 - `tag`         : a word used to identify a triplet
 - `length`      : integer length of the decompressed data
 - Must allocate enough memory to hold the decompressed sequence

:Side effects:
 The pre-allocated memory will be filled with decompressed data

:Code:
 1) Make new pointers: `read` = `start`, `write` = `desination`. These
    Pointers will be moved forward while the original pointers remain fixed
 2) While `length` > :math:`0` 

    1) Read `word` pointed at by `read`
    2) If `word` is `tag`

       1) Advance `read` by one word
       2) Make new integer `count` from word pointed at by `read`
       3) Advance `read` by one word
       4) while `count` > :math:`0` 

          1) Copy word under `read` to `write`
          2) Advance `write` by one word
          3) Decrement `count` and `length` by one
       5) Advance `read` by one word
    3) Else

       1) Copy word under `read` to `write`
       2) Advance `read` and `write` by one word
       3) Decrement `length` by one

--------------------------------------------------------------------------------

What about the word that's identical to `tag`? It will be compressed as `(tag,
0x01 0x00, tag)`, i.e. copy the word `tag` one time. This is actually a
threefold increase in data compared to the uncompressed version, but in
practice this is a better solution than having special cases.



Carmack compression
===================

The underlying idea of this compression method is that certain patterns of
information are going to be repeated several times. Instead of repeating the
pattern each time a reference to previous instances of the pattern is stored;
the already uncompressed data is referenced by the still compressed data.

The compressed data consists of uncompressed words, one of two types of
pointers (near pointers and far pointers), and exceptions where all four can
appear in the same file depending on which is necessary. Near pointers are byte
triplets and far pointers are byte quadruples. On top of this there are special
exceptions for words that might be confused for pointers. All offsets are given
in *words*, so to get the *byte* offset multiply the word offset by two.

Before we look at the pseudocode we need to understand the priciples first.


Near pointers
-------------

Near pointers are a sequence of three bytes `(count, 0xA7, offset)`. The first
byte tells us how many words to copy, it is an usingned 8-bit integer. The
second byte is the tag and always `0xA7`, it is used to identify a near
pointer.  The third byte is the unsigned 8-bit integer offset relative from the
last written word to the word to copy. Take the following example

=========================  ========================================
decompresssed data before  `0C 00 0A 00 CD AB 05 00 ??`            
near pointer               `02 A7 03`                              
decompresssed data after   `0C 00 0A 00 CD AB 05 00 0A 00 CD AB ??`
=========================  ========================================

The `??` is the current position of the destination pointer; it points at
memory that has been allocated but not yet been written to, its content is at
this point undefined. The near pointer tells us to copy two words (four bytes)
from three words ago. The resulting output would then be

First a copy of the destination pointer (called *copy pointer*) is moved four
words back, pointing at the byte `0A`. The byte pointed at by the copy pointer
is copied to the value pointed at by the destination pointer and both pointers
are incremented. This is repeated four times, at which point the copy pointer
has reached the original position of the destination pointer.


Far pointers
------------

The disadvantage of near pointers is that the offset is an 8-bit integer, so it
can only reach :math:`255` words back. Far pointers `(count 0xA8 low_offset
high_offset)` use a 16-bit offset, so they take up one more bytes in memory.
The offset is given relative to the start of the decompressed sequence, i.e.
the first destination pointer. Aside from the offset they work the same as near
pointers, their tag is `0xA8`.


Exception
---------

Words with a high byte (second byte) of `0xA7` or `0xA8` can be confused for
pointers. In compressed form the low byte is replaced by the byte `0x00` and
the low bytes value is appened after the high byte. A count of :math:`0` would
make no sense for a pointer, so the algorithm can tell when an exception has
occured.  Since the low byte comes after the high byte the word is actually
stored in big-endian notation and needs to be swapped around when written to
the destination.


Extraction
----------

To decompress the data we need to know the length of the decompressed data
because there is no indication when the end of the compressed sequence is
reached; the compressed data is often stored adjacent to other compressed data
in the same file. On top of that there is also uncompressed data between near-
and far pointers which must be copied verbatim.

Keep count of the bytes or words already written. When using words instead of
bytes to keep track make sure you divide the byte count by two. At first the
count is :math:`0` and it is incremented every time we write a word or byte.
Once the count reaches the size of the decompressed data the extraction is
done. After each write increment the count and advance the pointers
appropriately. This means the destination pointer is advanced by one byte for
every byte written and the source pointer is advanced by three bytes for near
pointers and exceptions, four for far pointers, and two for regular words.

During each iteration step read a word. If the word's high byte (second byte)
is neither the near- nor the far flag copy the word to the destination. If it's
the near flag and the count is not `0x00` step `offset` words back through the
decompressed data and copy `count` words from there to the decompressed data.
If it's a far pointer and the count is not `0x00` copy `count` words `offset`
words from the start of the decompressed data. If the count is zero advance the
pointer by one byte and copy the reversed word.


Pseudocode
----------

This pseudocode operates on words.

--------------------------------------------------------------------------------

:Constants:
 - `zero = 0x00`
 - `near = 0xA7`
 - `far  = 0xA8`

:Prerequisites:
 - `source`      : pointer to the start of the compressed input stream
 - `destination` : pointer to the start of the decompressed output stream
 - `length`      : length of the decompressed data sequence in words
 - Must allocate enough memory to hold the decompressed sequence

:Side effects:
 The pre-allocated memory will be filled with decompressed data

:Code:
 1) Make new pointers: `read = start`, `write = desination`. These pointers
    will be moved forward while the original pointers remain fixed
 2) While `length > 0`

    1) Read the word pointed at by `read`
    2) Make new integer `count` the numeric value of its low byte
    3) Make new integer `flag` the numeric value of its high byte
    4) If `flag` is `near` and `count` is not `zero`

       1) Advance `read` by one byte
       2) Read the word under `read`
       3) Make the new integer `offset` the numeric value of the word's high
          byte
       4) Make the new pointer `copy = write - offset`
       5) While `count > 0`

          1) Copy word under `copy` to `write`
          2) Advance `copy` and `write` by one word each
          3) Decrement `count` and `length` by one each
    5) Else if `flag` is `far` and `count` is not `zero`

       1) Advance read by one word
       2) Read the word under `read`
       3) Make the new integer `offset` the numeric value of the word
       4) Make the new pointer `copy = destination + offset`
       5) While `count > 0`

          1) Copy word under `copy` to `write`
          2) Advance `copy` and `write` by one word each
          3) Decrement `count` and `length` by one each
    6) Else if `flag` is `near` or `far` and `count` is `zero`

       1) Advance `read` by one byte
       2) Copy word under `read` to `write`
       3) Swap bytes of word under `write`
       4) Advance `read` and `write` by one word each
       5) Decrement `length` by one
    7) Else

       1) Copy word under `read` to `write`
       2) Advance `read` and `write` by one word each
       3) Decrement `length` by one

--------------------------------------------------------------------------------

Near- and far pointers are very similar, the only difference is in how the
offset is computed and that near pointer have to advance by one byte while far
pointers advance by one word.



Huffman compression
===================

Id's implementation of the Huffman compression algorithm uses a :math:`255`
node large Huffman tree stored as a flat array where each node consist of two
words, and node number :math:`255` (index :math:`254`) is always the root node.
Here is how the nodes work: a byte called the *node value* is being kept track
of, it is initially :math:`254`, the array position of the root node of the
tree. From there the input of the compressed stream is being read bit-wise, if
the bit is `0` the node value is set to the node's first word, otherwise to the
node's second word. If the node value is less than :math:`256` (i.e. within the
value range of a byte) the node value is written as a byte and the node pointer
is reset back to the root node.  Otherwise, if the node value is eaqual to or
greater than :math:`256` the node pointer is set to the node at array index
(node value - :math:`256`).


Pseudocode
----------

Since the input cannot be read bit-wise it has to be read one byte at a time
and then the input byte is being examined using a masking byte. This byte
starts out as `0x01` and is bitewise `AND`-ed with the input byte to decide
which path down the tree to take. Afterwards the 1-bit of the masking byte is
left-shifted by one to be able to examine the next input-bit. Once the mask
byte reached `0x80` the masking bit is all the way to the left, so we need to
reset it back to `0x01` and read the next input byte.

--------------------------------------------------------------------------------

:Constants: `root = 254`

:Prerequisites:
 - `source`: pointer to the start of the compressed input stream as bytes
 - `destination`: pointer to the start of the decompressed output stream as
   bytes
 - `length`: length of the decompressed data sequence in words
 - `huffman_tree`: array of Huffman-tree nodes for decompression
 - Must allocete enough memory to hold the decompressed sequence

:Data structures:
 `struct huffman_node {word word_0, word_1}` : a structure holding two words

:Side effects: The pre-allocated memory will be filled with decompressed data

:Code:
 1) Make new pointer `node` of type `huffman_node` and set it to
    `huffman_tree[root]`
 2) Make new pointers `read` and `write` and set them to `source` and
    `destination` respectively
 3) Make new byte `mask = 0x01` and `input`, set input to value of `read`,
    advance `read`
 4) Make new word `node_value`
 5) Repeat indefinitely

    1) If `(input & mask) == 0x00`

       1) `node_value = node->word_0`
    2) Else

       1) `node_value = node->word_1`
    3) If `mask == 0x80`, i.e. the masking bit is all the way to the right

       1) Set `input` to value pointed at by read, advance read
       2) Set `mask` back to `0x01`
    4) Else

       1) Bit-shift `mask` by one bit to the left
    5) If `node_value < 256` (hex `0xFF`)

       1) Write the value of `node_vale` as a byte to `write`, advance
          `write`
       2) Reset `node_pointer` back to `huffman_tree[root]`
       3) If the end of the output stream has been reached break out of the loop
    6) Else

       1) `node_pointer = huffman_tree[node_value - 256]`

--------------------------------------------------------------------------------




Graphics
########

There are two types of graphics in the game: *pics* and *sprites*. Pics are
rectangular pictures of any size without any transparent holes and used outside
the 3D portions of the game. An alternative name is *bitmaps*. Sprites are in-
game object graphics using the colur `0x980088` for transparency and are always
:math:`64 \times 64` pixels large.



Pics
====

To extract pics three files are needed:

==========   =========================================
File name    Purpose                                  
==========   =========================================
`VGADICT`    Huffman-tree for decopressing the pics   
`VGAHEAD`    Headers describing where to find the pics
`VGAGRAPH`   Compressed pics lumped together          
==========   =========================================

The pics are all Huffman-compressed, so first the Huffman tree has to be loaded.

VGADICT
   This file is :math:`1024` bytes large, but the last four bytes are just
   `0x00` byte padding. Four consequtive bytes each form a Huffman tree node
   and the node type itself is made of two words, so the file describes
   :math:`255` individual Huffman nodes (:math:`255 /times 4 = 1020`). Only
   those :math:`1020` bytes are read and stored verbatim in an array of
   Huffman-node type of length :math:`255` (size hard coded).  As explained
   above a Huffman-node is a struct holding two words.

VGAHEAD
   This file holds the offsets of the pics and is uncompressed. Each offset is
   a 32-bit signed number, but it is stored using only three bytes instead of
   four. The number of offsets is one more than the number of actual chunks;
   this last offset points to the end of the file. It is necessary because the
   length of a compressed chunk is not encoded anywhere, it needs to be
   computed using the starting offset of the next chunk.

VGAGRAPH
   This is the file containing the Huffman-compressed chunks. The number of
   pics is hard-coded into the executable and cannot be learned from this file
   as not all chunks are actually pics, some are text or palettes. The first
   chunk is the *picture table*, an array of widths and heights for each pic.
   Each array element is a pair of two words, the first being the width and the
   second being the height.


Extracting the pics
-------------------

Pics are stored Huffman-compressed, so first we need to read the Huffman-table.
This is straight forward, simply dump the contents of VGADICT into a pre-
allocated array. All sizes are hard coded. Next we need to read the pic headers
from VGAHEAD.

First we need to know that number of pics used by the game. This can vary
depending on which version of the game is played and the number is hard coded
into the executable. It can also be computed by getting the size of the VGAHEAD
file in bytes and dividing by three since each head is stored as three bytes.
Both approaches are valid and there is a proposal below under "Distributions of
the game and magic numbers" for using hard-coded numbers in a way that's
compatible with multiple versions of the game at runtime.

Using that number allocate space for an array of that many 32-bit integers and
fill each one with the corresponding offset value. Beware that the offsets are
stored in the file using only three bytes, not four. One exception is the
number `0x00FFFFFF` or its corressponding byte sequence `FF FF FF` which gets
mapped to the offset :math:`-1`. It does not appear in neither the registered
six-episode release nor in the shareware release. I am not sure what the reason
is here, but the original release has the following line in the `CA_FarRead`
function

.. code::

	if (length>0xffffl)
		Quit ("CA_FarRead doesn't support 64K reads yet!");

This seems to be a safety check for technical reasons and since that value does
not appear among the offsets anyway I am not certain if it is worth replicating.

Now we need to read the picture table, an array of widths and heights for the
individual pics. Open the VGAGRAPH file and jump to the first offset. We can
read the expanded length of the chunk in bytes as a signed 32-bit integer from
the first four bytes. Now compute the compressed length of this first chunk in
bytes by taking the offset to the next chunk, substracting the offset of the
current chunk and subtracting four (the extpanded length). Now allocate enough
bytes to hold that sequence and fill it with the first chunk minus the first
four bytes. Allocate enough memory to hold the decompressed picture table and
Huffman-expand the first chunk into it.

Now that the preperation work is done we can start extracting the individual
pics. So far we have the Huffman tree, an array of offsets, a pic table
describing the size of each pic and an open VGAGRAPH file. A chunk is
identified using its magic number. Get the offset of the chunk and that of the
next chunk using their magic numbers. If the offset of the chunk is :math:`-1`
abort. We can get the magic number of the next chunk by adding :math:`+1` to
the magic number of the current chunk. If the offset of the next chunk is
:math:`-1` keep adding :math:`+1` to the magic number until the offset is a
proper value. Compute the length of the compressed chunk as the difference in
chunk offsets and fill a buffer of that size and type 32-bit signed integer
with the data of the chunk.

Now we can expand the data. We need to know the expanded size of the chunk,
which can be read from the compressed chunk: the first four bytes are a signed
32-bit integer that tells us the size, so read it and advance the pointer by
four bytes. There is an exception if the chumk number is greater or equal to
`STARTTILE8` and less than `STARTEXTERNS`; I don't really understand what
that is supposed to represent, but the size is hard coded in that case and the
pointer is not advanced. Here it the code in question

.. code::

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

Allocate enough memory for the uncompressed chunk and pass the pointer to the
compressed source, decompressed destination, expanded size and Huffman tree to
the Huffman decompression routine. The destination will then hold the address of
the decompressed pic chunk. All that is left now is interpreting the chunk as an
image.


Interpreting pics
-----------------

Uncompressed pics are stored as sequences of bytes. A byte's unsigned integer
value can range from :math:`0` to :math:`255`, which is exactly how many
colours the VGA standard supports. Each byte stands for a colour index of a
pixel that can be mapped to a colour value using a palette. The palette depends
on the game and can be loaded from an external file or be hard-coded, it maps
the indices to whatever format the target API uses, such as RGBA. In order to
display the image as a two-dimensional surface we also need the width and
height from the picture table above.

Given the size of the picture and a palette we can then assemble the image the
following way

.. code::

	rgb_pixel[i + j*width] = palette[vga_pixel[(j*(width>>2)+(i>>2))+(i&3)*(width>>2)*height]] 

Here `rgb_pixel` is a linear array of output pixels starting in the top-left
corner and growing width-first, height-second. `palette` is an array that maps
a colpur index to an RGB colour value. `vga_pixel` is the array of picture
pixels.  The variables `i` and `j` stand for the current width and height
while building the output image. The operators `>>` and `&` are bitwise
right-shift and bitwise `AND` respectively.

I don't understand how or why pictures need to be "woven" in such a way, I
assume it has to do with the way that the VGA standard works. Trying to order
the pixels linearly instead of weaving them results in :math:`4 \times 4` tiles
of down-scaled versions of the picture; the original picture can still be
recognised. The original code does mention four "layers" when it is about to
send the picture to memory.


Sprites
=======

Sprites are stored in the file VSWAP, together with textures and sound effects,
there are no other files involved. Each sprite is :math:`64 \times 64` pixels
large. They are drawn column-wise and since there is a lot of empty columns
left and right of the visible picture. Only the columns between and including
the outer-most non- empty columns are given. Each column is described via a
variable-length list of drawing instructions, each instruction being six bytes
in size.


VSWAP
-----

The first six bytes of this file is the header consisting of three signed
:math:`16`-bit integers. The first integer is the total number of chunks in the
file, regardless of type. The second integer is the starting index of the
sprite chunks relative to the beginning of the file. The third integer is the
starting index of the sound effects. I will only be focusing on the sprites
here.

Next up is a list of all chunk offsets. They are stored as unsigned
:math:`32`-bit integers and their amount is the number of chunks. It is
followed by a second list, the list of chunk lengths, same amount but stored as
words. To decide whether a chunk is a texture, a sprite or a sound one has to
use the chunk's index and compare it to the number of sprite- and sound chunks
and their starting index. If you want to read a sprite or a sound you have to
add the starting index to the magic number, for example if the sprite index is
:math:`35` and we want to read sprite :math:`8` we have to read chunk
:math:`43`.

Once we have a sprite's offset and length we can read it. The sprite has its
own header consisting of two words followed by an array of up to :math:`64`
words. The first word is the index of the left-most non-empty column, the
second word is the index of the right-most column. The array is of variable
length and contains the offsets to the head of the drawing instruction list of
each column; the first array element is the offset to the drawing instruction
list of the left-most non-empty column, the last array element is the offset
for the right-most non-empty column, and evey element in between belongs to the
column after the previous one. All these offsets are relative to the beginning
of the sprite, not the VSWAP file. method

The number of instruction offsets can be computed as follows: `last_column -
first_column + 1`. The index of the beginning of the pixel data within the
sprite can thus be found as follows

.. code::

	(last_column - first_column + 1 + 2) * sizeof(word)

Here is a schematic of a sprite chunk

.. code::

	Word = first_column
	|
	Word = last_column
	|
	Word = offset[0] -> |W|W|W| ... |W|W|W|
	:
	Word = offset[n] -> |W|W|W| ... |W|W|W|
	|
	Byte = data
	:
	Byte = data

A `W` means `word`, a `B` means `byte`, a `- ` means "is" and a `->` means
"points to" or "is an offset to", offsets are relative to the beginnig of the
chunk. The data stands to any remaing data that's in the sprite, regardless of
what it represents. It is given in bytes, because that's how the pixels will be
read, but the column instructions are three *words*, so take care to read three
words or six bytes, not three bytes. method

To fill the image with pixels we fill the entire image with transparency (byte
`0xFF`). Next we iterate over the non-empty columns. Here the variable `x` will
refer to the index of the current column, it gives us the horizontal position
of the pixel. The vertical position is derived from the drawing instructions:
the first word divided by two is the lower starting point of the pixel
sequence, the third word is the upper end point of the sequence (columns are
drawn from bottom to top). If the first word is `0x0000` it means the end of
the column has been reached and we can advance `x` to the next one. The middle
word is used to reference which pixels to use, but oddly enough it is not
necessary.  method

All that's missing now is how which pixels to draw onto the sprite. Sprites use
a sort of RLE-compression: in the compressed sprite data each byte after the
instruction offsets is a pixel sequence and the n-th sequence belongs to the
n-th instruction. The extents of the instruction tell us how many pixels from
that sequence to draw. After an instruction has been executed move on to the
next pixel. Here is the pseudocode

--------------------------------------------------------------------------------

:Constants: `transparency = 0xFF`

:Prerequsites:
 - `chunk`: pointer to the compressed chunk as a byte sequence
 - `first_colum`: index of the first column (within range :math:`[0, 63)`, less
   than last_column)
 - `last_colum`: index of the last column (within range :math:`(0, 63]`,
   greater than first_column)
 - `offsets`: offsets of the column drawing instructions
 - `i`: `(last_column - first_column + 1 + 2) * sizeof(word)` 
 - Must allocate enough space to hold decompressed sprite (:math:`64 \times 64`
   bytes)

:Code:
 1) Fill entire sprite with the colour for transparency
 2) Make pointer to word `column_offset_reader` and set it to the first column
    instruction offset
 3) For (word `column = first_column`, while `column <= last_column`,
    iterate `++column`)

     1) Make pointer to word `drawing_instruction` and set to `chunk` +
        value of `column_offset_reader` (as word)
     2) Make integer `idx = 0`
     3) While `drawing_instruction[idx] != 0x0000`

         1) For (word `row = drawing_instruction[idx+2] / 2`,
            while `row < drawing_instruction[idx] / 2`, iterate `++row`)

             1) `result[column + (63 - row) * 64]` = `chunk[i]`
             2) `++i`
         2) `idx += 3`
 4) Advance `column_offset_reader` by one word

--------------------------------------------------------------------------------

Now about the second word of the instruction; rather than using the above
method to get the pixel sequence it is possible to use that word. Use the
numeric value of the word plus the current row as the offset from the beginning
of the compressed chunk. As far as I can tell both ways yield the same result,
so I don't know which one to prefer. If in doubt go with this one though, just
in case that there is a weird exception somewhere out there. Here is the
modified pseudocode from above

--------------------------------------------------------------------------------

:Code:
 1) ...
 2) ...
 3) ...

     1) ...
     2) ...
     3) ...

         1) ...

             1) `result[column + (63 - row) * 64] = 
                chunk[drawing_instruction[idx+1] + row]`
         2) ...
 4) ...

--------------------------------------------------------------------------------

We don't need the variable `i` anymore, and so we don't increment it either.


Interpreting sprites
--------------------

Sprites use the same palette as bitmap pictures, but the order in which pixels
are stored is different. If you have been following the above instructions the
sprite will be flipped horizontally, i.e. upside-down. This means the first row
in the raw byte data is the last row in the RGB data, the second row is the
second-to-last and so on. Columns are not affected. method



Textures
========

Textures are simple since they are not compressed. Just like sprites they are
always :math:`64 \times 64` pixels large, but they have no holes. They are also
stored in the VSWAP file, but their type has no offset, the magic number of a
texture is the number of its chunk. To read the texture simply read
:math:`4096` bytes from the chunk verbatim. That fixed number can be replaced
by the chunk length as discussed above for sprites. method

Textures use the same palette as bitmap pictures and sprites as well, but the
order of their pixels is different. The entire image is transposed, meaning that
the row and column of each pixel need to be swapped, like a transposed matrix.
Or in other words, Wolfenstein 3D drew the textures column-first, row-second.
method



Audio
#####

Audio is divided into two categories: sound effect and music tracks and they
share the same files. There is a head file called *AUDIOHED* that contains the
offsets to the the individual chunks as signed 32-bit integers and the chunks
are stored uncompressed in the *AUDIOT* file. method



AUDIOHED
========

There are three types of sound effects: PC speaker, AdLib sound and digitised
sound. Every sound effect exists in every format, although it may be defined
just as empty data, and they are stored in the same order, so the magic number
of a sound effect needs to be mapped to the appropriate chunk. Given the number
of sound effects, which is hard-coded, we can compute the starting offsets of a
format by multiplying a number with the total number of sound effects.

==========  =========
Type           Offset
==========  =========
PC-speaker  :math:`0`
AdLib       :math:`1`
Digitised   :math:`2`
Music       :math:`3`
==========  =========

To get the AdLib version of sound `n` we can thus compute its index as `1 *
number_of_sounds + n`. We can also see that the music chunks follow the sound
effect chunks, and their amount is also hard-coded. We can thus compute the
total number of chunk offsets as follows

.. code::

	number_of_offsets = start_music + number_of_tracks + 1

Where does that extra `1` come from? That's the offset to an imaginary chunk
one past the last chunk. It does not exist, but it is necessary for computing
the length of the last chunk. Computing the length of a chunk is done using the
offset of the next chunk; for the i-th chunk that would be

.. code::

	size[i] = offset[i+1] - offset[i]

It is possible that the size of some chunks is `0`, in this case the chunk can
be seen as non-existent and should be skipped. In fact, all the digitised sound
effects are like this, they are actually stored in the *VSWAP* file instead,
right after the sprite chunks. method



AUDIOT
======

This file is a container for various other files, stored as uncompressed chunks
all lumped together. To find a particular chunk use its offset and size gotten
from the *AUDIOHED* file. What to do with that chunk varies on a type-by-type
basis. There are also tags of the form `!ID!` (`0x21 0x49 0x44 0x21`) the the
end of each file format group, but they are skipped by the offsets anyway.
method

The AdLib sound effects and the music are stored in a format that has been
specifically designed for AdLib sound cards, so unlike the other data it cannot
be simply converted to wave data. One would have to emulate the AdLib hardware,
at least the necessary parts, or use a library. method


Sound effects
-------------

As explained above there are three different types of sound effects and they
are stored ordered by format first and magic number second. Digitised sound is
an exception though: MUSE, the program used by Id, offered that format but
never supported it. The data structures are all there, but they are never used
and the chunks in the AUDIOT file all the length :math:`0`. They are stored in
another file instead.


PC speaker
----------

PC speaker sound effects are a form of *inverse frequency sound format* where
the data bytes represent the inverse of the frequency to play. Here is how the
file is composed: the first four bytes are an unsigned 32-bit integer giving
the length of the sound data, it should be the size of the chunk minus
:math:`7`. It is followed by two bytes of unsigned 16-bit integer giving the
priority of the sound effect. Since in the original engine only one sound could
play at a time a sound will interrupt any sound of lower or equal priority.
Next up is the sequence of data bytes of the length encoded in the first four
bytes. Finally one single byte is used to terminate the file, it is usually
(always?) :math:`0x00`. The file has therefore :math:`7` bytes of non-sound
data (length, priority and terminator).  There is no file name encoded, so the
file can only be accessed using the magic number of the sound effect.

Each byte (unsigned 8-bit integer) of the audio data sequence represents a
certain sound frequency measured in *Hz*. The frequency can be computed this
way:

.. code::

	frequency = 1193181 / (value * 60)  // for value != 0
	          = 0                       // for value == 0

The number `1193181` has the hexadecimal value `0x1234DD`. The refresh rate of
the speaker is :math:`140` Hz, so each instruction lasts :math:`\frac{1}{140}`
seconds. Also keep in mind that multiplying a byte value by :math:`60` can
exceed the range of an :math:`8`-bit integer, so the computation has to be done
at least using :math:`16` bits.

============  ==========  ==============================================
Data type     Name        Description                           
============  ==========  ==============================================
Uint32        length      Length of sound data, chunk length - :math:`7`
Uint16        priority    Higher priority wins                  
Byte[length]  data        Actual audio data                     
Uint8         terminator  Unused by the game                    
============  ==========  ==============================================


Interpreting the data
~~~~~~~~~~~~~~~~~~~~~

Aside from the raw audio data there is no playback information stored in the
file, everything is hard-coded. Since the PC speaker was not able to play
different tones many developers used a trick called *pulse-width modulation* to
create the illusion. The frequency perceived by the listener is created by
precisely controlling short bursts of audio pulses. Explaining the mathematical
properties would be beyond the scope of this document, so I'll refer instead to
its [Wikipedia article](http://en.wikipedia.org/wiki/Pulse-width_modulation).

Each byte tells us how long the the phase needs so be. First we read a byte and
muliply its numeric value by :math:`60` (hard-coded number). This lets us
compute the length of the phase

.. code::

	tone         = input_byte * 60
	phase_length = sample_rate * (tone / 1193181) * 1/2

The *sample rate* depends on how precisely we want to sample the data. Higher
numbers are more precise, but take up more space. We also need to make sure the
sample rate matches the sample rate of our playback, i.e. it is the number of
samples played per second. A value of :math:`40,000` is adequate.

The formula works as follows: looking at the second formula we compute the
inverse of the frequency we want to simulate. This means a higher frequency
will have a shorter duration than a lower one. This inverse frequency is
multiplied by the sample rate; frequencies are measured in Hz, which is just
another way of writing :math:`\frac{1}{s}`, i.e. one per second of something,
so an inverse frequency is a duration, measured in seconds. The sample rate is
measured in *samples/second* and by multiplying it with the duration we get the
number of samples to generate. Finally we divide by two because we need to flip
back-and forth between high and low volume at the half-point mark.

Now it's time to write the sample bytes. How many samples should be written per
byte depends on the selected sample rate as well as the original playback rate
of :math:`140` Hz.

.. code::

	samples_per_byte = sample_rate / 140

For each byte written we also keep track of the "ticks": each written byte
increments the counter, and if the ticks have reached the phase length we flip
the sign and reset the counter. A tone of :math:`0` interrupts everything, it
writes the neutral sound (:math:`128`) and keeps the tick counter at :math:`0`.
The byte written is :math:`128` plus the volume level of the simulated speaker.
This level can be chose arbitrarily, as long as it's less or equal to
:math:`127.` 

Here is the pseudocode:

--------------------------------------------------------------------------------

:Constants:
 - `base_timer = 1193181`
 - `pcs_rate   =     140` (playback rate of PC speaker)
 - `volume     =      20` (arbitrarily chosen, must be :math:`≤ 127`)

:Prerequisites:
 - `source`: pointer to the start of the input stream as bytes.
 - `destination`: pointer to the start of the decompressed output stream as
   bytes
 - `pcs_length`: length of the decompressed data sequence in words
 - `sample_rate`: how many samples to play back per second

:Side effects: The destination buffer will be allocated and filled with data

:Code:
 1) Make new variable `samples_per_byte = sample_rate / pcs_rate`
 2) Make new variable `wav_length = pcs_length * samples_per_byte *
    sizeof(byte)`
 3) Allocate memory to `destination` of length `pcs_length * samples_of_bytes
    *sizeof(byte)`
 4) Make new pointers `read` and `write` and set them to `source` and
    `destination` respectively
 5) Make new signed integer variable `sign = -1`
 6) Make new unsigned integer variable `phase_tick = 0`
 7) While `pcs_length > 0`

     1) Make new variable `tone = (value of read) * 60`, advance `read` by
        one byte
     2) Make new variable `phase_length = sample_rate * (tone / base_timer) *
        1/2`
     3) For (`int i = 0`, while `i < samples_per_byte`, iterate `++i`)

         1) If `tone != 0`

             1) Write `(128 + sign * volume)` to `write`, advance `write`
             2) If `phase_tick >= phase_length`

                 1) `sign *= -1`
                 2) `phase_tick = 0`
                 3) `++phase_tick`
         2) Else

             1) `phase_tick = 0`
             2) Write 128 to `write`, advance `write`
     4) `--pcs_length`

--------------------------------------------------------------------------------

Bytes are in this document equivalent to unsigned 8-bit integers, so it might
look conflicting that we use a signed integer and use it for multiplication.
However, since the neutral sound is :math:`128`, the middle of the 8-bit value
range, it doesn't matter in C. For other languages this might not necessarily
hold true though, so make sure it is well-defined.


AdLib
~~~~~

AdLib sounds are written to specifically talk to the AdLib sound card. It
starts with a header of six bytes: the first four bytes are an unsigned 32-bit
integer for the *length* of the sound data in bytes, the remaining two bytes
are the *priority*, similar to the priority for PC speaker sound.

Then comes the relevant part: :math:`16` bytes of instrument settings followed
by a byte for the octave number and then the data bytes with the length from
above.

Finally we have a footer consisting of a terminator byte, not used by the game,
and a null-terminated ASCII string for the file name, not used either.

============  ===========  ========================
Data type      Name        Description             
============  ===========  ========================
Uint32        length       Length of the sound data
Uint16        priority     Higher priority wins    
Byte[16]      instrument   Instrument settings     
Byte          octave       Octave to play notes at 
Byte[length]  data         Actual audio data       
Uint8         terminator   Unused by the game      
Char[]        file name    Null-terminated string  
============  ===========  ========================

The instrument settings are as follows:

=========  =======  ============  ========================================
Data type  Name     OPL register  Description                             
=========  =======  ============  ========================================
Uint8      mChar          `0x20`  Modulator characteristics               
Uint8      cChar          `0x23`  Carrier characteristics                 
Uint8      mScale         `0x40`  Modulator scale                         
Uint8      cScale         `0x43`  Carrier scale                           
Uint8      mAttack        `0x60`  Modulator attack/decay rate             
Uint8      cAttack        `0x63`  Carrier attack/decay rate               
Uint8      mSus           `0x80`  Modulator sustain                       
Uint8      cSus           `0x83`  Carrier sustain                         
Uint8      mWave          `0xE0`  Modulator waveform                      
Uint8      cWave          `0xE3`  Carrier waveform                        
Uint8      nConn          `0xC0`  Feedback/connection (usually ignored and
                                  set to :math:`0`)
Uint8      voice            none  unused by game                          
Uint8      mode             none  unused by game                          
Uint8[3]   padding          none  pad instrument definition up to :math:`16`
                                  bytes
=========  =======  ============  ==========================================

Sound effects are played on channel :math:`0` because the other channels of the
sound card are reserved for music; the replay rate is :math:`140` Hz. The
octave value is written to AdLib register `0xB0` and it must be computed to
following way to prevent it from interfering with other bits stored in the
register

.. code::

	block = (octave & 7) << 2       // 7=00000111b
	regB0 = block | other_fields

The audio data consists oft he raw bytes to send to register `0xA0` and the
byte `0x00` means silence. Silence can be achieved by setting the fifth bit
(hexadecimal `0x20`) to `0x00` in register `0xB0`. Here is the pseudocode for
playback:

--------------------------------------------------------------------------------

:Constants:
 - `block   = (octave & 7) << 2`
 - `note_on = 0x20`

:Prerequisites: Byte sequence of audio data to read

:Code:
 1) Make boolean variable `note` and set to false
 2) Make byte variable `next_byte`
 3) While there is data to read

    1) Read `next_byte`
    2) If (`next_byte == 0x00)`

       1) Set register `0xB0` to `block`
       2) Set `note` to false
    3) Else

       1) Set register `0xA0` to `next_byte`
       2) If (`note == false`)

          1) Set register `0xB0` to (`block | note_on`)
          2) Set `note` to true
    4) Wait until next tick (playmback rate :math:`140` Hz)

--------------------------------------------------------------------------------

The original code also checked if the next byte was equal to the previous one,
and if so it kept playing the same note instead of sending the same data to the
sound card again.


Digitised
~~~~~~~~~

Digitised sound effects, such as voices or gun shots are stored in the VSWAP
file. That file has been discussed in the *sprites* section, so refer there for
information on how to read the file. The data chunks are raw PCM data, played
back at a sample rate of :math:`7000` Hz, mono sound and eight bytes per
sample.

Where it gets complicated is that some audio files are split over multiple
successive audio chunks; one example is the very first effect ("Achtung!),
which is split over the first and second chunk. That is also why there are more
sound effect chunks than there are sound effects (:math:`120` instead of
:math:`46`). We must read the last chunk of the VSWAP file, it contains the
audio list consisting of pairs of words; the first word is the index of the
biginning of the first audio chunk, the second word is the length of the
complete audio chunk. This means that the global list of lengths and offsets
detailed in the *sprites* section is only needed for the offsets.

The index of a sound effect chunk can be learned by adding the effect's index
from the audio list and the sound start index. This gives us the global file
index of the first chunk of the sound effect. Using this global index we can
find the offset of the chunk in the lgobal list. The length of the total audio
sequence is the length from the audio list.

The number of digitised sound effects is the length of the audio list divided
by four. The length of the list is the length of the VSWAP file minus the
offset of the list, i.e. the list is the very last chunk of the file.


Music tracks
------------

The music format is *WLF*, which is essentially type-1 *IMF* whith a playback
rate of :math:`700` Hz instead of :math:`560` Hz. Here is how a *WLF* file is
composed:

============  ==========  ======================================
Type          Name        Description                           
============  ==========  ======================================
Uint16        Length      Length of the sound data              
Byte[length]  Sound data  The sound data to play                
Byte[]        Metadata    Arbitrary metadata, unused by the game
============  ==========  ======================================

The sound data consists of byte quartets of the following form:

====  ==============
Type  Description   
====  ==============
Byte  AdLib register
Byte  AdLib data    
Word  Delay         
====  ==============

There is also an optional footer that contains metadata that will not be used
for playback but can be used by an audio editor:

========  =======  ==================================
Type      Name     Description
========  =======  ==================================
Unint16   ???      Unknown
Char[16]  Title    Title of the song
Char[64]  Remarks  Comments, usually source file name
Char[6]   cProg    Unknown, maybe from the compiler
========  =======  ==================================



Levels & Maps
=============

Levels are laid out on a :math:`64 \times 64` tile-based square map. This size
is not hard-coded into the game, so one should not make assumptions about the
level's size, instead the size should be read from the map file. Although there
are no official levels of any other size an engine or interpreter should be
able to support custom-made maps of different size. Each level in the game
actually consists for three maps overlaying each other:

Architecture
    The first map contains information about the level's architecture, i.e.
    walls, doors and floors.

Objects
    The second map contains the level's objects, i.e. enemies, decorations and
    pick-ups.

Other
    The third map contains other data and is not used in this game, it's a
    leftover from earlier Id titles.

These three individual maps together form the level the player will be playing.
Usually when speaking about maps one means the entire level, but here we will
maintain this distinction to avoid confusion or ambiguity.

Each of the tiles in a level describes a three-dimensional cube in the game
world with :math:`64` units in length to both sides and :math:`64` units in
height (i.e.  a cube in 3D world space).


MAPHEAD
-------

The file starts with the signature 16-bit integer `0xABCD` (represented as
`0xCD 0xAB` bytes in the file). This signature appears always to be the same,
but we should not make any assumptions; it is used as the signature for the
RLEW compression algorithm. The file is described by the structure
`mapfiletype` in the original source code.

Next are exactly :math:`100` 32-bit signed integer values containing the header
offsets of the actual levels, that amount is hardcoded into the source. Not all
of these 32-bit numbers have meaningful values, only the first n do, where n is
the total amount of levels in the game, i.e. :math:`10` in the shareware
version and :math:`30` or :math:`60` in the full version. The remaining numbers
are all padding with `0x00000000` as their value. This means the level offsets
are stored in a `nul`-terminated 4-byte array with a fixed length of
:math:`100.` 

The last remaining byte always appears to be be `0x00` and it's called the
`tileinfo` in the original source code and is declared as an array of
unspecified size of type `byte`. The type `byte` is a typedef for `unsigned
char` and equal to an 8-bit integer on the target architecture of Wolfenstein
3D's original code. It appears to be a leftover from the map format of previous
Id Software games that did use it.

Note that there is no information in this file as to how many levels there are
in the game. This information would have to be calculated from the file's size
itself. To compute that number one would have to step through the list of
header offsets until reaching the first offset that's `0x00000000` (start of
the padding). The number of steps is equal to the number of levels.

=============  ==========  ================================================
Name           Type        Description                                   
=============  ==========  ================================================
Signature      Word        Used for RLEW decompression, usually `0xCD 0xAB` 
Header offset  Int32[100]  Offsets into the gamemaps file                
Tile info      Byte        Unused, usually `0x00`                          
=============  ==========  ================================================


GAMEMAPS
--------

This file contains the actual information about the levels and their individual
maps. A level is made from a *level header*, which describes where to find the
level's maps, their compressed sizes, the size of the level and finally the
name of the level.

The header can be found using the offset from the *MAPHEAD* file as an absolute
value, i.e. relative to the start of the file. From there on the header is
stored as an uncompressed sequence of raw information.

The first three values are 32-bit signed integer values each. The first one is
holding the offset to the level's architecture map, the next value is the offset
to the level's object map and the third value is the offset to the level's logic
map. All values are absolute offsets from the beginning of the file, not
relative offsets from the header or relative to each other.

The next three values are unsigned 16-bit integer values describing the
Carmack-compressed length in bytes of each map; this is important because the
maps are lumped together adjacent to each other with no separator. Their order
is again first architecture, then objects and then logic.

Next are two unsigned 16-bit integers describing the width and height of the
level, in that order. The size appears to always be :math:`64 \times 64`, but
since it's not hardcoded it should not be assumed.

Finally :math:`16` characters, 8-bit ASCII each, form the level's name. In the
original implementation the characters are stored in an array of type `char`
with unspecified size. This is the standard way of storing ASCII strings in C,
but the string needs to be terminated with `\0` (the null character). In the
file any remaining bytes are filled with `\0`, but in the code there is nothing
to ensure that the string is indeed properly terminated, leaving a possibility
for an error to happen.

==============  =========  ===========================================
Name            Type       Description                                
==============  =========  ===========================================
Map offset      Int32[3]   Offset of the three maps, absolute from the
                           beginning of the file
Carmack length  Uint16[3]  Length of the Carmack-compressed map       
Width           Uint16     Width of the level                         
Height          Uint16     Height of the level                        
Level name      Char[16]   Name of the level                          
==============  =========  ===========================================

The first word of a map is the most north-western tile, and each column is one
more tile to the east, each row one tile to the south.


Extracting the maps
-------------------

Maps are compressed using the RLEW compression and then compressed on top of
that using Carmack compression. To decompress them one has to first
Carmack-decompress the data and then RLEW-decompress it. For Carmack
compression one can find the decompressed length encoded into the compressed
map as the fist word, it is given in bytes. This means the pointer to the
compressed sequence must be advanced by one before starting the decompression.
For some reason the pointer to the Carmack-decompressed but still
RLEW-compressed sequence must be advanced by one word as well; could be a
leftover from a previous map format.  The size of the uncompressed RLEW data is
hardcoded as `64*64*2` bytes or :math:`4096` words. Since the size is also
stored in the map format it might be a better idea to use that value instead
and allow levels of different size for mods. The RLEW tag can be found in the
MAPHEAD file as described above.



Appendix
########


Known bugs and limitations
==========================

1. A map needs at least one enemy, one piece of treasure and one secret door,
   or else the game will crash. This is the result of the game trying to
   calculate the percentage the player has picked up and ending up dividing by
   zero.


Distributions of the game and magic numbers
===========================================

Different versions of the game assign different key numbers to the graphics.
Each graphic can be identified using an integer "key" magic number and that
number depended on what the graphics artists produced. This means the
programmers would have had to keep a list of key numbers and always change their
code when the graphics changed. Instead the software used by the artists
produced alongside the data files also a header file that mapped each number to
a macro, so the developers could simply use the macro and the generated header
would map the macro to the correct number.

The problem with this is that each build was only suitable for a particular
distribution of the game, like shareware, registered, Japanese or Spear of
Destiny. The simple solution is to use `#ifdef` directives and set the version
at compile time, which is an acceptable solution when building for a particular
release, but ill-suited for a source port that needs to be as compatible as
possible. I propose the following solution that moves the version detection
from compile-time to run-time.

There will be one global header file that has a universal mapping that assigns a
number to any image that might exist in any distribution. It is not necessary
for it to be compatible to any existing distribution. The programmers will use
these global macros then. For each distribution there will be a mapping that
maps the universal macro to the distribution's corresponding key number. At
run-time when the program starts determine the distribution and assign a global
mapping variable to be the mapping for that distribution. The mapping could for
example be done using an array where the universal macro is the index and the
distribution's key the value. Trying to access an image that does not exist in
that particular distribution could be mapped to an invalid number such as -1.


References
==========

The following sources were used for reference and to guide me in the right
direction:

- `Wolfenstein 3D source code
  <https://github.com/id-Software/wolf3d>`_
- `Chocolate Wolfenstein 3D source code
  <https://github.com/fabiensanglard/Chocolate-Wolfenstein-3D>`_
- `Wolfensein 3D on Modding Wiki
  <http://www.shikadi.net/moddingwiki/Wolfenstein_3-D>`_
- `Some guy's abandoned attempt to understand the game data 
  <http://devinsmith.net/backups/bruce/wolf3d.html>`_

