.. The following is an accurate replication of the *Age of Wonders* Mannual
   Addendum, as published by the Stratos Group, in reStructuredText format.

   The source of original document is not available, only pre-built PDFs are in
   circulation and the Stratos Group no longer exits. This is an effort to
   preserve the addendum in an open format for possible further conversion. 

   The original format and content have been preserved as much as possible,
   including all hyperlinks, regardless of whether their targets actually exist
   anymore. Similarly, the text has been copy-pasted, so any errors that might
   exist are from the original source and will not be fixed.

   Changes from original:
   ----------------------

   - Centered text on first page is left-aligned because text alignment cannot
     be specified in reST.
   - Same with the note below *Manual Correction and Updates*.
   - Not applicable table items are written as ``\-\``, this is needed to escape
     the dashes to prevent them form being rendred as bullets. The string still
     renders as a ``-``, like in the original.

==============================
Age of Wonders Manual Addendum
==============================

About this Addendum:
====================
12-20-99 Version 2.3

This represents the second release of the Age of Wonders manual addendum.
Version 2.3 reflects the changes made with the Age of Wonders 1.2 patch and
corrections to errors in the previous version. If you have any suggestions for
future improvements, please keep the constructive criticism coming. Please visit
the `Age of Wonders Website <http://www.ageofwonders.com>`_ and post your
thoughts to the forum.

Written for the Players

**Special Thanks to:**

Lennart, Ray, Josh, Arno and everyone at Triumph who works on the game and whom
make a Wonderous effort to listen to the players ideas and make them a reality.

Thanks to peZLand for formatting the Addendum as cool as possible.

Feedback or questions about the addendum can be e-mailed to Nordramor at
nordramor@stratosgroup.com

©1999 Gathering of Developers I, Ltd. All rights reserved. The software and
related manual for this product are copyrighted. `Gathering of Developers
Website <http://www.godgames.com>`_

Age of Wonders, the Age of Wonders logo, Triumph Software, and the Triumph logo
are trademarks of Triumph Software, Inc. Copyright © 1999 Triumph Software, Inc.
All Rights Reserved.

Compiled and authored by the Stratos Group™, ©1999, `Stratos Group Website
<http://www.stratosgroup.com>`_

Race Relations and Morale
=========================

Default Initial Race Relations
------------------------------
(These can vary due to different scenario settings)

=========  =====  =====  ======  =====  ======  ========  =====  =====  =======  =====  ======  ======
..         Human  Azrac  Lizard  Frost    Elf   Halfling  Dwarf  High   D. Elf    Orc   Goblin  Undead
=========  =====  =====  ======  =====  ======  ========  =====  =====  =======  =====  ======  ======
Human       *F*     P       P      P       N        N       N      P       N       N       N       W  
Azrac        P     *F*      N      N       N        N       N      W       N       N       P       W  
Lizard       P      N      *F*     P       N        N       N      W       N       N       N       W  
Frostling    P      N       P     *F*      N        N       N      W       N       N       N       W  
Elf          N      N       N      N      *F*      *F*      P     *F*    **H**   **H**   **H**   **H**
Halfling     N      N       N      N      *F*      *F*     *F*     P     **H**   **H**   **H**   **H**
Dwarf        N      N       N      N       P       *F*     *F*    *F*    **H**   **H**   **H**   **H**
Highmen      P      W       W      W      *F*       P      *F*    *F*    **H**   **H**   **H**   **H**
Dark Elf    *F*     N       N      N     **H**    **H**   **H**  **H**    *F*     *F*      P       P  
Orc          N      N       N      N     **H**    **H**   **H**  **H**    *F*     *F*     *F*      P  
Goblin       N      P       N      N     **H**    **H**   **H**  **H**     P      *F*     *F*      P  
Undead       W      W       W      W     **H**    **H**   **H**  **H**     P       P       P      *F* 
=========  =====  =====  ======  =====  ======  ========  =====  =====  =======  =====  ======  ======

:F: Friendly
:P: Polite
:N: Neutral
:W: Wary
:H: Hate

Race Relation Modifiers
-----------------------
Each race relation type has a point range assigned to it:

=============  =========================
Race Relation  Race Relation Point Range
=============  =========================
Friendly                        80 - 100
Polite                          60 -  79
Neutral                         40 -  59
Wary                            20 -  39
Hate                             0 -  19
=============  =========================

The following actions will either improve or worsen your race relationships:

==============================================  ============================
Action Towards Race                             Race Realtion Point Modifier
==============================================  ============================
Raze city of race                                            -30
Loot city of race                                            -30
Migrate from race                                            -15
Migrate to race                                              +10
Upgrade city of race                                         +5
Upgrade walls of race's city                                 +5
Cancel looting of race's city                                +20
New game turn (until default relation reached)             +1 or -1
==============================================  ============================

==============================  ============================
Diplomatic Action Towards Race  Race Realtion Point Modifier
==============================  ============================
Making Alliance                            +20
Breaking Alliance                          +20
Making Peace                               +20
Breaking Peace/Declaring War               +20
==============================  ============================

Diplomatic actions can affects your race relationship with races not directly
involved in the action. Example: Declaring war on the Elves will likely hurt
your race relationship with the Halflings, but help your race relationship with
the Orcs. The indirect impact of diplomatic actions varies from race to race and
from action to action. The stronger the action, the greater the indirect race
relation impact.

