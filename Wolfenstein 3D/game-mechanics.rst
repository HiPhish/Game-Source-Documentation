.. default-role:: code

#############################
Wolfenstein 3D game mechanics
#############################

.. contents:: Table of Contents
   :depth: 3

The game rules have been derived mostly from the official iOS port by Id, which
in turn is based on the *Wolfenstein 3-D Redux* port. The rules are effectively
the same as for the original PC release, but the technical details might be
different.

Time is measured in *ticks* from now on. In the original implementation one
tick was intended to last :math:`\frac{1}{70}`-th of a second and the game was
intended to run at a rate of one ticks per frame or :math:`70` frames per
second.

Mathematics
###########

:TODO: This whole section might be superfluous

To faithfully recreate the gameplay of Wolfenstein 3D one has to understand how
the developers worked around the technical limitations of the original hardware.
Even if we were to use proper modern techniques we should at least know under
what quirks the original implementation had.


Fixed point instead of floating point
=====================================

The processor of the target hardware, the Intel 286 and 386, did not natively
support floating point operations, they would have to be implements in
software, which would have been too slow for gameplay. The solution was to use
fixed-point arithmetic by using integers. That would give the programmers half
the bits on both sides of the radix point. Truncating the fractional part of
such a number can be done by right-shifting by half the type's size. Here is an
example using a 32-bit integer

===========  =  =  =  =  =  =  =  =  ==  ==  ==  ==  ==  ==  ==  ==
:math:`2^n`  7  6  5  4  3  2  1  0  -1  -2  -3  -4  -5  -6  -7  -8
       bit   0  0  1  0  0  0  1  0   0   0   1   0   0   0   1   0
===========  =  =  =  =  =  =  =  =  ==  ==  ==  ==  ==  ==  ==  ==

.. math::
    1 \times 2^5 + 1 \times 2^1 + 1 \times 2^{-3} + 1 \times 2^{-7}
      = 32 + 2 + 0.125 + 0.0078125
      = 34.1328125

The number can be treated like an integer for the most part. In this document I
will treat these number as floating point anyway for the sake of simplicity. The
decision whether to adopt floating-point numbers of stick with fixed-point is up
to the implementation.


Enumerations and constants
==========================

The game has a number of hard-coded constants for gameplay.

============  =======  ==============  =====================================
Name          Type     Value           Description                          
============  =======  ==============  =====================================
`FLOATTILE`   Float    `65536.0f`      ???                                  
`TILEGLOBAL`  Integer  `0x10000`       ???                                  
`HALFTILE`    Integer  `0x08000`       :math:`0.5` as fixed-point decimal           
`MINDIST`     Integer  `0x05800`       ???                                  
`STEP`        Float    `0.0078125f`    How many degrees are one step        
`STEPRAD`     Float    `0.000136354f`  How many radians are one step        
`MAX_GUARDS`  Integer  `255`           Maximum number of enemies in the game
`SPDPATROL`   Integer  `512`           Patrolling speed of humans           
`SPDDOG`      Integer  `1500`          Patrolling speed of dogs              
============  =======  ==============  =====================================

These are the enumerations defined in the code

.. code::

	quadrant    = {first, second, third, fourth}
	direction_8 = {east, north_east, north, north_west, west, south_west, south, south_east}
	direction_4 = {east,             north,             west,             south            }

All enumerations are mapped to integer values as defined in the C standard: the
first element has value 0 and ever successive element has a value :math:`+1`
greater than the previous one. In the following enumeration elements will be
treated as equivalent to integers.


Random numbers
==============

Wolfenstein 3D does not have actual random numbers, instead it uses a table of
:math:`256` of predefined numbers and picks one of them. The result is good
enough to feel reasonably random to the player.

===   ===   ===   ===   ===   ===   ===   ===
  0     8   109   220   222   241   149   107
 75   248   254   140    16    66    74    21
211    47    80   242   154    27   205   128
161    89    77    36    95   110    85    48
212   140   211   249    22    79   200    50
 28   188    52   140   202   120    68   145
 62    70   184   190    91   197   152   224
149   104    25   178   252   182   202   182
141   197     4    81   181   242   145    42
 39   227   156   198   225   193   219    93
122   175   249     0   175   143    70   239
 46   246   163    53   163   109   168   135
  2   235    25    92    20   145   138    77
 69   166    78   176   173   212   166   113
 94   161    41    50   239    49   111   164
 70    60     2    37   171    75   136   156
 11    56    42   146   138   229    73   146
 77    61    98   196   135   106    63   197
195    86    96   203   113   101   170   247
181   113    80   250   108     7   255   237
129   226    79   107   112   166   103   241
 24   223   239   120   198    58    60    82
128     3   184    66   143   224   145   224
 81   206   163    45    63    90   168   114
 59    33   159    95    28   139   123    98
125   196    15    70   194   253    54    14
109   226    71    17   161    93   186    87
244   138    20    52   123   251    26    36
 17    46    52   231   232    76    31   221
 84    37   216   165   212   106   197   242
 98    43    39   175   254   145   190    84
118   222   187   136   120   163   236   249
===   ===   ===   ===   ===   ===   ===   ===

An unsigned 32-bit integer is used as the index for for picking a number from
the table. Initialising the table means setting the index to a number. It can
be done in two ways, fixed and randomised. Fixed means simply setting it to
`0`; randomised means setting it to `time(NULL) & 0xFF` where `time()` is the C
standard time function. The table is always randomised and it is initialised
only once when the game starts.

Retrieving a random number is done by incrementing the index and then `AND`-ing
it bitwise with `0xFF`, the corresponding number is picked from the table.


Functions and macros
====================

There are a number of functions and macros defined. The first batch is standard
stuff

===========  ==========================
`max(x, y)`  Maximum of two numbers
`abs(x)`     Absolute value of a number
===========  ==========================

The following are converting between world-space and tile-space; to understand
them we need to know that positions are stored as 32-bit integers representing
fixed-point decimals. Shifting a number by `TILESHIFT` (:math:`=16`) left turns
an integer into a decimal and shifting right turns a decimal into an integer.

==================  ==========================================================
`tile_to_pos(a)`    Converters tile coordinate to world coordinate; make `a`
                    into fixed-point, add `HALFTILE`.
`pos_to_tile(a)`    Converts world coordinate to tile coordinate; make `a`
                    into an integer.
`pos_to_tile_f(a)`  Converts world coordinate to floating-point tile
                    coordinate; divide `a` by `FLOATTILE`.
==================  ==========================================================


Angles & trigonometry
=====================

The limited precision offered by fixed-point arithmetic forced the developers
to work around it. Angles are given in *steps* and can be converted to degree
and radians. See the table of constants for the conversion ratios. Here is the
list of pre-defined angles in steps:

=======  =====
Degrees  Steps
=======  =====
    5        0
    1      128
    6      768
   15     1920
   22.5   2880
   30     3840
   45     5760
   67.5   8640
   90    11520
  112.5  14400
  135    17280
  157.5  20160
  180    32040
  202.5  25920
  225    28800
  247.5  31680
  270    34560
  292.5  37440
  315    40320
  337.5  43200
  360    46080
=======  =====

All of these numbers could be computed at runtime from one base value, but they
were manually pre-computed and hard-coded. Conversion between steps and angles
works as follows:

.. code::

	step_to_radian(a) = (`a` * PI) / `angle_180`
	radian_to_step(a) = (`a` * `angle_180`) / PI

	step_to_degree(a)   = (float)(a) / angle_1
	step_to_degree_f(a) = (a) / (float)angle_1
	degree_to_step(a)   = (a) * angle_1

The first cast prevents precision loss during division, the second cast makes
the result of the division itself a floating-point number.

After defining these discrete angles we build tables of trigonometric values.
The sine- cosine and tangent table simply hold the respective values for each
angle. Finally we have a number of angle-related functions

.. code::

	normalize_angle(a) : convert any integer to a number between 0 and 360, in steps

