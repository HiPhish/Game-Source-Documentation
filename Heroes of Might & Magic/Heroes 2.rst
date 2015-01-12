.. header:: Game Source Documentation project - Heroes of Might & Magic II

.. See the very bottom of this docuent on format conventions used. If follows
   the reStructuredText syntax.

===============================================
Heroes of Might & Magic II code design document
===============================================

Analysing and documenting the source code workings of Heroes of Might & Magic
II for DOS and Windows.

.. contents:: Table of Contents
   :depth: 3

--------------------------------------------------------------------------------

The source code for Heroes of Might & Magic has never been released to the
public, so it is questionable whether this docuent will ever reach a usable
state. However, there is a project called the Free Heroes 2 Engine that is
trying to reverse engineer the inner workings, so their knowledge will be
helpful in this endevour. The other major source of information is the player's
manual, it list the statistics and a bulk of the game's rules.

=====================
Part I - File Formats
=====================

The file formats can be learned from the Free Heroes 2 Engine.

====================
Part II - Game Rules
====================

This includes general rules of gameplay, as well as rules for AI, map
generation and random numbers.

==============================
Part III - Tables & Statistics
==============================

This information is straight from the player manual.

Maps and terrain
================

Movement Cost
-------------
Heroes move at the speed of the slowest unit in their army unless they are in a
ship, in which case they move at the same speed. The *Logistics* secondary
skill increases the heroes' land movement and the *Navigation* secondary skill
increases the heroes' water movement. Heroes have a penalty to their movement
when on rough terrains, but this penalty can be offset by having the
*Pathfinding* secondary skill. The effects of terrain and *Pathfinding* are
expressed in the chart below by percent of the penalties to normal movement
(200% means you move at half speed).

=======  ====  =====  ========  ======
Terrain  None  Basic  Advanced  Expert
=======  ====  =====  ========  ======
Desert   200%   175%      150%    100%
Swamp    175%   150%      125%    100%
Snow     175%   150%      100%    100%
Cracked  125%   100%      100%    100%
Beach    125%   100%      100%    100%
Lava     100%   100%      100%    100%
Water    100%   100%      100%    100%
Dirt     100%   100%      100%    100%
Grass    100%   100%      100%    100%
Road      75%    75%       75%     75%
=======  ====  =====  ========  ======

Notice that you get a 25% bonus to movement while traveling on a road. Moving
on a road is faster than moving on grass or dirt, and the roads make the travel
over rough terrains very easy. *Pathfinding* becomes one of the most useful
skills on a map with large areas of rough terrain.

Spells
======
asdf

Level One Spells
----------------
asdf

Level Two Spells
----------------
asdf

Level Three Spells
------------------
asdf

Level Four Spells
-----------------
asdf

Level Five Spells
-----------------
asdf

Heroes - Classes, Castles and Units
==============================================

Classes of Heroes
-----------------
Each hero will have a different attributes and skill. Heroes start with a few
experience points, a small number of creatures, and the follwing statistics:

+-----------------+--------+---------+-------------+-----------+-----------+---------------+
| Class           | Attack | Defense | Spell Power | Knowledge | Spells    | Skill         |
+=================+========+=========+=============+===========+===========+===============+
| **Barbarian**   |      3 |       1 |           1 |         1 | None      | Pathfinding+, |
+-----------------+--------+---------+-------------+-----------+-----------+---------------+
| **Knight**      |      2 |       2 |           1 |         1 | None      | Ballistics,   |
|                 |        |         |             |           |           | Leadership    |
+-----------------+--------+---------+-------------+-----------+-----------+---------------+
| **Necromancer** |      1 |       0 |           2 |         2 | Haste     | Wisdom,       |
|                 |        |         |             |           |           | Necromancy    |
+-----------------+--------+---------+-------------+-----------+-----------+---------------+
| **Sorceress**   |      0 |       0 |           2 |         3 | Bless     | Wisdom,       |
|                 |        |         |             |           |           | Navigation+   |
+-----------------+--------+---------+-------------+-----------+-----------+---------------+
| **Warlock**     |      0 |       0 |           3 |         2 | Curse     | Wisdom,       |
|                 |        |         |             |           |           | Scouting+     |
+-----------------+--------+---------+-------------+-----------+-----------+---------------+
| **Wizard**      |      0 |       1 |           2 |         2 | Stoneskin | Wisdom+       |
+-----------------+--------+---------+-------------+-----------+-----------+---------------+