Unit Morale
-----------
Unit morale is a reflection of a unit’s willingness to fight for your empire. A
unit’s morale is equal to your race relation with the unit’s race plus or minus
and modifiers. Machines, like Battering Rams, have no morale value and never
receive stat penalties or figure into party status decisions.

===========  =======================  =========================
Unit Morale  Unit Morale Point Range  Unit Stat Modifiers
===========  =======================  =========================
High                 80 - 100                   None
Good                 60 -  79                   None
Okay                 40 -  59                   None
Poor                 20 -  39         -1 Defense, -1 Resistance
Terrible              0 -  19         -2 Defense, -2 Resistance
===========  =======================  =========================

If applicable, the following factors are added or subtracted from your race
relation points when determining a unit’s morale.

================================  ==========================
Name                              Unit Morale Point Modifier
================================  ==========================
Friendly Terrain                              +10
Hostile Terrain                               -10
Panicked                                      -40
Fear Trauma                                   -40
Insufficient upkeep                    0 to -50 (5 turns)
Unit with Bard’s skills in party              +10
Hostile unit in party                    -10 (each unit)
================================  ==========================

Party Status
------------
Party status is a reflection of the overall morale of all units in the party. If
the majority of the party contains units with okay, good, or high morale, the
party status will be stable, content, or cheerful. A stable or better party
status means no units in that party will desert or rebel. If, however, units
with poor or terrible morale are in the majority, the party status will fall
into unrest or unruly and their will be a chance each turn the units from that
party will defect or rebel against your leadership. Higher-level units exert
more influence in party status decisions than do lower- level units. Examples:
An Orc Red Dragon with terrible morale will require a multiple Dwarven Giants
with high morale to effectively suppress, while an Elf Archer with good morale
can typically suppress a Goblin Spearman with poor morale.

=================  ============================
Party Status Name  Chance of Desertion per Turn
=================  ============================
Cheerful                       0%
Content                        0%
Stable                         0%
Unrest                        10%
Unruly                        50%
=================  ============================

City Status
-----------
Each city has a unique relationship with your empire. City status falls into two
separate scales, hostile cities and friendly cities. To determine a city’s
status, start with your race relation value point value, then apply any
applicable modifiers listed below. There are two separate scales listed, the
first is for cities with whom your race relation (not city status) with the
population is neutral, polite, or friendly. The second scale is for cities with
whom your race relation (not city status) with the population is wary or hate.

+---------------------------+-------------------------+---------------------+
| City Status for Friendly, | City Status Point Range | Chance of Rebellion |
| Polite, and Neutral Race  |                         | per Turn            |
| Relations                 |                         |                     |
+===========================+=========================+=====================+
| Cheerful                  | 80 - 100                |  0%                 |
+---------------------------+-------------------------+---------------------+
| Content                   | 60 -  79                |  0%                 |
+---------------------------+-------------------------+---------------------+
| Stable                    | 40 -  59                |  0%                 |
+---------------------------+-------------------------+---------------------+

+-------------------------+-------------------------+---------------------+
| City Status for Hate,   | City Status Point Range | Chance of Rebellion |
| and Wary Race Relations |                         | per Turn            |
+=========================+=========================+=====================+
| Enslaved                | 80 - 100                |  0%                 |
+-------------------------+-------------------------+---------------------+
| Oppressed               | 60 -  79                |  0%                 |
+-------------------------+-------------------------+---------------------+
| Stable                  | 40 -  59                |  0%                 |
+-------------------------+-------------------------+---------------------+
| Unrest                  | 20 -  39                | 10%                 |
+-------------------------+-------------------------+---------------------+
| Unruly                  |  0 -  19                | 50%                 |
+-------------------------+-------------------------+---------------------+

=======================  ============================
Name                     City Relation Point Modifier
=======================  ============================
Friendly Terrain                     +50
Hostile Terrain                      +50
Wooden Wall                          +50
Stone Wall                           +50
Hate relation                        +50
Wary relation                        +50
Neutral relation                     +50
Polite relation                      +50
Friendly relation                    +50
Strong occupied forces               +50
Average occupied forces              +50
Weak occupied forces                 +50
=======================  ============================

Racial Friendly/Hostile Terrains for City Status/Unit Morale
============================================================

=========  ==========================  ==========================
Race Name  Racial Friendly Terrains    Racial Hostile Terrains
=========  ==========================  ==========================
Acrac      Desert                      Snow, Ice, Underground Ice
Dark Elf   Dirt                        None
Dwarf      Dirt                        None
Elf        Grass                       Wasteland
Frostling  Snow, Ice, Underground Ice  Desert
Goblin     Dirt                        None
Halfling   Grass                       Wasteland
Highman    None                        None
Human      None                        None
Lizardman  Water                       Desert, Wasteland
Orc        None                        None
Undead     Wasteland                   Grass
=========  ==========================  ==========================

Upkeep Costs
============

=============  ===============================
Name                 Gold Upkeep per Turn
=============  ===============================
Level 1 Unit                  4
Level 2 Unit                  6
Level 3 Unit                  8
Level 4 Unit                 10
Hero Upkeep    5 + (2 * Hero Experience Level)
Leader Upkeep                 0
=============  ===============================