To convert an angle to a direction we use the *floor*: an angle always
corresponds to the nearest direction that's below an angle. For instance, an
:math:`89°` angle would correspond to north-east, because it's rounded down to
:math:`45°`.



Levels
######

As discussed in the data formats chapter, levels in Wolfenstein 3D are built
from tiles. A level is usually :math:`64 \times 64` tiles large, but even
though that number is hard-coded into the engine the level files also specify
their size, so from now on the size of the level will be assumed to be variable
between levels, but constant within each level. This means if the level is m x
n tiles large, then all its maps are that large as well and the level will
neither shrink nor grow during gameplay.

Various mathematical operations a carried out on a discrete tile-based basis,
but actual movement takes place in a continuous fashion. We must be able to do
both interchangeably and we will often convert back and forth between tile- and
world coordinates.

Aside from keeping track of all the actors and providing architecture to play
in, levels have three major sub-aspects as well: areas, doors and push-walls.


Anatomy of a level
==================

A level is made of two maps: the *architecture* map and the *objects* map. The
architecture tells us which tiles are doors, areas and walls. The objects map
lists the map objects, such as enemies, power ups or static decoration objects.
Some objects only appear on harder difficulties than others.

A level has the following members:

==============  ========================  ====================================
Name            Type                      Description                         
==============  ========================  ====================================
Size X          Integer                   Horizontal size of the level        
Size Y          Integer                   Vertical size of the level          
File Name       Char[32]                  File name of the level              
Architecture    Word[Size X * Size Y]     Architecture map                    
Objects         Word[Size X * Size Y]     Objects map                         
Other           Word[Size X * Size Y]     Other map                           
Tile Map        Int32[Size X * Size Y]    ?                                   
Spotvis         Byte[Size X * Size Y]     Unused                              
Wall Texture X  Integer[Size X * Size Y]  Horizontal wall texture references  
Wall Texture Y  Integer[Size X * Size Y]  Horizontal wall texture references  
Areas           Integer[Size X * Size Y]  Area numbers                        
Doors           Level Doors type          Doors of the level                  
Player Spawn    Place on Plane type       Spawning point for the player       
Map Name        Char[128]                 Name of the map                     
Music Name      Char[128]                 Name of the music track to play     
Ceiling Colour  Colour3 type              Colour of the ceiling               
Floor Colour    Colour3 type              Colour of the ceiling               
Tile Seen       Byte[Size X * Size Y]     Whether a tile has ever been seen by
                                          the player
==============  ========================  ====================================

The members *Size X* and *Size Y* are my additions. Originally the size of the
level is hard-coded into the code and the arrays always have size :math:`64
\times 64`. That makes it possible for the structure to have predictable size
and is required for setting the size of the arrays at compile type (arrays in C
are second-class objects).

The *Tile Seen* member is used for the automap and was added by Id to later
ports, such as the iOS port. It tells us whether the player has seen a given
tile already. This might be what *Spotvis* was supposed to do.

The *Level Doors* type will be discussed later when we discuss doors. For now
it's enough to know that it keeps track of all the doors in the level and their
status.

The *Place on Plane* type is defined as follows:

==========  =======
Name        Type   
==========  =======
Position X  Integer
Position Y  Integer
Angle       Integer
==========  =======


Loading a level
===============

The structure of the level head and how to extract the maps is described above
in the *file formats* chapter in the *data files* section. I will now assume
the header and the maps are in memory.

We start by looping over the level size. It does not matter whether we process
the architecture- or objects map first, they are not dependent on each other.
All map elements are words, so they will be compared to their numerical value
here. Remember that multi-byte numbers are stored in little-endian order, so
the word `0xCD 0xAB` has the numerical value `0xABCD`.

-------------------------------------------------------------------------------

:Constants: - `NUMBER_OF_AREAS = 37`
            - `AMBUSH_TILE     = 0x6A`
            - `FIRST_AREA      = 0x6B`