A hero with no spells has no spell book either, but they can buy one from the
mage's guild for 500 gold. A skill with a plus sign denotes an *advanced* skill
and two plus signs denote a *mastery* skill.

Skill Advancement
-----------------
Skills are advanced by advancing in levels. When a hero advances a level, a
screen will appear giving the hero a bonus to a primary skill (*Attack*,
*Defense*, *Spell Power*, or *Knowledge*), and the choice between two secondary
skills. One or both may be skills already known by the hero which the hero can
then advance in, otherwise the skill(s) are new to the hero and are learned at
the Basic level. The skills a hero has to choose from are randomly selected,
weighted by the class of the hero.

Primary Skill Advancement
~~~~~~~~~~~~~~~~~~~~~~~~~
The table below gives the percentage chance of learning a primary skill when
going up each level. For the first nine levels heroes tend to be specialised in
one or two skills, but at tenth level and beyond they generalise much more.

===============  ======  =======  =====  =========
Class and Level  Attack  Defense  Power  Knowledge
===============  ======  =======  =====  =========
Barbarian 2-9       55%      35%     5%         5%
Barbarian 10+       30%      50%    20%        20%

Knight 2-9          35%      45%    10%        10%
Knight 10+          25%      25%    25%        25%

Necromancer 2-9     15%      15%    35%        35%
Necromancer 10+     25%      25%    25%        25%

Sorceress 2-9       10%      10%    30%        50%
Sorceress 10+       20%      20%    30%        30%

Warlock 2-9         10%      10%    50%        30%
Warlock 10+         20%      20%    30%        30%

Wizard 2-9          10%      10%    40%        40%
Wizard 10+          20%      20%    30%        30%
===============  ======  =======  =====  =========

Secondary Skill Advancement
~~~~~~~~~~~~~~~~~~~~~~~~~~~
Each secondary skill has a different chance to be learned by each hero type.
The table below lists the secondary skills and the hero types with an
indication of how often a skill will be learned. The higher the number, the
higher the chance that the skill will be available to learn; the lower the
number, the lower the chance that the skill will the available (a 0 mens no
chance).

===========  =========  ======  ===========  =========  =======  ======
Skill        Barbarian  Knight  Necromancer  Sorceress  Warlock  Wizard
===========  =========  ======  ===========  =========  =======  ======
Archery          **3**     2            1        **3**      1       1
Ballistics         3     **4**          3          3        3       3
Diplomacy          2     **3**          2          2        2       2
Eagle Eye          1       1          **3**        2      **3**   **3**
Estates            2     **3**          2          2        2       2
Leadership         3     **5**          0          1        1       2
Logistics        **3**   **3**          2          2        2       2
Luck               2       1            1        **3**      1       2
Mysticism          1       1            3          3        3     **4**
Navigation         3       2            2        **4**      2       2
Necromancy         0       0          **5**        0        1       0 
Pathfinding      **4**     3            3          2        2       2
Scouting         **4**     2            1          1      **4**     2
Wisdom             4       2            4          4      **5**   **5**
===========  =========  ======  ===========  =========  =======  ======

Bold items mark the skill that is most likely to increase of the particular
class.

Creatures
---------
Creatures with no *shots* have no ranged attacks.

Barbarian
~~~~~~~~~
Barbarian creatures are generally slow, but have high *Attack* skills and hit
points. While excellent in the early and midgame, the Barbarian creatures begin
to falter towards the endgame. The early game advantage is accented in small
and medium maps.

Goblin
^^^^^^
A solid low level creature, pack of Goblins are a match for most level two
creatures.