===============  ====================
Name             Mana Upkeep per Turn
===============  ====================
Air Elemental             12
Black Dragon              12
Black Spider               6
Earth Elemental           12
Fire Elemental            12
Fire Sprite                6
Giant Frog                 4
Gold Dragon                8
Great Eagle                6
Water Elemental           12
Wild Boar                  4
===============  ====================

Attack Ranges
=============

==========  ==============
Range Name  Range in Hexes
==========  ==============
Touch              0
Melee              0
Short              4
Medium             8
Long              12
==========  ==============

Attack Abilities
================

+---------------------+--------+----------+----------+------------+------------+
| Name                | Range  | Attack   | Damage   | Repetition | Type       |
+=====================+========+==========+==========+============+============+
| Archery             | Medium |      4   |      2   |          2 | Physical   |
+---------------------+--------+----------+----------+------------+------------+
| Black Bolts         | Medium |      6   |      3   |          1 | Death      |
+---------------------+--------+----------+----------+------------+------------+
| Black Breath        | Short  |      7   |      5   |          1 | Death      |
+---------------------+--------+----------+----------+------------+------------+
| Call Flames         | Medium |      5   |      4   |          1 | Fire       |
+---------------------+--------+----------+----------+------------+------------+
| Charm               | Touch  |      5   |      X   |          1 | Special    |
+---------------------+--------+----------+----------+------------+------------+
| Cold Breath         | Short  |      7   |      5   |          1 | Cold       |
+---------------------+--------+----------+----------+------------+------------+
| Cold Strike         | Melee  | Unit’s   | Unit’s   |          2 | Cold,      |
|                     |        | Attack   | Damage   |            | Physical   |
+---------------------+--------+----------+----------+------------+------------+
| Death Strike        | Melee  | Unit’s   | Unit’s   |          2 | Death,     |
|                     |        | Attack   | Damage   |            | Physical   |
+---------------------+--------+----------+----------+------------+------------+
| Divine Breath       | Short  |      7   |      5   |          1 | Holy       |
+---------------------+--------+----------+----------+------------+------------+
| Dominate            | Touch  |      6   |      X   |          1 | Special    |
+---------------------+--------+----------+----------+------------+------------+
| Doom Gaze           | Long   |      6   |      5   |          1 | Death      |
+---------------------+--------+----------+----------+------------+------------+
| Entangle            | Touch  |      7   |      X   |          1 | Special    |
+---------------------+--------+----------+----------+------------+------------+
| Entagle Strike      | Melee  | Unit's   | Unit's   |          2 | Entangle,  |
|                     |        | Attack   | Damage   |            | Physical   |
+---------------------+--------+----------+----------+------------+------------+
| Fire Breath         | Short  |      7   |      5   |          1 | Fire       |
+---------------------+--------+----------+----------+------------+------------+
| Fire Cannon         | Long   |      5   |      8   |          1 | Physical,  |
|                     |        |          |          |            | Wall       |
+---------------------+--------+----------+----------+------------+------------+
| Fire Musket         | Long   |      7   |      5   |          1 | Physical   |
+---------------------+--------+----------+----------+------------+------------+
| Fire Strike         | Melee  | Unit's   | Unit's   |          2 | Fire,      |
|                     |        | Attack   | Damage   |            | Physical   |
+---------------------+--------+----------+----------+------------+------------+
| Flame Throwing      | Short  |      4   |      3   |          1 | Fire       |
+---------------------+--------+----------+----------+------------+------------+
| Frost Bolts         | Medium |      6   |      3   |          1 | Cold       |
+---------------------+--------+----------+----------+------------+------------+
| Holy Bolts          | Medium |      6   |      3   |          1 | Holy       |
+---------------------+--------+----------+----------+------------+------------+
| Holy Strike         | Melee  | Unit's   | Unit's   |          2 | Holy,      |
|                     |        | Attack   | Damage   |            | Physical   |
+---------------------+--------+----------+----------+------------+------------+
| Hurl Boulder        | Long   |      3   |      7   |          1 | Physical,  |
|                     |        |          |          |            | Wall       |
+---------------------+--------+----------+----------+------------+------------+
| Hurl Boulder        | Medium |      3   |      1   |          4 | Physical   |
+---------------------+--------+----------+----------+------------+------------+
| Invoke Death        | Touch  |      6   |      X   |          1 | Death      |
+---------------------+--------+----------+----------+------------+------------+
| Lightning Bolts     | Medium |      6   |      3   |          1 | Lightning  |
+---------------------+--------+----------+----------+------------+------------+
| Lightning Strike    | Melee  | Unit's   | Unit's   |          2 | Lightning, |
|                     |        | Attack   | Damage   |            | Physical   |
+---------------------+--------+----------+----------+------------+------------+
| Magic Bolts         | Medium |      7   |      3   |          1 | Magic      |
+---------------------+--------+----------+----------+------------+------------+
| Magic Strike        | Melee  | Unit's   | Unit's   |          2 | Magic,     |
|                     |        | Attack   | Damage   |            | Physical   |
+---------------------+--------+----------+----------+------------+------------+
| Poison Darts        | Medium |      4   |      1   |          3 | Poison     |
+---------------------+--------+----------+----------+------------+------------+
| Poison Strike       | Melee  | Unit's   | Unit's   |          2 | Physical,  |
|                     |        | Attack   | Damage   |            | Poison     |
+---------------------+--------+----------+----------+------------+------------+
| Possess             | Touch  |      5   |      X   |          1 | Special    |
+---------------------+--------+----------+----------+------------+------------+
| Round Attack        | Melee  | Unit's   | Unit's   |          2 | Physical   |
|                     |        | Attack   | Damage   |            |            |
+---------------------+--------+----------+----------+------------+------------+
| Seduce              | Touch  |      4   |      X   |          1 | Special    |
+---------------------+--------+----------+----------+------------+------------+
| Self Destruct       | Touch  |      7   |      6   |          1 | Fire, Wall |
+---------------------+--------+----------+----------+------------+------------+
| Shoot Black Javelin | Long   |      5   |      5   |          2 | Death,     |
|                     |        |          |          |            | Physical   |
+---------------------+--------+----------+----------+------------+------------+
| Shoot Javelin       | Long   |      5   |      5   |          2 | Physical   |
+---------------------+--------+----------+----------+------------+------------+
| Strike              | Melee  | Unit's   | Unit's   |          2 | Physical   |
|                     |        | Attack   | Damage   |            |            |
+---------------------+--------+----------+----------+------------+------------+
| Turn Undead         | Touch  | 3 + Turn | 3 + Turn |          1 | Special    |
|                     |        | Level    | Level    |            |            |
+---------------------+--------+----------+----------+------------+------------+
| Venomous Spit       | Short  |        5 |        4 |          1 | Poison     |
+---------------------+--------+----------+----------+------------+------------+
| Wall Crushing       | Touch  |        6 |        6 |          1 | Special    |
+---------------------+--------+----------+----------+------------+------------+
| Web                 | Touch  |        4 |        X |          1 | Special    |
+---------------------+--------+----------+----------+------------+------------+

