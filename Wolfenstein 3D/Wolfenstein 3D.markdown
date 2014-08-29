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
	| |- Carmack compression
	|
	- Data files
	| |- Graphics
	| |- Audio
	| |- Maps
	|    |- MAPHEAD
	|    |- GAMEMAPS
	|
	- Known bugs and limitations


Conventions and nomenclature
----------------------------
Decimal integer numbers are given as regular numbers or rarely suffixed with "d", binary numbers are suffixed with "b", octal numbers with "o" and hexadecimal numbers are either suffixed with "h" or prefixed with "0x" or "#". Example:

	13 = 13d = 1011b = 15o = Dh = 0xD

All numbers are written in the standard big-endian notation used in the English language. This means the left-most digit is the most significant one and a number like 1011b is computed as (1 * 2^3 + 0 * 2^2 + 1 * 2^1 + 1 * 2^0). If the endianness needs to be explicitly stated it will be given as (BE) for big-endian and as (LE) for little-endian.

Notes are written as "//NOTE", tasks are written as "//TODO" and bugs in the original implementation are written as "//BUG", all without the quotation marks. This allows using the editor's search function to quickly jump to these points. An optional colon (:) can be suffixed, so make sure the editor ignores these.


2D or 3D
--------
Despite its name Wolfenstein 3D is not a true 3D game; the game's data and simulation all happen in a flat 2D space on a strict grid, while the rendering appears to take place in a 3D world to the player. In this document the game world (called "world space") will be treated as if it was actually a three-dimensional space, since that is what the player is experiencing. On the other hand, simulation space, where all the game's actual mechanics are implemented, will be treated like a two-dimensional plane, since that is the way the game works.


Compression algorithms
----------------------

###RLEW compression###
A variant of RLE (Run length Encoding) that uses words instead of bytes as the underlying unit.

###Carmack compression###

###Carmack compression###


Data files
----------
endianness: little-endian, byte-size: 1 byte = 8 bits, word-size: 1 word = 2 bytes = 16 bits

The game assets for WS3D are stored in various files with the extension WLn, where n is either 1, 3 or 6 depending on the version of the game and stands for the number of episodes (1 = shareware, 3 = early three-episode full version, 6 = newer full version). For simplicity the file extension will be omitted form here on unless a specific file extension is needed. The assets are distributed as follows:

	Graphics -> VGADICT, VGAHEAD, VGAGRAPH
	Audio    -> AUDIOHED, AUDIOT
	Maps     -> MAPHEAD, GAMEMAPS (or MAPTEMP in some versions?) //NOTE: do such files even exist in official releases?

(//TODO: all of this needs to be properly confirmed)

The header files contain information about the structure of the actual asset files

### Graphics ###

### Audio ###

### Levels & Maps ###
Levels are laid out on a 64 x 64 tile-based square map. This size is not hard-coded into the game, so one should not make assumptions about the levels's size, instead the size should be read from the map file. Although there are no official levels of any other size an engine or interpreter should be able to support custom-made maps of different size. Each level in the game actually consists for three maps overlaying each other:

- The first map contains information about the level's architecture, i.e. walls, doors and floors.
- The second map contains the level's objects, i.e. enemies, decorations and pick-ups.
- The third map contains the level's logic data, such as waypoints.

These three individual maps together form the level the player will be playing. Usually when speaking about maps one means the entire level, but here we will maintain this distinction to avoid confusion or ambiguity.

Each of the tiles in a level describes a three-dimensional cube in the game world with 64 units in length to both sides and 64 units in height (i.e. a cube in 3D world space).

#### MAPHEAD ####
The file starts with the signature 16-bit integer 0xABCD (represented as 0xCD 0xAB bytes in the file). This signature appears always to be the same, but we should not make any assumptions; it is used as the signature for the RLEW compression algorithm. The file is described by the structure `mapfiletype` in the original source code.

Next are exactly 100 32-bit (4 Byte) values containing the header offsets of the actual levels. Not all of these 32-bit numbers have meaningful values, only the first n do, where n is the total amount of levels in the game, i.e. 10 in the shareware version and 30 or 60 in the full version. The remaining numbers are all 0x00000000.

The last remaining byte always appears to be be 0x00 and it's called the `tileinfo` in the original source code and is declared as an array of unspecified size of type `byte`. The type `byte`is a typedef for `unsigned char` and equal to an 8-bit integer on the target architecture of Wolfenstein 3D's original code. This value is not used anywhere in the original code, so I assume it is simply ignored.

Note that there is no information in this file as to how many levels there are in the game. This information would have to be calculated from the file's size itself. To compute that number one would have to step through the list of header offsets until reaching the first offset that's 0x00000000 (start of the padding). The number of steps is the number of levels.

#### GAMEMAPS ####
This file contains the actual information about the levels and their individual maps. A level is made from a *level header*, which describes where to find the level's maps, their sizes, the size of the level and finally the name of the level.

The header can be found using the offset from the MAPHEAD file as an absolute value. From there on the header is stored as an uncompressed sequence of raw information.

The first three values are 32-bit unsigned integer values each. The first one is holding the offset to the level's architecture map, the next value is the offset to the level's object map and the third value is the offset to the level's logic map. All values are absolute offsets from the beginning of the file, not relative offsets from the header or relative to each other.

The next three values are unsigned 16-bit integer values describing the length of each map; this is important because the maps themselves are compressed, so the length of a given map is not the same as its size. Their order is again first architecture, then objects and then logic.

Next are two unsigned 16-bit integers describing the width and height of the level, in that order. The size appears to always be 64 x 64, but since it's not hardcoded it should not be assumed.

Finally 16 characters, 8-bit ASCII each, form the level's named. In the original implementation the characters are stored in an array of type `char` with unspecified size. This is the standard way of storing ASCII strings in C, but the string needs to be terminated with `\0` (null character). In the file any remaining bytes are filled with `\0`, but in the code there is nothing to ensure that the string is indeed properly terminated, leaving a possibility for an error to happen.

#### Extracting the maps ####
Maps are compressed using the RLEW compression and then possibly compressed on top of that using Carmack compression.


Known bugs and limitations
--------------------------

1. A map needs at least 1 enemy, 1 piece of treasure and 1 secret door, or else the game will crash. This is the result of the game trying to calculate the percentage the player has picked up and ending up dividing by zero.