:Attack:      3
:Defense:     1
:Damage:      1-2
:Cost:        40 gold
:Hit Points:  3
:Speed:       Average

Orc
^^^^
Though slow, Orcs provide range attacks until Trolls can be recruited.

:Attack:      3
:Defense:     4
:Damage:      2-3
:Cost:        140 gold
:Hit Points:  10
:Speed:       Very Slow
:Shots:       8

Orc Chieftan
^^^^^^^^^^^^
The upgrade to the Orcs gives them longer durability in combat.

:Attack:      3
:Defense:     4
:Damage:      **3-4**
:Cost:        175 gold
:Hit Points:  **15**
:Speed:       **Slow**
:Shots:       **16**

Wolf
^^^^
Wolves are incredible offensive unints, but they need to be used carefully
because they cannot take damage well.

:Attack:      6
:Defense:     2
:Damage:      3-5
:Cost:        200 gold
:Hit Points:  20
:Speed:       Very Fast
:Special:     2 Attacks

Ogre
^^^^
Ogres are the anchor of the Barbarian units. Though tough, Ogres are very slow
on the battlefield, making it difficult tor them to attack.

:Attack:      9
:Defense:     5
:Damage:      4-6
:Cost:        300 gold
:Hit Points:  40
:Speed:       Very Slow

Ogre Lord
^^^^^^^^^
The upgrade to the Ogre ont only adds speed, but a sizable increase in hit
points.

:Attack:      9
:Defense:     5
:Damage:      **5-7**
:Cost:        500 gold
:Hit Points:  **60**
:Speed:       **Average**

Troll
^^^^^
The ability to regenerate and strike at range make trolls incredibly useful in
castle sieges.

:Attack:      10
:Defense:      5
:Damage:      5-7
:Cost:        600 gold
:Hit Points:  40
:Speed:       Average
:Shots:       8
:Special:     Regenerates

War Troll
^^^^^^^^^
The Troll upgrade increases in damage and speed, while keeping the ability to
regenerate.

:Attack:      10
:Defense:      5
:Damage:      **7-9**
:Cost:        700 gold
:Hit Points:  40
:Speed:       **Fast**
:Shots:       **16**
:Special:     Regenerates

Cyclops
^^^^^^^
Cyclopes are poweful ground combatants.

:Attack:      12
:Defense:      9
:Damage:      12-24
:Cost:        750 gold,
                1 crystal
:Hit Points:  80
:Speed:       Fast
:Description: Attack affects 2 spaces,
              20% chance to paralyze creatures
              
Knight
~~~~~~
Knight creatures have high defense skills, and at the upper levels are fairly
fast. Like the Barbarian, however, the Knight creatures are best in the early
and midgame. In the endgame, the best success will be fighting against the
Sorceress or Necromancer. Small and medium maps allow the Knight to press his
early game advantage.

Peasant
^^^^^^^
The weakest creature, their only redeeming quality is numbers- they are cheap
and plentiful.

:Attack:      1
:Defense:     1
:Damage:      1-1
:Cost:        20 gold
:Hit Points:  1
:Speed:       Very Slow

Archer
^^^^^^
The only range strike unit for the Knight, the slow speed of the Archers can be
a setback.

:Attack:      5
:Defense:     3
:Damage:      2-3
:Cost:        150 gold
:Hit Points:  10
:Speed:       Very Slow
:Shots:       12

Ranger
^^^^^^
The best low level upgrade available, the Ranger is almost twice as good as an
Archer on offense.

:Attack:      5
:Defense:     3
:Damage:      2-3
:Cost:        200 gold
:Hit Points:  10
:Speed:       **Average**
:Shots:       **24**
:Special:     Fires 2 shots per turn

Pikeman
^^^^^^^
Pikemen compose half of the standard units of the Knight. Though weak on
offense, the Pikemen's *Defense* allows them to last in battle.

:Attack:      5
:Defense:     9
:Damage:      3-4
:Cost:        200 gold
:Hit Points:  15
:Speed:       Average

Veteran Pikeman
^^^^^^^^^^^^^^^
The upgrade of the Pikemen gives increased speed and hit points.