Attack Ability Types
====================

=========  ===============================================================
Name       Attack Effects
=========  ===============================================================
Entagle    Traps enemy in vines for 3 combat turns
Fir        Sets target aflame for 3 combat turns
Cold       Freezes the target for 3 combat turns
Death      Curses the target for 3 days
Holy       Curses target to suffer from vertigo for the duration of combat
Lightning  Stuns the target for 1 combat turn
Magic      None
Physical   None
Poison     Poisons target for 3 days
Wall       None, but attack can damage walls
=========  ===============================================================

Combat Mechanics
================
Each attack, spell, and ability will compare two stats and make a random roll
to see if it succeeds. Some forms of attacks, abilities, and spells require
multiple successful rolls to have any effect. Some attacks, abilities, and
spells with multiple, different effects will make separate, individual rolls
for each effect and apply only the effects that had successful rolls. Most
rolls involve only the comparison of two stats to determine chance of success.

+-------------------------------------+----------------------------------------+
| Default chance of success           |                  50%                   |
+-------------------------------------+----------------------------------------+
| Difference in stats being compared  | +10% for each point Attacker is higher |
| (Attacker's Stat - Defender's Stat) | -10% for each point Defender is higher |
+-------------------------------------+----------------------------------------+
| Minimum Chance of Success           |                  10%                   |
+-------------------------------------+----------------------------------------+
| Maximum Chance of Success           |                  90%                   |
+-------------------------------------+----------------------------------------+

Damage rolls are calculated differently. Damage is calculated and applied
immediately after each successful hit, before any other rolls. Defender’s
current Hit Points are subtracted by the final Damage amount. Units “die”
immediately when their current Hit Points reach 0. A unit’s listed Damage stat
is only used for melee damage calculations. Any shown Damage stat applies
individually to each missile or melee strike in a volley. Attacks with high
Attack stats (5+ greater than the targets Defense stat) have a higher Minimum
Damage, but do not have a higher Maximum Damage.

+--------------------------------+---------------------------------------------+
| Minimum Damage for when Attack |                                             |
| stat is not 5 greater than     |                      1                      |
| Defense Stat                   |                                             |
+--------------------------------+---------------------------------------------+
| Minimum Damage for when Attack | Minimum Damage increases above 1, but never |
| stat is 5 or more greater than | exceeds Max Damage. The more the Attack     |
| Defense Stat                   | stat exceeds the Defense stat by 5, the     |
|                                | more Minimum Damage is raised.              |
+--------------------------------+---------------------------------------------+
| Maximum Damage                 | Stat listed on attacker's ability, spell    |
|                                | or unit. Archery's Damage stat = 2 = Max    |
|                                | Damage 2. Solar Flare Damage stat = 4 = Max |
|                                | Damage 4. Unit's Damage stat = 3 = Max      |
|                                | Damage 4.                                   |
+--------------------------------+---------------------------------------------+
| Defender has Protection verus  | Half (50%) Damage applied                   |
| the Attack type.               |                                             |
+--------------------------------+---------------------------------------------+
| Defender has Immunity versus   | No (0%) Damage applied                      |
| the Attack type                |                                             |
+--------------------------------+---------------------------------------------+

Builder’s Guild and Shipyard Units
==================================