:Code:
 For every tile do:

 1) Read the architectural structure from the architecture map and the object
    from the object map
 2) Spawn `object` on tile from objects map
 3) If `structure == 0x0000`
     1) Set level area of this tile to -3 (unknown area)
 4) Else

     1) If `(0x005A <= structure <= 0x005F) || (0x0064 <= structure < 0x0065)`
        (door)

         1) Set the Door flag on the tile and spawn a door
         2) Set level area of this tile to -2 (door)
     2) Else

         1) Set the Wall flag on the tile
         2) Set level area of this tile to -1 (wall)
         3) Assign textures
         4) If `strucure == 0x15`

             1) Set the Elevator flag on the tile
     3) Else if `structure == 0x6A`

         1) Set the Ambush flag on the tile
         2) Set level area of this tile to -3 (unknown area)
     4) Else if `FIRST_AREA <= structure < (FIRST_AREA + NUMBER_OF_AREAS)`

         1) If `structure` == FIRST_AREA`

             1) Set the Secret Level flag on the tile
         2) Set level area of this tile to `(structure - FIRST_AREA)`
     5) Else

         1) Set level area of this tile to -3 (unknown area)

-------------------------------------------------------------------------------

The numbers `0x0064` and `0x0065` stand for elevator doors. We also see that
elevators are just special instances of walls. The index of a wall texture can
be computed from the numerical value of the texture

.. code::

	texture_x = (numerical_value - 1) * 2 + 1
	texture_y = (numerical_value - 1) * 2

After initiating all the tiles we need to fix the unknown ares to prevent
problems from occurring. To this end we attempt to connect every unknown area
to an adjacent area.

-------------------------------------------------------------------------------

:Prerequisites: `area` = table of tile area numbers

:Code:
 1) For integer `x = 1`, while `x < 63`, iterate `++x`
     1) For integer `y = 1`, while `y < 63`, iterate `++y`
         1) If `area[x][y] == -3`
             1) If eastern area `>= 0` set `area[x][y]` to it
             2) Else if western area `>= 0` set `area[x][y]` to it
             3) Else if southern area `>= 0` set `area[x][y]` to it
             4) Else if northern area `>= 0` set `area[x][y]` to it

-------------------------------------------------------------------------------

Finally, we must set up the areas of the doors. We will discuss doors later,
but for now it's enough to know that each door has a member that tracks the
area of either side of the door.

-------------------------------------------------------------------------------

:Prerequisites:
 - `level_doors`: Array of door structures for the current level
 - `level_areas`: Array of the areas for the current level

:Code:
 For every door in the level do:

 1) If the door is a vertical one
     1) Set the areas of the door to the areas west and east  (in that order)
        If the area number is less than 0 set it to 0
 2) If the door is a horizontal one
     1) Set the areas of the door to the areas north and south (in that order)
        If the area number is less than 0 set it to 0

-------------------------------------------------------------------------------

We can now set the ceiling colour to `0x38 0x38 0x38`, or a 32-bit RGBA colour
of `(56 56 56 0)`, and the floor colour to `0x70 0x70 0x70`, or a 32-bit RGBA
colour of `(112 112 112 0)`. These values are hard-coded in the original
engine, but oddly enough they are included in the map format of the iOS release
at offset :math:`10`, first ceiling, then floor and both four bytes in length.


Classes of architecture tiles
-----------------------------

Each tile can have one of the following flags set. It doesn't make sense to
have more than one of them per tile, and the level file format makes it even
impossible, but there is nothing in the engine to prevent it either. The flags
are as follows:

============  ======================
Flag          Description           
============  ======================
Wall          Solid wall            
Pushwall      Pushable secret wall  
Secret        ?                     
Dressing      Unused                
Blocking      Impassable obstacle   
Actor         ?                     
Dead Actor    ?                     
Powerup       Powerup to pick up    
Ambush        Ambush tile for actors
Exit          ?                     
Secret Level  ?                     
Elevator      Exit from this level  
East          Waypoint east         
North-East    Waypoint north-east   
North         Waypoint north        
North-West    Waypoint north-west   
West          Waypoint west         
South-West    Waypoint south-west   
South         Waypoint south        
South-East    Waypoint south-east   
============  ======================

The Dressing and Dead Actor flags are not used by the game, they might be
leftovers from an earlier stage in development when Wolfenstein 3D was meant to
be a more stealth-oriented game.

These flags can be grouped into "classes of tiles" where a tile belongs to that
class if it has one of the flags set. These are the classes:

Solid
   walls, pushwalls or blocking obstacles
blocks move
   walls, pushwalls or actors
waypoints
   any of the waypoints

The *Blocks Move* class is unused by the game.



Areas
=====

Areas are a way of grouping what could be considered "rooms" in a level (there
is no concept of a "room" in the source code, but the player perceives parts of
the levels as rooms). Since areas are defined on the architecture map an area
is always a free tile, never a wall or a door.

Areas are a way of grouping what could be considered "rooms" in a level (there
is no concept of a "room" in the source code, but the player perceives parts of
the levels as rooms). Since areas are defined on the architecture map an area
is always a free tile, never a wall or a door.

Areas can be connected to each other via doors, allowing sound to travel
between them, so an enemy could hear one of its friends being attacked by the
player and rush in to help. Two areas are connected if and only if at least one
door between them is open. The *adjacency* between areas is measured as the
number of open doors directly between them. Usually there is only one door, but
some areas can have multiple doors connecting them and as long as at least one
door is open the areas are connected.

From this we can see that the areas and door form a graph structure where the
areas are vertices and the doors are edges. The original implementation used a
directed graph where it would technically be possible to have one-way doors
that allow sound to travel from one area to the other, but not back. Such doors
don't exist in the game though, and the function for setting the degree of a
node always works both way. For the sake of authenticity I will continue using
a directed graph.

It is also possible for a pair of vertices to have several edges connecting
them; this means that multiple doors can be opened to connect them. One door
could have been opened by the player and another one by an enemy. In the
original source the graph is implemented as an adjacency matrix of type
integer.

To allow the player to hear sound we must keep track of which areas are
connected to the player's current area. This is done via a list of boolean
values where each list item stand for an area and the value is `true` if the
area is connected to and area that's connected to the player. The player's
current area is always connected and the list gets updated every time a door
opens and closes.


Connecting and disconnecting areas
----------------------------------

To connect two areas `a` and `b` increment the adjacency matrix entries `(a,
b)` and `(b, a)`. We have to increment both entries because the graph is
directed.  To disconnect areas decrement their entries instead. If two areas
are connected by multiple doors the entries get incremented for every door,
allowing them to grow beyond 1. This is necessary because enemies might open
other doors on their own.


Initialisation
--------------

To initialise the areas the level has to have been loaded. Then set the
adjacency matrix to the zero-matrix (all doors closed), set the player area
list to all-false, except for the area the player starts in.


Update connections
------------------

Whenever a door is opened or closed or the player moves to a new area we need
to update the connections.

-------------------------------------------------------------------------------

:Code:
 1) Set player area list to all-false, except for area of the player
 2) Connect recursively to the player area

-------------------------------------------------------------------------------

Connecting recursively is done like this

-------------------------------------------------------------------------------

:Prerequisites: `area` = area to connect to
:Constants: `NUM_AREAS` = number of areas in the game (hardcoded 37)
:Code:
 1) For integer `i = 0`, while `i < NUM_AREAS`, iterate `++i`
 2) If `area` and `i` are connected and the player area list for `i` is
    false

    1) Set the player area list for `i` to true
    2) Carry out this routine recursively for area `i`

-------------------------------------------------------------------------------

This routine loops through all the areas connected to the current layer and
connects them to the player. We need the second condition to avoid getting
stuck in an infinite loop.



Doors
=====

Doors have a three-fold purpose: they physically block the player from passing
from one room to another, and they prevent sound from traveling from one are to
another (they don't stop sound from traveling throughout the same area though).
Finally, they block or allow line of sight depending on whether they are closed
or open, but LOS is discussed later.

There is a hard-coded limit of :math:`64` doors per level. This limit makes it
possible for the C compiler to know the size of the door array at compile time,
but the array might only be filled partially if there are fewer doors in the
level.


Anatomy of a door
-----------------

A door is always in one of four states:

=======  ====================================================
State    Meaning                                             
=======  ====================================================
Closing  Has been open and is now in the process of closing  
Closed   Closed door                                         
Opening  Has been closed and is now in the process of opening
Open     Open door                                           
=======  ====================================================

There are several types of doors:

===================  ================  ======
Name                 Description       Number
===================  ================  ======
Normal vertical      Normal door          255
Normal horizontal    Normal door          254
Elevator vertical    Elevator door        253
Elevator horizontal  Elevator door        252
Gold vertical        Needs gold key       251
Gold horizontal      Needs gold key       240
Silver vertical      Needs silver key     249
Silver horizontal    Needs silver key     248
===================  ================  ======

A door has the following structure:

==========  ==========  ================================
Type        Name        Description                     
==========  ==========  ================================
Integer     Position X  Horizontal tile of the door     
Integer     Position Y  Vertical tile of the door       
Boolean     Vertical    Whether this is a vertical door 
Integer     Tic Count   ?                               
Door state  State       Current state of the door       
Integer     Area 1      One area connected by the door  
Integer     Area 2      Other area connected by the door
Door type   Type        Type of the door                
Integer     Texture     Texture of the door             
==========  ==========  ================================

Door textures are stored right after the regular wall textures. They are as
follows in this order

=========  =========  =======  =======  ==========  ==========  ========  ========
regular_h  regular_v  plate_h  plate_v  elevator_h  elevator_v  locked_h  locked_v
=========  =========  =======  =======  ==========  ==========  ========  ========

Plate is the plate on the walls left and right of the sliding door. These two
textures are applied on top of the existing wall texture, effectively hiding it
beneath.


Preparing doors
---------------

The level keeps track of the number of doors, a list of actual doors and a
matrix of possible doors. The list is implemented as an array of door
references with hard-coded size of :math:`256`, but there is no particular
reason for this aside from how C handles arrays inside structs. The size of the
matrix is :math:`64 \times 64`, where every matrix item stands for a tile that
might have a door.


Spawning a door
~~~~~~~~~~~~~~~

Spawning a door is straight-forward: we take in the tile coordinates and the
number of the door, we use that to set the door member and then we assign the
door to the level's track-keeping.

-------------------------------------------------------------------------------

:Prerequisites: - `x` = vertical tile position
                - `y` = horizontal tile position
                - `n` = number of the door
                - The door tracking of the level has to be set up already

:Code:
 1) Register the new door in the door matrix of the level
 2) Set the door members according to the type of the door (type, vertical and
    texture)
 3) Set the position of the door to `x` and `y`
 4) Set the state of the door to closed
 5) Add the door to the door list
 6) Increment the door count for the level

-------------------------------------------------------------------------------


Setting door areas
~~~~~~~~~~~~~~~~~~

After the doors have been spawned their areas need to be assigned, only then can
the door let sound pass through.

-------------------------------------------------------------------------------

:Prerequisites: - `doors` = list of doors in the level
                - `areas` = table of areas in the level

:Code:
 1) For every door in `doors` do

    1) Make variables `x` and `y` the position of the door
    2) If the door is vertical

       1) Set Area 1 of the door to `areas[x+1][y]`
       2) Set Area 2 of the door to `areas[x-1][y]`
    3) Else

       1) Set Area 1 of the door to `areas[x][y+1]`
       2) Set Area 2 of the door to `areas[x][y-1]`
    4) If any of the areas just set are `< 0`, then set it to `0`

-------------------------------------------------------------------------------

This functions simply uses the areas table and the position of the door to pick
the area indices east and west (or north and south) of the door.


Managing doors
--------------

Now that we have set the doors up we can get to how to use them during play
time. For to following routines the variable `door` will always be a
prerequisite and refer to the door we want to operate on.


Changing the door state
~~~~~~~~~~~~~~~~~~~~~~~

A door can be opened at any time unless it is already open, but a door can only
close if it isn't blocked

-------------------------------------------------------------------------------

:Constants: `FULLOPEN = 63`

:Code:
 1) If the door state is closed or closing
 
    1) Open the door (see below)
 2) Else if the door is open and can be closed (see below)
 
    1) Change the door state to closing
    2) Set the `ticcount` of the door to `FULLOPEN`

-------------------------------------------------------------------------------

As we can see a door can be opened at any time, even interrupting the closing
process, but the opening process cannot be interrupted, the door must fully
open. Manually closing the door is supported in the DOS version but was
commented out in the iOS version. This was done due to the automatic using on
touchscreen devices.


Opening doors
~~~~~~~~~~~~~

If the door is already open we reset its timer, otherwise we start opening it.

-------------------------------------------------------------------------------

:Code:
 1) If the door's state is open
 
    1) Set the door's `ticcount` to `0`
 2) Else
 
    1) Set the door's state to opening

-------------------------------------------------------------------------------

If the door was already in the process of being opened this will have no effect.


Can a door be closed?
~~~~~~~~~~~~~~~~~~~~~

A door can only be closed if it wouldn't squish anyone in the process.

-------------------------------------------------------------------------------

:Constants: `CLOSEWALL = 0x5800` (Space between wall & player)

:Code:
 1) If the player's tile position is the position of the door
 
    1) Return false
 2) If the door is vertical
 
    1) If the player's vertical tile is the same as the door's
 
       1) If the horizontal tile of the player's horizontal position plus/minus
          `CLOSEWALL` is the same as the door's
 
          1) Return false
    2) For every actor in the level
 
       1) If the actor's tile position is the position of the door
 
          1) Return false
       2) If the actor's vertical tile is the same as the door's and the actor's
          horizontal tile minus/plus 1 is the same as the door's and the
          horizontal tile of the actors's horizontal position plus/minus
          `CLOSEWALL` is the same as the door's
 
          1) Return false
 3) Else
 
    1) Same as for vertical doors, except horizontal and vertical are swapped
 4) Return true

-------------------------------------------------------------------------------

The easy thing to test is whether and actor or the player is standing on the
door tile. The other, more complicated check is whether an actor or the player
is too close to the door to close. To elaborate, every actor as well as the
player have a sort of "radius" (it's really a bounding box) that prevents them
from getting too close to a wall, so we need to check if the border of the
entity is intersecting with the door tile.

To this end we add (or subtract) the bounding radius from the entity's position
on the coordinate axis in question. Then we convert this shifted position to a
tile coordinate and compare it with the door's tile coordinate. Remember that
the integer value of `CLOSEWALL` is actually a fixed-point decimal number.

The check for actor's is more complicated than for the player, this is to
prevent doing the more expensive check on every actor in the level. Instead we
first check if the actor is even close enough for consideration and the
compiler should take care that the more expensive check is optimised away if
the fist one fails. Other than that the checks are the same for both the player
and actors.


Is a door open?
~~~~~~~~~~~~~~~

We return a number that tells us not only whether a door is open, but also *how
far* open it is. A return value of 0 means the door is closed, a value of
`FULLOPEN` means the door is fully open, any value in between is partially
open.

-------------------------------------------------------------------------------

:Constants: `FULLOPEN = 63`

:Code:
 1) If the door is open
    1) Return `FULLOPEN`
 2) Else
    1) Return `ticcount` of the door

-------------------------------------------------------------------------------


Trying to use a door
~~~~~~~~~~~~~~~~~~~~

Regular doors and elevator doors can always be opened, but locked doors require
a key

-------------------------------------------------------------------------------

:Prerequisites: Information on what keys the player has collected so far

:Code:
 1) If the door is a regular- or elevator door
 
    1) Change the door state and return true
 2) If the door is a gold key door
 
    1) If the player has the gold key
 
       1) Change the door state and return true
    2) Else
 
       1) Inform the player (optional) and returns false
 3) If the door is a silver key door
 
    1) If the player has the silver key
 
       1) Change the door state and return true
    2) Else
 
       1) Inform the player (optional) and returns false

-------------------------------------------------------------------------------


Processing a door
~~~~~~~~~~~~~~~~~

Doors are processed during every frame. We look at the state of each door and
decide what to do. Doors are driven by time: unless the door is closed each
time the `ticcount` is incremented until it has reached a certain point, and
then the door does things on its own without outside input.

-------------------------------------------------------------------------------

:Prerequisites: `ticks` = ticks since last frame

:Constants: 
   - `OPENINGTIME =  63` (time it takes a door to open)
   - `OPENTIME    = 300` (time a door will remain open)

:Code:
 Looping over every door in the level, in every iteration switch based on the
 state of the door

 1) Closed

    1) Skip to the next iteration of the loop
 2) Opening

    1) If the `ticcount` of the door `>= OPENINGTIME`

       1) Set the state of the door to open
       2) Set the `ticcount` of the door to 0
    2) Else

       1) If the `ticcount` of the door `== 0`

          1) Connect the areas of the doors and update the connections
          2) If the player's area is connected to the first area of the door

             1) Play the door opening sound
       2) Add `ticks` to the `ticcount` of the door
       3) Cap the `ticcount` at `OPENINGTIME`
    3) Skip to the next iteration of the loop
 3) Closing

    1) If the `ticcount` of the door <= 0

       1) Disconnect the areas of the doors and update the connections
       2) Set the state of the door to closed
       3) Set the `ticcount` of the door to 0
    2) Else

       1) If the `ticcount` of the door `== OPENINGTIME` and the door's first
          area is connected to the player's area

          1) Play the door closing sound
       2) Subtract `ticks` from the `ticcount` of the door
       3) Cap the `ticcount` from below at `0`
    3) Skip to the next iteration of the loop
 4) Open

    1) If the door's `ticcount` `>= OPENTIME`

       1) If the door can be closed set the door's state to closing and ticcount
          to `OPENINGTIME`
    2) Else

       1) Add `ticks` to the door's `ticcount`, cap at `OPENTIME`

-------------------------------------------------------------------------------

For the most part this is straight-forward. Closed doors don't do anything,
opening doors are either still in the process of opening or they have just
finished doing so. Closing doors are the same in reverse. Open doors don't do
anything until the time comes to close, at which point they first check to see
if it's OK.

Opening and closing doors must also take care to connect and disconnect areas.
An opening door establishes connections the moment it starts opening and a
closing door disbands connections once it has finished closing. A door takes
the same time to open as it takes to close, that's why closing doors count in
reverse. It also means that when an entity interrupts one process (opening or
closing) we only need to invert the direction of the counter.

If a door cannot be closed after its time has passed it will stay open until it
can be closed, at which point it will close without delay.

All increments are capped to prevent the numbers from rolling over back to 0 or
into the negative range. That would screw up the timers.



Push-walls
==========

Push-walls look like regular walls, but the player can interact with them to
push them and reveal a secret. They are regular textured walls on the
architecture map, the push-wall information is on the objects map as the word
`0x0062`.

Pushwalls are rendered just like normal walls as long as they are not moving.
Once they start moving they are no longer regular walls, we can imagine it as
the wall disappearing and being replaced with a new object at the same position
and with the same texture. This object is then moved over time and the
raycaster adds the translation of the pushwall to the ray.


Anatomy of a push-wall
----------------------

A push-wall has the following members:

===============  ============  ==================================
Type             Name          Description                       
===============  ============  ==================================
Boolean          Active        Is the wall moving?               
Integer          Tiles Moved   How far have we moved (in tiles)? 
Integer          Points Moved  How far have we moved (in points)?
4-way direction  Direction     Direction to move in              
Integer          X             Tile of the push-wall             
Integer          Y             Tile of the push-wall             
Integer          Delta X       Offset in the direction           
Integer          Delta Y       Offset in the direction           
Integer          Texture X     Texture of the wall               
Integer          Texture Y     Texture of the wall               
===============  ============  ==================================

The game only keeps track of one push-wall: the wall that's currently being in
the process of moving, we'll call this object the *push-wall tracker*. This
means only one push-wall can be active at a time. It has its own textures
because the original wall has been "destroyed" and we need them to apply them
to the new wall when it stops moving.


Resetting push-walls
--------------------

Resetting means setting to members of the push-wall being kept track of to zero
(or false).


Pushing push-walls
------------------

This is what happens when the player tries pushing a push-wall. We check to see
if the tile behind the push-wall is free, then we mark the tile as a push-wall
tile, block the tile behind and get ready to start moving the wall.

-------------------------------------------------------------------------------

:Prerequisites: 
   - `x`   = horizontal tile of the push-wall
   - `y`   = vertical tile of the push-wall
   - `dir` = direction the player is facing

:Code:
 1)  If there is already an active push-wall

     1) Return
 2)  Turn the direction of the player to tile-deltas
 3)  If the tile behind the push-wall is a solid- or door tile

     1) Return
 4)  Remove the Secret- and Wall flags from the tile of the push-wall
 5)  Add the push-wall flag
 6)  Increment the secrets counter of the level and display a message to the
     player
 7)  Play the push-wall sound
 8)  Add the push-wall flag to the tile behind (prevents stepping on it and
     making things stuck)
 9)  Set the push-wall tracker to active
 10) Set the tracker's tiles and points moved to 0
 11) Set the tracker's position, deltas and direction to what we have
 12) Set the tracker's textures to the textures of the wall

-------------------------------------------------------------------------------

A tile-delta is the difference (delta) of two tiles for each axis, meaning
there is a `delta_x` and `delta_y`. The position "behind" means behind the
push-wall from the player's perspective in the direction of the delta.


Processing push-walls
---------------------

Push-walls are processed every frame.

-------------------------------------------------------------------------------

:Code:
 1) If there is no active push-wall

    1) Return
 2) Add the ticks since the last frame to the points moved
 3) If the points moved `< 128`

    1) Return
 4) Subtract the 180 from the points moved and add 1 to the tiles moved
 5) Remove the Push-wall flag from the current tile
 6) Add the deltas to the current tile and make that the current tile
 7) If the tile behind the current tile is a solid tile, a door tile, an actor
    tile or a player tile or the tiles moved `== 3`

    1) Remove the Push-wall flag from the current tile and add the Wall flag
    2) Assign the textures from the push-wall to the newly created wall tile
    3) Set the push-wall tracker to not active
 8) Else

    1) Add the Push-wall flag to the tile behind the current tile

-------------------------------------------------------------------------------

Every frame we move the wall a little bit. Once the wall has moved by one tile
we unlock the tile in front of the wall and block the tile behind the wall.
That is, only if the wall can actually move further, otherwise we turn the
push-wall into a new regular wall.



Line of sight
=============
To be done...



Actors / Entities
#################

(AI is an utter mess and on hold for now)

Actors, or entities as they can also be referred to in the code, are any
in-game entities that can move around in the world. They include enemies as
well as projectiles like fireballs or rockets and even BJ himself, but not
static objects like weapons, food, chairs or stone columns. An actor's
behaviour is modelled using a finite-state machine where each state holds
information on what sprite to display, how long the state lasts, what state to
transition to.


The actor structure
===================

An actor is define as a structure with the following members:

===========  ==============  ==================================
Type         Name            Description                       
===========  ==============  ==================================
Float        position_x      Horizontal position on the map    
Float        position_y      Vertical position on the map      
Integer      angle           Angle the actor is facing         
Integer      type            Class of the actor (e.g. guard)   
Integer      current_health  Current health of the actor       
Integer      maximum_health  Maximum health of the actor       
Integer      speed           Walking speed                     
Integer      tic_count       Timer driving the actions         
Integer      reaction        Reaction time for noticing player?
Integer      distance;       ???                               
Character    tile_x          Tile the actor is standing on     
Character    tile_y          Tile the actor is standing on     
Character    area_number     Area on the map                   
Integer      waitfordoor_x   Waiting on this door if non 0  
Integer      waitfordoor_y                                     
Actor_flags  flags           Various flags for game rules      
Actor_state  state           Currents state                    
Dir8type     direction       Direction to move into            
Integer      sprite          Sprite to display                 
===========  ==============  ==================================

The type `actor_flags` is a combination of various options which can be either
on or off.

===========  =======
Option       Meaning
===========  =======
Shootable    ?      
Bonus        ?      
Nevermark    ?      
Visable      ?      
Attackmode   ?      
Firstattack  ?      
Ambush       ?      
Nonmark      ?      
===========  =======


Starting hit points
-------------------

The starting hit points of an actor depend on the chosen game difficulty. The
list can be found in the enemy table file.


Actor states
============

Each actor state uses the same basic state structure:

=======  ===========  =======================================================
Type     Name         Description                                            
=======  ===========  =======================================================
Boolean  can_rotate   `true` if actor has unique sprites for every rotation
Int      base_sprite  Base sprite for when facing the player                 
Int      timeout      Duration of the state until transitioning to next state
Think    thought      Function to call every frame during this state         
Think    action       Function to call when changing state                   
State    next_state   Next state to transition to naturally                  
=======  ===========  =======================================================

The first member tells us whether the actor has different sprites for rotation
or if it is always facing the player; for example, guards have different
directions for walking, allowing the player to sneak behind them, but they
always face the player when they are shooting or when they are dying.

The second member tells us the index of the base sprite, the image to display
when the actor is facing the player. For non-rotatable states this is the
sprite to always display, but for rotatable states the right sprite has to be
found using the base sprite and adding an appropriate offset to get the index
of the proper sprite. The offset depends on the rotation of the actor relative
to the player.

The `Think` type is a function pointer to a function that takes one actor as
its argument, usually the actor calling it, and returns nothing

.. code::

	typedef void (*think_t)( entity_t *self )

We can see that these states allow the actors to naturally transition from one
state into another solely based on time passed. A patrolling enemy will cycle
between patrolling states on its own as long as it doesn't become aware of the
player, an enemy in pain will naturally transition to shooting and a dying
enemy will automatically be dead once the dying animation has finished playing.
The exact actor states are hard-coded and can be found within the
*wolf_act_stat.h* file of the original source. There can be several states with
similar function, like several walking states, they are driving the animation
frames.


Groups of states
----------------

States can be split into the following groups:

Standing still:
   The actor is just standing in one spot and waiting
Patrolling:
   The actor is moving along a pre-defined part and can open doors if needed.
   Dogs cannot stand still and must always walk.
In pain:
   Temporarily paralysed after getting shot at
Attacking:
   Shooting for humans and biting for dogs
Chasing:
   Actively pursuing the player and occasionally stopping to shoot
Dying:*
   In the process of dying
Dead:
   Having died
Removed:*
   ???

Each of these groups consists of several actual states, with the exception of
the standing- and dead state since there is only one way of standing still or
being dead. If a state is unused it is still defined, but its members are
useless junk data and the sprite is the "demo" sprite. Each state can only
display one sprite, so in order to cycle through animation frames the states
within one group must be cycled through. In the case of the brown guard there
are three shooting frames, so the guard cycles through the first three of his
shooting states with the remaining shooting states being unused. There also
appear to be special states for some actors, but those are just the above
states re-purposed.


Changing state
--------------

To change the state of an actor set its state to the target state. If the state
is the `remove` state set the `tic_count` to `0`, otherwise set it to the
`timeout` of the target state.

-------------------------------------------------------------------------------

:Prerequisites: - `actor`       = existing actor
                - `target`      = target state
                - `state_table` = maps actor and state to concrete state

:Side effects: will change the `state` and `tic_count` of `actor`

:Code:
 1) Set state of `actor` to `target`
 2) If `target == remove`
 
    1) Set tic_count of `actor` to 0
 3) Else
 
    1) Set tic_count of `actor` to timeout of `state_table(actor, target)`

-------------------------------------------------------------------------------


Actor routine
~~~~~~~~~~~~~

The following routine if called every frame on every actor when processing
actors (see below). The variable `ticks` measures the number of ticks that have
passed since the last frame; for a :math:`30` FPS game that would be two ticks.

-------------------------------------------------------------------------------

:Prerequisites:
   - `actor` = the actor to run the routine on
   - `tics`  = ticks passed since last time

:Side effects:
   - might change the state of `actor`
   - might call the `thought` and `action` of the state

:Return value:
   boolean, false if `actor` ends up in the `remove` state

:Code:
 1) If `tic_count` of `actor != 0`

    1) Subtract `ticks` from `tic_count` of `actor`
    2) While `tick_count` of `actor <= 0`

       1) Set `action` to the `action` of `actor`
       2) If `action` is not `NULL`

          1) Perform `action`
          2) If `state` of `actor` is `remove`

             1) Return false
       3) Transition to next state
       4) If the state is `remove`

          1) Return false
       5) If `timeout` of the state is 0

          1) Set `tic_count` of `actor` to 0
          2) Break out of the loop
       6) Add `timeout` of the state to the `tic_count` of `actor`
 2) Set `thought` to the `thought` of `actor`
 3) If `thought` is not `NULL`

    1) Perform `think`
    2) If the state of `actor` is `remove`

       1) Return false
 4) Return true

-------------------------------------------------------------------------------

The routine has two major parts. In the first part we subtract the time passed
from the actor's tick count. If the count drops to 0 or below we have to call
the actor's action and change the state. We have to do this for every state
that has passed since the last run of the routine.

This routine is not perfect, if the game speed drops too low the subtracted
ticks might skip too many calls of the actor's *think* function.


Removing an actor
-----------------

To remove an actor remove it from the global list of actors. This will make any
functions that iterates over actors skip it, but the actor will still remain as
a corpse sprite in the game.


Processing actors
-----------------

Pseudocode:

-------------------------------------------------------------------------------

1) For each living (i.e. not dead) actor do the following

   1) Run the actor routine on the current actor
   2) If the routine returned false

      1) Remove the actor and skip to the next actor
   3) Adjust the position and angle of the actor's sprite
   4) If the actor state can rotate

      1) Add the rotation to the index of the base sprite
   5) Display the sprite

-------------------------------------------------------------------------------

Rotating a sprite means taking the actor's angle and computing the closest
direction. Each direction can be mapped to an integer number and this number is
added to the index of the base sprite texture (the one facing the player). The
mapping is as follows

.. code::

	r_add8dir[ 9 ] = { 4, 7, 6, 5, 0, 1, 2, 3, 0 };  // for rockets and hrockets
	a_add8dir[ 9 ] = { 4, 5, 6, 7, 0, 1, 2, 3, 0 };  // for every other actor

The index of the direction to use is the direction of the angle difference
between the player and the actor. This means we first compute the absolute
difference in angles between actor and player and use that angle to get an
eight-way direction. This direction is the index of the number to add.


Creating a new actor
--------------------

Creating a new actor is the inverse of removing it. Instantiate a new empty
actor and add it to the list of actors. Its members will be initialised by the
function calling this.


Spawning actors
---------------

Spawning actors is split into a number of similar but not exactly same
functions.  There are standing actors, patrolling actors, dead actors, bosses
and ghosts.  All the spawning functions call one general spawning function.

In my opinion these are too many special cases that should be resolved using a
sort of table and only one spawning function.


Spawn general actor
~~~~~~~~~~~~~~~~~~~

This function is called by other functions to spawn an actor in the world.

-------------------------------------------------------------------------------

:Prerequisites: - `class` = actor class of the new actor
                - `x`     = tile X-coordinate of the actor
                - `y`     = tile Y-coordinate of the actor
                - `dir`   = 4-way direction for the actor to face
                - `level` = the level to spawn in

:Code:
 1) Create a new actor as `actor`
 2) Convert `x` and `y` to to world positions and set them as the actor
    position
 3) Set `angle` and `direction` of `actor` to `dir`
 4) Set `area_number` to area of tile the actor is standing on
 5) If `area_number < 0`
    1) Set `area_number` to 0
 6) Set `type` of the actor to `class`
 7) Set `health` of the actor from the health table (see appendix)
 8) Set `sprite` of the actor to a newly created sprite

-------------------------------------------------------------------------------


Spawning standing actor
~~~~~~~~~~~~~~~~~~~~~~~

This function spawns a regular still-standing actor. The actor can be either on
guard or in ambush mode (deaf).

-------------------------------------------------------------------------------

:Prerequisites:
   - `class` = actor class of the new actor
   - `x`     = tile X-coordinate of the actor
   - `y`     = tile Y-coordinate of the actor
   - `dir`   = 4-way direction for the actor to face
   - `level` = the level to spawn in

:Code:
 1) Spawn a new actor as `actor`
 2) Set `state` of the actor to `stand` and `speed` to `SPDPATROL`
 3) If `timeout` of the state for this actor class and state class
    `stand != 0`

    1) Set `tic_count` of the actor to `timeout + 1`
 4) Else

    1) Set `tic_count` of the actor to 0
 5) Add the Shootable flag to the actor
 6) If the actor is standing on an ambush tile

    1) Add the Ambush flag to the actor
 7) Increment enemy count of the level

-------------------------------------------------------------------------------


Spawning patrolling actor
~~~~~~~~~~~~~~~~~~~~~~~~~

This function spawns a patrolling actor, dogs always patrol.

-------------------------------------------------------------------------------

:Prerequisites:
   - `class` = actor class of the new actor
   - `x`     = tile X-coordinate of the actor
   - `y`     = tile Y-coordinate of the actor
   - `dir`   = 4-way direction for the actor to face
   - `level` = the level to spawn in

:Code:
 1) Spawn a new actor as `actor`
 2) Set `state` of the actor to `path1` and `speed` to `SPDPATROL`
 3) Set `speed` of the actor to `SPDPATROL`, or `SPDDOG` if the actor is a
    dog
 4) Set `distance` of the actor to `TILEGLOBAL`
 5) If the `timeout` of the state from the state table != 0

    1) Set `tic_count` of the actor to the `timeout + 1`
 6) Else

    1) Set `tic_count` of the actor to 0
 7) Add the Shootable flag to the actor
 8) Increment enemy count of the level

-------------------------------------------------------------------------------


Spawning dead actor
~~~~~~~~~~~~~~~~~~~

Dead actors are special in that they have no direction to look at.

-------------------------------------------------------------------------------

:Prerequisites: 
   - `class` = actor class of the new actor
   - `x`     = tile X-coordinate of the actor
   - `y`     = tile Y-coordinate of the actor

:Code:
 1) Spawn a new actor as `actor` with no direction
 2) Set `state` of the actor to `dead`
 3) Set health and `speed` of the actor to 0
 4) If the `timeout` of the state from the state table != 0

    1) Set `tic_count` of the actor to the `timeout + 1`
 5) Else

    1) Set `tic_count` of the actor to 0

-------------------------------------------------------------------------------


Spawning boss actor
~~~~~~~~~~~~~~~~~~~

The direction of bosses depend on the particular boss.

-------------------------------------------------------------------------------

:Prerequisites: 
   - `class` = actor class of the new actor
   - `x`     = tile X-coordinate of the actor
   - `y`     = tile Y-coordinate of the actor

:Code:
 1)  Make 4-way direction variable `dir`
 2)  Value of `dir` is

     1) South for: Hans, Schabbs, Fettgesicht and Hitler
     2) North for: Fake Hitler, Gretel and Giftmacher
     3) No direction for everything else
 3)  Spawn a new actor as `actor` with direction `dir`
 4)  Set the state of the actor to `path_1` for a spectre and `stand` for
     everyone else
 5)  Set `speed` of the actor to `SPDPATROL`
 6)  Set `health` of the actor from the starting health table (redundant?)
 7)  If the `timeout` of the state from the state table != 0

     1) Set `tic_count` of the actor to the `timeout + 1`
 8)  Else

     1) Set `tic_count` of the actor to 0
 9) Add the Shootable and Ambush flag to the actor
 10) Increment enemy count of the level

-------------------------------------------------------------------------------


Spawning ghost actor
~~~~~~~~~~~~~~~~~~~~

This function spawns Pac-Man ghosts.

-------------------------------------------------------------------------------

:Prerequisites: 
   - `class` = actor class of the new actor
   - `x`     = tile X-coordinate of the actor
   - `y`     = tile Y-coordinate of the actor

:Code:
 1) Spawn a new actor as `actor` with no direction
 2) Set `state` of the actor to `chase1`
 3) Set `speed` of the actor to `SPDPATROL*3`
 4) Set `health` of the actor from the starting health table (redundant?)
 5) If the `timeout` of the state from the state table != 0

    1) Set `tic_count` of the actor to the `timeout + 1`
 6) Else

    1) Set `tic_count` of the actor to 0
 7) Add the Ambush flag to the actor
 8) Increment enemy count of the level

-------------------------------------------------------------------------------



Actor AI
========

All the functions in this sub-section have an actor as a prerequisite. To save
redundancy I will not list it as a prerequisite and I'll refer to it in the
pseudo-code as *the actor* or `actor`.


General AI routines
-------------------

These AI routines are not called directly, but they are called by other
functions, both AI routines and thoughts.


Check Sight
~~~~~~~~~~~

This routine scans the line of sight of the actor for the presence of the
player.

-------------------------------------------------------------------------------

:Constants: 
   `MINSIGHT = 1.1` (below this distance the player is always noticed)

:Code:
 1) If the actor does not have the Ambush flag set and the player is not in the
    same area
    1) Return false
 2) If the difference of the actor's and player's position on both coordinates
    is less than `MINSIGHT`

    1) Return true
 3) If the player is not in front of the actor return false. We only compare the
    direction of the player, not if the actor can actually see the player
 4) Return the result of the Check Line function using the actor and the player.
    The function is discussed in the level section

-------------------------------------------------------------------------------

First we exclude the cases that are easy to verify. Then we exclude the cases
where the player is behind the actor, e.g. if the actor is facing south and the
player is north of the actor. Finally, we check if the line between the actor
and the player is unobstructed; the player could be hiding around a corner or
behind a door like in the title screen.


First Sighting
~~~~~~~~~~~~~~

This routine puts the actor into an attack state and makes it face the player.

-------------------------------------------------------------------------------

1) Play a sound and multiply the actor's `speed` by a factor
   Both depend on the actor's class, there is a table below
2) If the actor's `waitfordoor_x != 0`

   1) Set the actor's `waitfordoor_x` and `waitfordoor_y` to 0
3) Change the actor's state to `Chase1`
4) Set actor's direction to no direction
5) Set the actor's Attackmode and Firstattack flags on

-------------------------------------------------------------------------------

If the actor was waiting for a door to open while is spotted the player the
waiting is cancelled since the actor is now primarily concerned with killing
the player, not opening a door. Here is the table with the sound effects and
speed factors for the individual actor classes.

===========  =============  ===============
Actor class  Sound Number   Factor         
===========  =============  ===============
Guard                   1               x 3
Officer                71               x 5
SS                     15               x 4
Dog                     2               x 2
Hans                   71   = SPDPATROL x 3
Schabbs                65               x 3
Fake                   54               x 3
Mecha                  40               x 3
Hitler                 40               x 5
Mutant                                  x 3
Blinky                                  x 2
Clyde                                   x 2
Pinky                                   x 2
Inky                                    x 2
Gretel                112               x 3
Gift                   96               x 3
Fat                   102               x 3
                                           
Officer                43                  
Spectre                 3            =  800
Angel                  95            = 1536
Trans                  66            = 1536
Uber                                 = 3000
Will                   73            = 2048
Death                  85            = 2048
===========  =============  ===============

The officer has a different sound for Spear of Destiny, but the same speed.
Speeds prepended with "=" are set to a fixed value. The ghosts and the
(uber)mutant have no sound to play.


Find Target
~~~~~~~~~~~

This routine is scanning the surroundings of the actor for the player. After
the player has been spotted the actor will act surprised for a while and
actions will be delayed; this is achieved by the actor's `reaction` member.

-------------------------------------------------------------------------------

:Returns: true if the player was detected, false otherwise

:Side effects: 
   - Changes the `react` of the actor
   - Might change the actor's Ambush flag

:Code:
 1) If `reaction` of the actor > 0

    1) Subtract ticks since last frame from `reaction`
    2) If `reaction > 0`

       1) Return false
    3) Set `reaction` to 0
 2) Else

    1) Set `reaction` to 0
    2) If the player has the Notarget flag set ("notarget" cheat)

       1) Return false
    3) If the actor does not have the ambush flag and the player is not in the
       same area

       1) Return false
    4) If Check Sight returned false //failed to see, attempt to hear

       1) If the actor has the Ambush flag set or if the player hasn't made
          any noise

          1) Return false
    5) Remove the Ambush flag from the actor
    6) Set the actor's `reaction` depending on the actor's class

       1) For guards to (1 + Random/4)
       2) For officers to 2
       3) For SS and mutants to (1 + Random/6)
       4) For dogs to (1 + Random/8)
       5) For everyone else to 1
    7) Return false
 3) Run the First Sighting routine on the actor
 4) Return true

-------------------------------------------------------------------------------

This function works in two ways: If the player hasn't been spotted it will keep
looking. Once the player has been spotted a reaction delay will be initialised.
As long as that delay persists the function will just keep decrementing it.
Only after the reaction delay has passed will the function be called where the
actor does actually react.

Note that once an actor has spotted the player it will eventually react, there
is no way to quickly run into hiding or that the actor will forget about the
player.


Change Direction
~~~~~~~~~~~~~~~~

This routine changes the direction an actor is facing, if that direction is a
valid one.

-------------------------------------------------------------------------------

:Prerequisites: 
   - `new_direction` = direction for the actor to face
   - `level_data`    = data of the current level

:Return: true if not facing a solid obstacle after changing direction

:Code:
 1)  Make a new position variable `old` from actor's position
 2)  Make a new position variable `new` from actor's position plus the new
     direction
 3)  If `new_direction` is diagonal

     1) If the vertically adjacent tile towards the new direction is solid
        or the horizontally adjacent tile towards the new direction is solid
        or the diagonally adjacent tile towards the new direction is solid

        1) Return false
     2) If a non-dead actor is on one of the above tiles
        1) return false
 4)  Else

     1) If the tile towards the new direction is solid
        1) Return false
     2) If the new tile is a door

        1) If the actor is either a dog or Fake Hitler

           1) If the door is not open

              1) Return false
        2) Else

           1) Set the `waitfordoor_x` and `_y` of the actor to the new tile
           2) Go to 5)
     3) If a non-dead guard is standing on the new tile
        1) Return false
 5)  Set the actor's tile coordinates to the new tile
 6)  Remove the Actor flag from the old tile and add it to the new tile
 7)  If the area number of the new tile > 0
     1) Set the actor's `area_number` to the tile's area number
 8)  Set the actor's `distance` to `TILEGLOBAL`
 9)  Set the actor's `direction` to `new_direction`
 10) Return true

-------------------------------------------------------------------------------

Checking if another actor is occupying a tile is done by comparing the actor's
tile coordinates `tile_x` and `tile_y` with the coordinates of the tile in
question.


Move
~~~~

-------------------------------------------------------------------------------

:Prerequisites: `distance` = the distance to move by

:Constants: `MINACTORDIST = 1.0`

:Code:
 1) If the actor's direction is No Direction or `distance == 0`

    1) Return
 2) Add `distance` times the actor's distance to the actor's position
 3) If difference in coordinates of either axes between actor and player <
    `MINACTORDIST`

    1) If the actor is a Pac-Man ghost or spectre

       1) Run the Damage routine on the player from the actor for 2 damage
    2) Back up the actor (subtract what we added at step 2))
 4) Subtract `distance` from the actor's `distance`
 5) If the actor's `distance < 0`

    1) Set the actor's `distance` to 0

-------------------------------------------------------------------------------


Advance
~~~~~~~

Advances the actor.

-------------------------------------------------------------------------------

:Prerequisites: `thought` = The thought to execute before advancing

:Code:
 1) If `thought` is `NULL`

    1) Return
 2) Make new variable `move` as `speed` of the actor times ticks since last
    frame
 3) While `move > 0`

    1) If the actor is waiting for a door to open

       1) Open the door
       2) If the door is not open
          1) Return
       3) Set the actor's `waifordoor_x` and `waifordoor_y` to 0
    2) If `move < distance` of the actor
       1) Run the Move routine on the actor using `move`
       2) Break out of the loop
    3) Set the actor's position based on the tile
    4) Subtract the actor's distance from `move`
    5) Run the thought on the actor
    6) Set the actor's angle based on its direction
    7) If the actor's direction is No Direction

       1) Return

-------------------------------------------------------------------------------

Setting the actor's position based on the tile means casting the tile
coordinates to a floating point number and adding :math:`0.1`.


Dodge
~~~~~

This routine advances the actor towards the player while trying to sidestep to
dodge attacks. It does not actually move the player, it just selects which
direction to face.

-------------------------------------------------------------------------------

1)  Make new 8-way direction variable `turnaround`
2)  If the actor has the Firstattack flag set

    1) Set `turnaround` to No Direction
    2) Remove the Firstattack flag from the actor
3)  Else

    1) Set `turnaround` to opposite of actor's `direction`
4)  Get the difference in tiles between the actor's and the player's position
    for both axes
5)  Make a new array of five 8-way directions `trydir`
6)  If the player-actor X-difference is > 0

    1) Set `trydir[1] = East` and `trydir[3] = West`
7)  Else

    1) Same as 6.1) but swapped
8)  Same as 6) but for Y

    1) Set `trydir[2] = North` and `trydir[4] = South`
9)  Else

    1) Same as 8.1) but swapped
10) If the absolute value of the X-delta > absolute value of theY-delta

    1) Swap `trydir` 1 and 3 and swap `trydir` 2 and 4
11) If a random number < 128

    1) Swap `trydir` 1 and 3 and swap `trydir` 2 and 4
12) Set `trydir[0]` to the diagonal of 1 and 2
13) For every direction in `trydir`

    1) If the direction is No Direction or the same as `turnaround`

       1) Skip to the next iteration of the loop
       2) If running the Change Direction on the actor using the current
          `trydir` returns true

          1) Return
14) If `turnaround` is not No Direction

    1) If running the Change Direction on the actor using `turnaround` returns
       true

       1) Return
15) Set the actor's `direction` to No Direction

-------------------------------------------------------------------------------

If the actor has only now spotted the player it will not be able to turn
around, otherwise it will. The steps 4) to 9) set up the movement directions,
but those directions are for when the player is east or west of the actor, so
we need to swap horizontal and vertical when the difference horizontally is
greater than the difference vertically. Then we randomly swap (again) to
simulate a spontaneous side step. Finally we create a diagonal direction as our
first choice.

After the directions have been set up we iterate through them and try them out.
If a direction is undefined or the opposite direction it is illegal and we move
on to the next one. Otherwise we try to change to the direction and are done if
everything worked out.

If all directions fail attempt to turn around. Turning around was forbidden
above, because other directions need to take precedence, turning around is only
the last resort. If even that is not possible the direction of the actor is set
to undefined.

There appears to be a bug: In the original code a comment says turning around
is only OK the first time noticing the player, but the implementation is the
opposite: turning around is impossible on first notice. I have documented the
code as it was written, since that is the behaviour the game shipped with.


Chase
~~~~~

This routine is similar to Dodge, but without the side stepping. Pseudocode:

-------------------------------------------------------------------------------

1)  Make integer variables `delta_x` and `delta_y` and assign them the
    difference in tiles between the actor and the player
2)  Make new variable `turnaround` and make it the opposite of the actor's
    direction
3)  Make new array `dir` of two 8-way directions
4)  If `delta_x` > 0

    1) Set `dir[0] = East`
5)  Else

    1) Set `dir[0] = West`
6)  If `delta_y > 0`

    1) Set `dir[1] = North`
7)  Else

    1) Set `dir[1] = South`
8)  If absolute value of `delta_x` > absolute value of `delta_y`

    1) Swap `dir[0]` and `dir[1]`
9)  If either of the two == `turnaround`

    1) Set that one to No Direction
10) If `dir[0]` != No Direction

    1) If the result of running Change Direction on the actor using `dir[0]`
       is true

       2) Return
11) Same as above, except using `dir[0]`
12) If the original direction != No Direction

    1) If the result of running Change Direction on the actor using the original
       direction is true

       2) Return
13) If a random number > 128

    1) Loop counter-clockwise through the non-diagonal directions starting East

       1) If the direction != `turnaround`

          1) If the result of running Change Direction on the actor using the
             direction == true

             1) Return
14) Else

    1) Loop clockwise through the non-diagonal directions starting South

       1) If the direction != `turnaround`

          1) If the result of running Change Direction on the actor using the
             direction == true

             1) Return
15) If `turnaround` != No Direction

    1) If the result of running Change Direction on the actor using the
       `turnaround == true`

       1) Return
16) Set `direction` of the actor to No Direction

-------------------------------------------------------------------------------

See the discussion of the Dodge routine above. There is no side-stepping here,
so we need only two directions, but the basic idea is the same. If we can't
find a proper direction we cycle through them, the cycle being randomly picked.
If we still can't decide we turn the actor around. If even that fails the
direction is undefined and the actor can't move.


Orientate
~~~~~~~~~

This routine will change the direction of the actor if it is standing on a
waypoint.

-------------------------------------------------------------------------------

1) If the position of the actor is on a map tile that's a waypoint

   1) If the tile is an East tile

      1) Set the actor's direction to East
   2) If the tile is a North-East tile
   3) ...
   4) ...
   5) ...
   6) ...
   7) ...

      1) Set the actor's direction to North-East
   8) If the tile is a South-East tile

      1) Set the actor's direction to South-East
2) If running the Change Direction routine on the actor and the actor's
   direction returns false

   1) Set the actor's direction to no direction

-------------------------------------------------------------------------------


Thinking AI routines
--------------------

These AI routines are called directly as part of an actor's state machine. To
differentiate them from the routines above they will be called *thoughts*,
since they are used by the actor state machines.


Stand
~~~~~

Run the Find Target routine on this actor. That's all there is to it for actors
who are standing still.


Path
~~~~

This is the thought for actors patrolling on a path.

-------------------------------------------------------------------------------

1) If the result of running Find Target is true

   1) Return
2) If the speed of the actor is 0

   1) Return
3) If the direction of the actor is No Direction

   1) Run the Orientate routine on the actor
   2) If the direction is still No Direction

      1) Return
4) Run the Advance routing on the actor using the AI Path routine

-------------------------------------------------------------------------------


Ghosts
~~~~~~

Thought for ghost-type actors.

-------------------------------------------------------------------------------

1) If the `direction` of the actor is No Direction

   1) Run the Chase AI routine on the actor
   2) If the `direction` of the actor is still No Direction

      1) Return
2) Run the Advance AI routine on the actor using the Chase AI routine

-------------------------------------------------------------------------------