:Attack:      5
:Defense:     9
:Damage:      3-4
:Cost:        250 gold
:Hit Points:  **20**
:Speed:       **Fast**

Swordsman
^^^^^^^^^
The other half of the standard Knight units, Swordsmen are tougher than the
Pikemen, and do considerably more damage.

:Attack:      7
:Defense:     9
:Damage:      4-6
:Cost:        250 gold
:Hit Points:  25
:Speed:       Average

Master Swordsman
^^^^^^^^^^^^^^^^
The Swordsman upgrade gives increased speed and hit points.

:Attack:      7
:Defense:     9
:Damage:      4-6
:Cost:        300 gold
:Hit Points:  **30**
:Speed:       **Fast**

Cavalry
^^^^^^^
Cavalry deal considerable damage, and their high speed allows them to manuever
easily around the battlefield.

:Attack:      10
:Defense:      9
:Damage:      5-10
:Cost:        300 gold
:Hit Points:  30
:Speed:       Very Fast

Champion
^^^^^^^^
One of the fastest units, the Cavalry upgrade can move around almost at will on
the battlefield.

:Attack:      10
:Defense:      9
:Damage:      5-10
:Cost:        375 gold
:Hit Points:  **40**
:Speed:       **Ultra Fast**

Paladin
^^^^^^^
Expert warriors, Paladins are best suited on offense, where their ability to
strike twice gives them the biggest advantage.

:Attack:      11
:Defense:     12
:Damage:      10-20
:Cost:        600 gold
:Hit Points:  50
:Speed:       Fast
:Special:     2 attacks

Crusader
^^^^^^^^
The Paladin upgrade becomes a nightmare for the unwary Necromancer.

:Attack:      11
:Defense:     12
:Damage:      10-20
:Cost:        1000 gold
:Hit Points:  **65**
:Speed:       **Very Fast**
:Description: 2 attacks,
              Immune to curse,
              x2 damage vs. undead

Necromancer
~~~~~~~~~~~
The Necromancer creatures, while weak at the low levels become much more
powerful at the high levels. The Necromancer is weak in the early game, but
strong in the mid and endgame. Larger maps give the Necomancer time to develop
the higher level creatures.

All Necromancer units (and ghosts) are undead, and therefore immune to mind
affecting spells, Bless and Curse, and are always at neutral morale.

Skeleton
^^^^^^^^
The best level one creature, Skeletons should be hoarded by Necormancers, as
they provide easily available power early on.

:Attack:      4
:Defense:     3
:Damage:      2-3
:Cost:        75 gold
:Hit Points:  4
:Speed:       Average

Zombie
^^^^^^
Though havig more hits than Skeletons, Zombies have a low Defense and speed.

:Attack:      5
:Defense:     2
:Damage:      2-3
:Cost:        150 gold
:Hit Points:  15
:Speed:       Very Slow

Mutant Zombie
^^^^^^^^^^^^^
The Zombiew upgrade increases the speed of the Zombies. Mutant Zombies are
worthwhile additions to any fledgling undead army.

:Attack:      5
:Defense:     2
:Damage:      2-3
:Cost:        200 gold
:Hit Points:  **20**
:Speed:       **Average**

Mummy
^^^^^
The best ground creature available for the Necromancer.

:Attack:      6
:Defense:     6
:Damage:      3-4
:Cost:        250 gold
:Hit Points:  25
:Speed:       Average
:Special:     20% chance to curse enemy creatures

Royal Mummy
^^^^^^^^^^^
The upgrade of the Mummy has improved speed and toughness.

:Attack:      6
:Defense:     6
:Damage:      3-4
:Cost:        300 gold
:Hit Points:  **30**
:Speed:       **Fast**
:Special:     30% chance to curse enemy creatures

Vampire
^^^^^^^
Vampires are necessary for the success of the Necromancer.

:Attack:      8
:Defense:     6
:Damage:      5-7
:Cost:        500 gold
:Hit Points:  30
:Speed:       Average
:Special:     Flies,
              Creatures attacked by Vampires cannot retaliate