+-----------+--------+---------+------------+--------+------+-------+------------------+
| Name      | Attack | Defense | Resistance | Damage | Hits | Moves | Abilities        |
+===========+========+=========+============+========+======+=======+==================+
| Builder   |      1 |       2 |          3 |      1 |    5 |    20 | Walking,         |
|           |        |         |            |        |      |       | Cold             |
|           |        |         |            |        |      |       | Immunity, Poison |
|           |        |         |            |        |      |       | Immunity,        |
|           |        |         |            |        |      |       | Fearless,        |
|           |        |         |            |        |      |       | Construct        |
+-----------+--------+---------+------------+--------+------+-------+------------------+
| Drill     |      2 |       2 |          2 |      3 |    6 |    10 | Walking, Poison  |
|           |        |         |            |        |      |       | Immunity,        |
|           |        |         |            |        |      |       | Tunneling,       |
|           |        |         |            |        |      |       | Fearless, Cold   |
|           |        |         |            |        |      |       | Protection, Wall |
|           |        |         |            |        |      |       | Crushing         |
+-----------+--------+---------+------------+--------+------+-------+------------------+
| Flame     |      1 |       2 |          3 |      1 |    8 |    20 | Walking, Poison  |
| Thrower   |        |         |            |        |      |       | Immunity, Flame  |
|           |        |         |            |        |      |       | Throwing,        |
|           |        |         |            |        |      |       | Fearless,        |
|           |        |         |            |        |      |       | Cold Protection  |
+-----------+--------+---------+------------+--------+------+-------+------------------+
| Dragon    |      1 |       3 |          3 |      1 |   20 |    32 | Sailing, Poison  |
| Ship      |        |         |            |        |      |       | Immunity, Shoot  |
|           |        |         |            |        |      |       | Javelin, Vision  |
|           |        |         |            |        |      |       | II, Fearless,    |
|           |        |         |            |        |      |       | Cold Protection  |
+-----------+--------+---------+------------+--------+------+-------+------------------+
| Galley    |      1 |       3 |          3 |      1 |   25 |    36 | Sailing, Shoot   |
|           |        |         |            |        |      |       | Javelin, Vision  |
|           |        |         |            |        |      |       | II, Fearless,    |
|           |        |         |            |        |      |       | Cold, Protection |
+-----------+--------+---------+------------+--------+------+-------+------------------+
| Galleon   |      1 |       3 |          3 |      1 |   30 |    40 | Sailing, Poison  |
|           |        |         |            |        |      |       | Immunity,        |
|           |        |         |            |        |      |       | Marksmanship I,  |
|           |        |         |            |        |      |       | Shoot Javelin,   |
|           |        |         |            |        |      |       | Vision II,       |
|           |        |         |            |        |      |       | Fearless, Cold   |
|           |        |         |            |        |      |       | Protection       |
+-----------+--------+---------+------------+--------+------+-------+------------------+
| Transport |      1 |       3 |          3 |      1 |   20 |    28 | Sailing, Poison  |
| Ship      |        |         |            |        |      |       | Immunity, Vision |
|           |        |         |            |        |      |       | II, Fearless,    |
|           |        |         |            |        |      |       | Cold Protection  |
+-----------+--------+---------+------------+--------+------+-------+------------------+

Summoned Units
==============

+-----------+--------+---------+------------+--------+------+-------+------------------+
| Name      | Attack | Defense | Resistance | Damage | Hits | Moves | Abilities        |
+===========+========+=========+============+========+======+=======+==================+
| Air       |    5   |    2    |      3     |    3   |  12  |   32  | Flying, Fire     |
| Elemental |        |         |            |        |      |       | Immunity, Cold,  |
|           |        |         |            |        |      |       | Immunity,        |
|           |        |         |            |        |      |       | Lightning        |
|           |        |         |            |        |      |       | Immunity, Poison |
|           |        |         |            |        |      |       | Immunity,        |
|           |        |         |            |        |      |       | Physical         |
|           |        |         |            |        |      |       | Immunity,        |
|           |        |         |            |        |      |       | Strike, Vision   |
|           |        |         |            |        |      |       | II               |
+-----------+--------+---------+------------+--------+------+-------+------------------+
| Black     |    6   |    5    |      8     |    6   |  20  |   32  | Flying, Death    |
| Dragon    |        |         |            |        |      |       | Immunity,        |
|           |        |         |            |        |      |       | Strike, Vision   |
|           |        |         |            |        |      |       | II, Fearless,    |
|           |        |         |            |        |      |       | Poison,          |
|           |        |         |            |        |      |       | Protection,      |
|           |        |         |            |        |      |       | Black Breath     |
+-----------+--------+---------+------------+--------+------+-------+------------------+
| Black     |    4   |    3    |      4     |    3   |   6  |   28  | Walking, Cave    |
|           |        |         |            |        |      |       | Crawling, Poison |
|           |        |         |            |        |      |       | Immunity, Poison |
|           |        |         |            |        |      |       | Strike, Strike,  |
|           |        |         |            |        |      |       | Web, Wall        |
| Spider    |        |         |            |        |      |       | Climbing         |
+-----------+--------+---------+------------+--------+------+-------+------------------+
| Earth     |    6   |    4    |      4     |    8   |  20  |   20  | Walking, Cave    |
| Elemental |        |         |            |        |      |       | Crawling,        |
|           |        |         |            |        |      |       | Mountaineering,  |
|           |        |         |            |        |      |       | Poison Immunity, |
|           |        |         |            |        |      |       | Strike,          |
|           |        |         |            |        |      |       | Tunneling, Fire  |
|           |        |         |            |        |      |       | Protection,      |
|           |        |         |            |        |      |       | Lightning        |
|           |        |         |            |        |      |       | Protection,      |
|           |        |         |            |        |      |       | Wall Crushing    |
+-----------+--------+---------+------------+--------+------+-------+------------------+
| Fire      |    6   |    4    |      8     |    5   |  17  |   26  | Walking, Fire    |
| Elemental |        |         |            |        |      |       | Immunity,        |
|           |        |         |            |        |      |       | Lightning, Fire  |
|           |        |         |            |        |      |       | Strike, Strike,  |
|           |        |         |            |        |      |       | Call Flames,     |
|           |        |         |            |        |      |       | Ignition,        |
|           |        |         |            |        |      |       | Physical         |
|           |        |         |            |        |      |       | Protection       |
+-----------+--------+---------+------------+--------+------+-------+------------------+
| Fire      |    5   |    2    |      4     |    3   |   7  |   24  | Walking, Fire,   |
| Sprite    |        |         |            |        |      |       | Immunity,        |
|           |        |         |            |        |      |       | Lightning        |
|           |        |         |            |        |      |       | Immunity, Poison |
|           |        |         |            |        |      |       | Immunity, Fire   |
|           |        |         |            |        |      |       | Strike, Strike,  |
|           |        |         |            |        |      |       | Ignition         |
+-----------+--------+---------+------------+--------+------+-------+------------------+
| Giant     |    4   |    1    |      2     |    3   |   5  |   28  | Walking,         |
| Frog      |        |         |            |        |      |       | Swimming, Strike |
+-----------+--------+---------+------------+--------+------+-------+------------------+
| Gold      |    5   |    7    |      8     |    5   |  20  |   32  | Flying, Holy     |
| Dragon    |        |         |            |        |      |       | Immunity,        |
|           |        |         |            |        |      |       | Strike, Vision   |
|           |        |         |            |        |      |       | II, Fearless,    |
|           |        |         |            |        |      |       | Fire Protection, |
|           |        |         |            |        |      |       | Divine Breath    |
+-----------+--------+---------+------------+--------+------+-------+------------------+
| Great     |    4   |    2    |      3     |    3   |   7  |   40  | Flying, Strike,  |
| Eagle     |        |         |            |        |      |       | Vision II        |
+-----------+--------+---------+------------+--------+------+-------+------------------+
| Water     |    6   |    2    |      3     |    4   |  14  |   24  | Walking,         |
| Elemenatl |        |         |            |        |      |       | Swimming,        |
|           |        |         |            |        |      |       | Strike,          |
|           |        |         |            |        |      |       | Physical         |
|           |        |         |            |        |      |       | Protection,      |
|           |        |         |            |        |      |       | Water            |
|           |        |         |            |        |      |       | Concealment      |
+-----------+--------+---------+------------+--------+------+-------+------------------+
| Wid Boar  |    3   |    3    |      3     |    3   |   5  |   36  | Walking, Strike, |
|           |        |         |            |        |      |       | Charge           |
+-----------+--------+---------+------------+--------+------+-------+------------------+

Terrain and Movement Types
==========================

================================================  ===================
Terrain and Mevement Types                        Movement Point Cost
================================================  ===================
City                                                       3
Desert                                                     4
Dirt (with Cave Crawling ability)                        4 (3)
Flying and Floating (over Mountains)                     4 (8)
Forest (with Forestry ability)                           6 (4)
Grassland                                                  4
Hill (with Mountaineering skill)                         6 (3)
Ice                                                        4
Lava (with Fire Halo spell enchantment)              Impassable (4) 
Mountain (with Mountaineering ability)               Impassable (8)
Road (with Enchant Roads spell in effect)                3 (2)
Snow                                                       4
Steppe                                                     4
Tunneling (Actually digging the tunnel, per hex)          10
Water (Swimming and Sailing movement types)                4
================================================  ===================

Experience and Gaining Levels
=============================

Units and Heroes alike each earn experience when they deliver the killing
attack to a unit. Each unit is worth experience equal to its level when killed.

*Examples:*
A Dwarf Axeman, a level 1 unit, is worth 1 experience when killed. A Goblin
Karagh, a level 4 unit, is worth 4 experience when killed.  Units require 2 x
their level to earn a silver medal and 6 x their level to earn a gold medal.

*Example:*
An Elf Archer, a level 1 unit, require 2 experience to earn its silver medal and
6 experience to earn its gold medal.

=======================  ==========================  =========================
Unit Experience Level    Earned Experience Required  Unit Stat Modifiers
=======================  ==========================  =========================
Silver Medal [Veteran]   2 x Level of Unit           +1 Attack, +1 Defense,
                                                     +1 Hit Point
Gold Medal [Elite]       6 x Level of Unit           +1 Damage, +1 Resistance,
                                                     +1 Hit Point
=======================  ==========================  =========================

Additionally, some units gain abilities when they earn their medals, but this
varies from unit to unit. Most archer / ranged units gain the Marksmanship
ability or increase their current Marksmanship ability when they earn medals.

Heroes and Leaders require more experience to level up. Heroes and Leaders
receive 1 experience at the start of each new turn. When a Hero or Leader levels
up, they earn 10 skill points to spend. These points are stored up over multiple
levels if not spent.

==========  ============================
Hero Level  Experience Required to Level
==========  ============================
  1 - 10            15 per level
 11 - 20            20 per level
 21 - 30            25 per level
==========  ============================

==========  =======================
Stat Name   Skill Point Cost Per +1
==========  =======================
Attack                 5
Defense                5
Damage                10
Resistance             5
Movement               2
Hit Point              5
==========  =======================