Vampire Lord
^^^^^^^^^^^^

:Attack:      8
:Defense:     6
:Damage:      5-7
:Cost:        500 gold
:Hit Points:  **40**
:Speed:       **Fast**
:Special:     Flies,
              Creatures attacked by Vampire Lords cannot retaliate,
              Vampire Lords gain back some of the damage they do as hit points

Lich
^^^^
Liches are the only range strike unit available to the Necromancer.

:Attack:      7
:Defense:     12
:Damage:      8-10
:Cost:        900 gold
:Hit Points:  25
:Speed:       Fast
:Shots:       12
:Special:     Range attack affects adfacent hexes

Power Lich
^^^^^^^^^^
This upgrade of the Lich improves the durability of the Lich in combat.

:Attack:      7
:Defense:     **13**
:Damage:      8-10
:Cost:        900 gold
:Hit Points:  **25**
:Speed:       **Very Fast**
:Shots:       **24**
:Special:     Range attack affects adfacent hexes

Bone Dragon
^^^^^^^^^^^
Bone Dragons are fierce creatures, second only to the Warlock Dragons in raw
damage.

:Attack:      11
:Defense:      9
:Damage:      25-45
:Cost:        1500 gold
:Hit Points:  150
:Speed:       Average
:Special:     Flies,
              Lowers the morale of opposing creatures

Sorceress
~~~~~~~~~
Sorceress creatures are generally very fast, but have low hit points. Weak in
the early game, and moderate at the best in the endgame, the Sorceress
creatures are the best in midgame. The combination of speed, flying, and range
attack becomes incredibly potent. Medium maps are perfect for Sorceress
creatures.

Sprite
^^^^^^
Sprites are powerful in large numbers.

:Attack:      4
:Defense:     2
:Damage:      1-2
:Cost:        50 gold
:Hit Points:  2
:Speed:       Average
:Special:     Flies,
              Creatures attacked by Sprites cannot retaliate

Dwarf
^^^^^
Dwarves make excellent garrison units bcause of their toughness and magic
resistance.

:Attack:      6
:Defense:     5
:Damage:      2-4
:Cost:        200 gold
:Hit Points:  20
:Speed:       Very Slow
:Special:     25% Magic resistance

Battle Dwarf
^^^^^^^^^^^^
The upgrade of the Dwarf is faster and tougher.

:Attack:      6
:Defense:     **6**
:Damage:      2-4
:Cost:        250 gold
:Hit Points:  20
:Speed:       **Average**
:Special:     25% Magic resistance

Elf
^^^
Elves are incedible offensive units, able to deal large amounts of damage at
range.

:Attack:      4
:Defense:     3
:Damage:      2-3
:Cost:        250 gold
:Hit Points:  15
:Speed:       Average
:Shots:       24
:Special:     Fires 2 shots per turn

Grand Elf
^^^^^^^^^
The Elf upgrade becomes faster and more skilled. Grand Elves are able to
whittle down enemy foces quickly.

:Attack:      **5**
:Defense:     **5**
:Damage:      2-3
:Cost:        300 gold
:Hit Points:  15
:Speed:       **Very Fast**
:Shots:       24
:Special:     Fires 2 shots per turn

Druid
^^^^^
Druids are one of the best range strike units available. Though weak, few units
can close to melee range on the Druid without dying.

:Attack:      7
:Defense:     5
:Damage:      5-8
:Cost:        350 gold
:Hit Points:  25
:Speed:       Fast
:Shots:       8

Grater Druid
^^^^^^^^^^^^
The Druid upgrade is faster and tougher. Greater Druids and Grand Elves
complement each other well.

:Attack:      7
:Defense:     **7**
:Damage:      5-8
:Cost:        400 gold
:Hit Points:  25
:Speed:       **Very Fast**
:Shots:       **16**

Unicorn
^^^^^^^
Unicorns are solid ground creatures. They are tough, fast, and deal good
damage.