Spell Spheres and Mana Node Generation
======================================

======================  ========================================
Number of Sphere Picks  Mana Generation per Matching Sphere Node
======================  ========================================
       0                                 0
       1                                15
       2                                20
       3                                25
       4                                30
**Power Node**             **10 [Regardless of Sphere Picks]**
======================  ========================================

Spell Reference Lists
=====================
Currently, only the Ranges, Attack, and Damage values of combat spells are
listed. Radius stands for the number of hexes outward from the center hex the
spell covers. Radius 0 spells affect only 1 hex, radius 1 is 7 hexes, radius 2
is 19 hexes, radius 3 is 37 hexes, and radius 4 is 61 hexes. Cone spells affect
12 hexes in a triangle shaped pattern.

Life Spells
-----------

==============  =====  ======  ======  ==========  ======  ====
Name            Range  Attack  Damage  Repetition  Radius  Type
==============  =====  ======  ======  ==========  ======  ====
Rejuvenate      \-\      \-\     \-\       \-\          2  \-\
Solar Flare     Long        8       4           1       0  Holy
Holy Woods      \-\      \-\     \-\       \-\          1  Holy
Turn Undead     Long        6       5           1       0  \-\
Recall Spirits  \-\      \-\     \-\       \-\          1  \-\
Sacred Wrath    \-\         5       5           1    \-\   Holy
Divine Storm    \-\      \-\     \-\       \-\          4  Holy
==============  =====  ======  ======  ==========  ======  ====

Death Spells
------------

================  =====  ======  ======  ==========  ======  =======
Name              Range  Attack  Damage  Repetition  Radius  Type
================  =====  ======  ======  ==========  ======  =======
Death Ray         Long        8       4           1       0  Death
Disease Cloud     Long        8       3           1       1  Death
Evil Woods        \-\      \-\     \-\         \-\        1  Death
Animate Dead      \-\      \-\     \-\         \-\        1  \-\
Terror            \-\         5    \-\            1    \-\   Special
Pestilence Cloud  \-\      \-\     \-\         \-\        2  Poison
Mind Decay        Long        5    \-\            1       1  Special
Death Storm       \-\      \-\     \-\         \-\        4  Death
================  =====  ======  ======  ==========  ======  =======

Air Spells
----------

===============  =====  ======  ======  ==========  ======  =========
Name             Range  Attack  Damage  Repetition  Radius  Type
===============  =====  ======  ======  ==========  ======  =========
Vaporize         Long       7        5           1       0  Physical
Chain Lightning  Long       6        5  Special          0  Lightning
Winds of Fury    Long    8(10)       5           1       0  Physical
Freeze Water     \-\     \-\      \-\   \-\              1  \-\
Cold Breath      Short      6        5           1  Cone    Cold
Shockwave        0          8        5           1       3  Physical,
                                                            Wall
Lightning Storm  \-\     \-\      \-\   \-\              1  Lightning
===============  =====  ======  ======  ==========  ======  =========

Earth Spells
------------

=============  ======  ======  ======  ==========  =======  =========
Name           Range   Attack  Damage  Repetition  Radius   Type
=============  ======  ======  ======  ==========  =======  =========
Entangle       Long         7    \-\            1     \-\   Entagle
Slow           \-\          9    \-\            1     \-\   Special
Poison Woods   \-\       \-\     \-\         \-\         1  Poison
Stoning        Long         5       2           6        0  Physical
Level Terrain  \-\       \-\     \-\         \-\         1  \-\
Tremors        \-\          5       5           1     \-\   Physical,
                                                            Wall
Raise Terrain  \-\       \-\     \-\         \-\         1  \-\
=============  ======  ======  ======  ==========  =======  =========

Fire Spells
-----------

=================  ======  ======  ======  ==========  =======  ==============
Name               Range   Attack  Damage  Repetition  Radius   Type
=================  ======  ======  ======  ==========  =======  ==============
Flame Arrow        Long         8       4           1        0  Fire, Physical
Call Flames        Medium       9       3           1        0  Fire
Cloud of Ashes     \-\       \-\     \-\         \-\         3  \-\
Fire Breath        Short        6       5           1  Cone     Fire
Swarm              Long         6       1  Special     Special  Fire
Fire Barrier       \-\       \-\     \-\         \-\         1  Fire
Fireball           Long         8       6           1        1  Fire, Wall
Sacrificial Flame  Long         8       5           1        2  Fire
Fire Storm         \-\       \-\     \-\         \-\         1  Fire
=================  ======  ======  ======  ==========  =======  ==============

Water Spells
------------

===============  ======  ======  ======  ==========  ======  ==============
Name             Range   Attack  Damage  Repetition  Radius  Type
===============  ======  ======  ======  ==========  ======  ==============
Ice Shards       Long         6       5           3       0  Physical
Ooze             Medium    \-\     \-\         \-\        2  \-\
Vortex           \-\       \-\     \-\            1       0  Physical
Geyser           Long         9       5           1       0  Physical
Frost Beam       Long         9       3           1       1  Cold
Great Hail       Long         5       5           3       1  Physical, Wall
Healing Showers  \-\       \-\     \-\         \-\        1  \-\
Ice Showers      \-\       \-\     \-\         \-\        4  Cold
===============  ======  ======  ======  ==========  ======  ==============