:Attack:      10
:Defense:      9
:Damage:      7-14
:Cost:        500 gold
:Hit Points:  40
:Speed:       Fast
:Special:     20% chance to Blind enemy creature.

Phoenix
^^^^^^^
Extremely fast and powerful, Phoenixes can be formidable oppeonents.

:Attack:      10
:Defense:      9
:Damage:      7-14
:Cost:        500 gold
:Hit Points:  40
:Speed:       Fast
:Special:     Flies,
              Attack affects two hexes,
              Immune to elemental spells.


Warlock
~~~~~~~
The Warlock units are slow and expensive, but have high hit points and good
*Attack* and *Defense* skills. Poor in the midgame, Warlock creatures are
effective in the early game, and show their true colors in the endgame, where
Dragons rule the battlefield. Warlocks can have success on small maps, but
generally so better on larger maps where they have time do develop Dragons.

Centaur
^^^^^^^
Centaurs are the only range strike creature available to the Warlock, and are
valuable for that reason.

:Attack:      3
:Defense:     1
:Damage:      1-2
:Cost:        60 gold
:Hit Points:  5
:Speed:       Average

Gargoyle
^^^^^^^^
Due to their speed and toughness, Gargoyles are one of the most useful Warlock
creatures.

:Attack:      4
:Defense:     7
:Damage:      2-3
:Cost:        200 gold
:Hit Points:  15
:Speed:       Very Fast
:Special:     Flies

Griffin
^^^^^^^
Griffins are able to fight large numbers of creatures and prove victorious.

:Attack:      6
:Defense:     6
:Damage:      3-5
:Cost:        300 gold
:Hit Points:  25
:Speed:       Average
:Special:     Flies

Minotaur
^^^^^^^^
Minotaurs are good offensive creatures, but are slow compared to the earlier
Warlock creatures.

:Attack:      9
:Defense:     8
:Damage:      5-10
:Cost:        400 gold
:Hit Points:  35
:Speed:       Average

Minotaur King
^^^^^^^^^^^^^
The Minotaur upgrade saves the Warlock in the midgame because of the increased
speed and toughness.

:Attack:      9
:Defense:     8
:Damage:      5-10
:Cost:        400 gold
:Hit Points:  **35**
:Speed:       **Very Fast**

Hydra
^^^^^
Though powerful, their slow speed makes the Hydra most useful as a garrison
creature.

:Attack:      8
:Defense:     9
:Damage:      6-12
:Cost:        800 gold
:Hit Points:  75
:Speed:       Very Slow
:Special:     Attacks all adjacen enemies.

Green Dragon
^^^^^^^^^^^^
The Dragon easily reigns as one of the best sixth level creature, and can fight
small armies itself.

:Attack:      12
:Defense:     12
:Damage:      25-50
:Cost:        3000 gold,
                 1 sulfur
:Hit Points:  200
:Speed:       Average
:Special:     Flies,
              Attack affects two hexes,
              Immune to spells

Red Dragon
^^^^^^^^^^
The first upgrade to the Dragon improves in speed, thoughness and skill.

:Attack:      **13**
:Defense:     **13**
:Damage:      25-50
:Cost:        3500 gold,
                 1 sulfur
:Hit Points:  **250**
:Speed:       **Fast**
:Special:     Flies,
              Attack affects two hexes,
              Immune to spells

Black Dragon
^^^^^^^^^^^^
The second upgrade to the Dragon improves again in speed, thoughness and skill.

:Attack:      **14**
:Defense:     **14**
:Damage:      25-50
:Cost:        3500 gold,
                 2 sulfur
:Hit Points:  **300**
:Speed:       **Very Fast**
:Special:     Flies,
              Attack affects two hexes,
              Immune to spells

Wizard
~~~~~~
Wizard creatures have a little of everything, some toughness, some speed, some
range strike ability. Like the Necromancer, the Wizard is weak in the early
game, strong in the midgame, and challenges the Warlock for endgame power.
Titans and Archmages are the best range strike creatures around, and Titans
match up to Dragons in power.

Halfling
^^^^^^^^
Halflings provide solid, early range strike ability for the Wizard.

:Attack:      2
:Defense:     1
:Damage:      1-3
:Cost:        50 gold
:Hit Points:  3
:Speed:       Slow
:Shots:       12

Boar
^^^^
Boars are fast and strong, and make excellent units for exploring.

:Attack:      5
:Defense:     4
:Damage:      2-3
:Cost:        150 gold
:Hit Points:  15
:Speed:       Very Fast

Iron Golem
^^^^^^^^^^
The high defense, parial magic resistance, and slow speeed make Golems exellent
garrison creatures.

:Attack:      5
:Defense:     10
:Damage:      4-5
:Cost:        300 gold
:Hit Points:  30
:Speed:       Very Slow
:Special:     1/2 damage from elemental spells

Steel Golem
^^^^^^^^^^^
The Golem upgrade is faster, tougher, and stronger.

:Attack:      **7**
:Defense:     10
:Damage:      4-5
:Cost:        350 gold
:Hit Points:  **35**
:Speed:       **Slow**
:Special:     1/2 damage from elemental spells

Roc
^^^
The only flying creature available to the Wizard, the Roc offers solid offense
and defense.

:Attack:      7
:Defense:     7
:Damage:      4-8
:Cost:        400 gold
:Hit Points:  40
:Speed:       Average
:Special:     Flies

Mage
^^^^
Though weak, Mages provide incredible offensive power.

:Attack:      11
:Defense:      7
:Damage:      7-9
:Cost:        600 gold
:Hit Points:  30
:Speed:       Fast
:Shots:       12
:Special:     No penalty for attacking adjacent units.

Archmage
^^^^^^^^
Archmages are second only to Titans in range strike ability.

:Attack:      **12**
:Defense:      **8**
:Damage:      7-9
:Cost:        700 gold
:Hit Points:  **35**
:Speed:       **Very Fast**
:Shots:       24
:Special:     No penalty for attacking adjacent units,
              20% chance to dispel beneficial spells on their target

Giant
^^^^^
Giants do good damage and have enormous hit points, making them the scariest
creature on the ground.

:Attack:      13
:Defense:     10
:Damage:      20-30
:Cost:        1250 gold,
                 1 gems
:Hit Points:  150
:Speed:       Average
:Special:     Immune to mind affecting spells

Titan
^^^^^
Titans are capable of defeating Dragons in one on one combat.

:Attack:      **15**
:Defense:     **15**
:Damage:      20-30
:Cost:        5000 gold,
                 2 gems
:Hit Points:  **300**
:Speed:       **Very Fast**
:Shots:       **16**
:Special:     Immune to mind affecting spells

Neutral
~~~~~~~
There creatures do not belong under any hero type, and range from ghosts to
rogues to elementals. Any of these creatures can end up in a hero's army
(except ghosts), and using them can sometimes make the difference between
victory and defeat. Recruiting these creatures becomes a necessity on higher
game difficulties, where you need to fill your armies with whatever you can
find, but on the lower difficulties they are more of a bonus.

Rogue
^^^^^
Rogues are useful early in the game, providing extra offense to any hero's
army.

:Attack:      6
:Defense:     1
:Damage:      1-2
:Cost:        50 gold
:Hit Points:  4
:Speed:       Fast
:Special:     Creatures attacked by Rogues cannot rataliate


Nomad
^^^^^
Nomads provide inexpensive fast creatures that can deal and take damage
reasonably well.

:Attack:      7
:Defense:     6
:Damage:      2-5
:Cost:        200 gold
:Hit Points:  20
:Speed:       Very Fast

Ghost
^^^^^
Ghosts are fearsome opponents. Never attack ghosts with level one creatures!

:Attack:      8
:Defense:     7
:Damage:      4-6
:Hit Points:  20
:Speed:       Fast
:Special:     Flies,
              Undead,
              Creatures killed by ghosts become ghosts

Genie
^^^^^
Between Paladin and Phoenix in power, the Genies low cost and awesome special
ability are always useful.