All global, terrain altering spells without an upkeep cost last for three turns.
Level Terrain, Animate Ruins, and Rejuvenate are exceptions to this rule. This
includes all cloud spells (except Cloud of Ashes which has an upkeep cost) and
all Holy Woods type spells.

Manual Corrections and Updates
==============================
(If you mark any corrections in the Manual, USE PENCIL, these changes are not
necessarily final!)

Spells
------

Hold Champion, Life, 2nd Level
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
*Corrected Description:* Gives a +2 bonus to Attack and Damage against units of
Evil alignment.

High Prayer, Life, 3rd Level
~~~~~~~~~~~~~~~~~~~~~~~~~~~~
*Corrected Description:* Blesses all friendly units during combat, increasing
Defense (+1), Resistance (+1), and restores up to 5 lost Hit Points.

Evil Champion, Death, 2nd Level
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
*Corrected Name:* Unholy Champion*

*Corrected Description:*
Gives a +2 bonus to Attack and Damage against units of Good alignment.

Terror, Death, 2nd Level
~~~~~~~~~~~~~~~~~~~~~~~~
*Corrected Description:*
All enemy units that fail a resist roll are panicked, severely hurting their
morale (- 40 to morale) for the duration of combat.

Mind Decay, Death, 3rd Level
~~~~~~~~~~~~~~~~~~~~~~~~~~~~
*Corrected Description:*
Evil spirits attempt to Dominate non-Undead units. If successful, the victims
fall under your control and loose 1 hit point per turn. At the end of combat,
all units controlled by Mind Decay die.

Haste, Air, 1st Level
~~~~~~~~~~~~~~~~~~~~~
*Corrected Description:*
All terrain types require 2 less movement points to move over, down to a minimum
of 2.

Winds of Fury, Air, 2nd Level
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
*Added effect with versions 1.1+:*
Receives a +2 Attack bonus when targeting flying units.

Wind Walking, Air, 3rd Level
~~~~~~~~~~~~~~~~~~~~~~~~~~~~
*Corrected Description:*
Gives enchanted unit the ability to Float over terrain.

Entangle, Earth, 2nd Level
~~~~~~~~~~~~~~~~~~~~~~~~~~
*Updated Description:*
Attempts to entangle the target in vines, paralyzing them for 3 combat turns and
lowering their defense by 2.

Stoning, Earth, 2nd Level
~~~~~~~~~~~~~~~~~~~~~~~~~
*Corrected Description:*
Sends 6 small stones towards an enemy during combat, each with a separate chance
to hit and do damage.

Concealment, Earth, 3rd Level
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
*Updated Description:*
Allows the enchanted unit to hide in forests and thick underbrush. While hidden,
the enchanted unit may only be seen on the global map by enemies directly
adjacent to the unit or by units with True Seeing.

Level Terrain, Earth, 3rd Level
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
*Updated Description:*
Permanently lowers mountains, hills, forests, and underbrush to flatter, more
easily passable terrain.

Fire Mastery, Fire, 4th Level
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
*Added Bonus:*
Also bestows Fire Protection upon all of the caster’s units.

Warmonger, Fire, 4th Level
~~~~~~~~~~~~~~~~~~~~~~~~~~
*Clarification:*
Veteran experience level is the same as if the unit earned the experience for a
Silver Medal. (+1 to Attack, Defense, and Hit Points)

Dispel Magic, Cosmos, 1st Level
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
*Clarification:*
Can also be used to randomly remove an enchantment on an enemy unit.

Units
-----

Elves
~~~~~

Nymph
"""""
:Correction: Damage = 2

Ranger
""""""
:Correction: Movement = 32

Nature Elemental
""""""""""""""""
:Updated, Version 1.2+: Entagle ability removed, Entagle Strike added, Healing
                        ability added

Halflings
~~~~~~~~~

Rogue
"""""
:Correction: Parry ability added

Centaur
"""""""
:Correction: Defense = 4

Dwarves
~~~~~~~~

Berserker
"""""""""
:Update, Version 1.1+: Cave Crawling ability added

Boar Rider
""""""""""
:Correction: Defense = 3

Balloon
"""""""
:Clarification: Transports 5 units

Mole
""""
:Error: Unit picture is that of the Drill unit
:Correction: Unit picture is that of a giant Mole (No weasels or badgers, just a
             mole)

Highmen
~~~~~~~

Avenger
"""""""
:Correction: Damage = 4

Human
~~~~~

Air Galley
""""""""""
:Clarification: Transports 7 units

Azracs
~~~~~~

Swordsman
"""""""""
:Correction: Damage = 3

Elephant
""""""""
:Update, Version 1.2+: Attack = 3, Damage = 3

Lizardmen
~~~~~~~~~

Green Wyvern
""""""""""""
:Update, Version 1.2+: Cost = 98

Basilisk
""""""""
:Update, Version 1.2+: Cost = 202

Dark Elves
~~~~~~~~~~

Executioner
"""""""""""
:Correction: Defense = 5

Goblins
~~~~~~~

Big Beetle
""""""""""
:Update, Version 1.2+: Night Vision ability added

Undead
~~~~~~

Swordsman
"""""""""
:Correction: Death Immunity ability added

Skull Thrower
"""""""""""""
:Correction: Movement = 20, Death Immunity ability added

Demon
"""""
:Correction: Attack = 6, Poison Strike ability removed, Fire Strike ability
             added Reaper
:Update, Version 1.2+: Defense = 5, Damage = 6