:Attack:      10
:Defense:      9
:Damage:      20-30
:Cost:        650 gold,
                1 gems
:Hit Points:  50
:Speed:       Very Fast
:Special:     Flies,
              10% halve enemy unit

Medusa
^^^^^^
Medusas make a welcome addition to any army or garrison force.

:Attack:      8
:Defense:     9
:Damage:      6-10
:Cost:        500 gold,
:Hit Points:  35
:Speed:       Average
:Special:     20% chance to turn victim to stone for the combat

Air Elemental
^^^^^^^^^^^^^

:Attack:      7
:Defense:     7
:Damage:      2-8
:Hit Points:  35
:Speed:       Very Fast
:Special:     Neutral morale,
              Immune to mind spells and Meteor Swarm,
              Storm and Lightning Bolt do x2 damage

Earth Elemental
^^^^^^^^^^^^^^^

:Attack:      8
:Defense:     8
:Damage:      4-5
:Hit Points:  50
:Speed:       Slow
:Special:     Neutral morale;
              Immune to mind spells, Lightning Bolt and Storm;
              Meteor Swarm does x2 damage

Fire Elemental
^^^^^^^^^^^^^^

:Attack:      8
:Defense:     6
:Damage:      4-6
:Hit Points:  40
:Speed:       Fast
:Special:     Neutral morale,
              Immune to mind- and fire spells,
              Fire spells do x2 damage

Water Elemental
^^^^^^^^^^^^^^^

:Attack:      6
:Defense:     8
:Damage:      3-7
:Hit Points:  45
:Speed:       Average
:Special:     Neutral morale,
              Immune to mind- and cold spells,
              Fire spells do x2 damage

Structures
==========
All the structures available are listed below, arranged by town.

Common
------
The following structures are available in all towns.

Mage Guild
~~~~~~~~~~
Allows spellbook purchase and teaches spells. Additional levels become
increasingly more expensive.

:Cost:  2000 gold,
           5 wood,
           5 ore

Tavern
~~~~~~
Gives defenders a bonus to morale and offers rumors. Not available in
Necromancer towns.

:Cost:  500 gold,
          5 wood

Thieves' Guild
~~~~~~~~~~~~~~
Gives information comparing the players. Additional Guilds give more
information.

:Cost:  750 gold,
          5 wood

Shipyard
~~~~~~~~
Allows construction of ships.

:Cost:  2000 gold,
          20 wood
:Unit:  1000 gold,
          10 wood

Statue
~~~~~~
Increases income of town by 250 gold.

:Cost:  1250 gold
           5 ore

Marketplace
~~~~~~~~~~~
Allows trading of resources. Additions Marketplaces give a better exchange
rate.

:Cost:  500 gold,
          5 wood

Well
~~~~
Increases creature production of each dwelling by two per week.

:Cost:  500 gold

Horde Building
~~~~~~~~~~~~~~
Increases creature production of the lowest dwelling by eight per week.

:Cost:  1000 gold

Left Turret
~~~~~~~~~~~
Adds a smaller ballista in the castle walls.

:Cost:  1500 gold,
           5 ore

Right Turret
~~~~~~~~~~~~
Adds a smaller ballista in the castle walls.

:Cost:  1500 gold,
           5 ore

Moat
~~~~
Entering moat stops ground movement, and units have -3 Defense while in the
moat.

:Cost:  750 gold

Barbarian
---------

Knight
------

Necromancer
-----------

Sorceress
---------

Warlock
-------

Wizard
------

New Structure (PoL)
-------------------
The Necomancer castle has a new building in the Price of Loyalty expansion.
Where the other castles have a tavern, the Necromancer castle now has an Evil
Shrine.

Evil Shrine
~~~~~~~~~~~
Increases the number of skeletons resurrected after a battle by 10%, to a
maximum of 60%.

:Cost:  4000 gold,
          10 wood,
          10 crystal

========
Appendix
========

References
==========
The following sources were used for reference and to guide me in the right
direction:

`Free Heroes 2 Engine
<http://sourceforge.net/projects/fheroes2/>`_
