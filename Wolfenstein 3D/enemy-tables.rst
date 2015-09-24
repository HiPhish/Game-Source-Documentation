.. default-role:: math

#################
Tables for actors
#################


List of actor classes and starting hit points
#############################################

The following is a list of all types of actors defined in Wolfenstein 3D and
their description, as well as their starting hit points for every dificulty.
They are ordered by their appearance in the game.

=======  =================================  ====  ====  ======  ====
Name     Description                        Baby  Easy  Normal  Hard
=======  =================================  ====  ====  ======  ====
Guard    Brown guards                         25    25      25    25
Officer  White soldiers                       50    50      50    50
SS       Blue soldiers                       100   100     100   100
Dog      Shepherd dogs                         1     1       1     1
Hans     Hans Grosse                         850   950    1050  1200
Schabbs  Dr. Schabbs                         850   950    1550  2400
Fake     Fake Hitler                         200   300     400   500
Mecha    Mecha Hitler                        800   950    1050  1200
Hitler   Real Hitler                         500   700     800   900
Mutant   Mutant soldier                       45    55      55    65
Blinky   Red Pac-Man ghost                    25    25      25    25
Clyde    Orange Pac-Man ghost                 25    25      25    25
Pinky    Pink Pac-Man ghost                   25    25      25    25
Inky     Cyan Pac-Man ghost                   25    25      25    25
Gretel   Gretel Grosse                       850   950    1050  1200
Gift     Otto Gifmacher                      850   950    1050  1200
Fat      General Fettgesicht                 850   950    1050  1200

Needle   Syringe thrown by Schabbs             0     0       0     0
Fire     Fireball from Fake Hitler             0     0       0     0
Rocket   Some bosses have rocket launchers     0     0       0     0
Smoke    Smoke from rockets                    0     0       0     0

BJ       BJ doing the victory jump           100   100     100   100
=======  =================================  ====  ====  ======  ====

Spear of destiny adds the following actors as well:

=======  ================  ====  ====  ======  ====
Name     Description       Baby  Easy  Normal  Hard
=======  ================  ====  ====  ======  ====
Spark    ???                  0     0       0     0
hrocket  ???                  0     0       0     0
hsmoke   ???                  0     0       0     0

Spectre  Spectre enemy       10    10      15    25
Angel    Angle of Death    1550  1550    1650  2000
Trans    Trans Grosse       950   950    1050  1200
Uber     Ubermutant        1150  1150    1250  1400
Will     Barnacle Wilhelm  1050  1050    1150  1300
Death    Death Knight      1350  1350    1450  1600
=======  ================  ====  ====  ======  ====

The mission packs introduce the following enemies:

=====  =====================  ====  ====  ======  ====
Name   Description            Baby  Easy  Normal  Hard
=====  =====================  ====  ====  ======  ====
Bat    Bats, replace mutants    10    10      15    25
Willi  Sumbarine Willi        1550  1550    1650  2000
Quark  Dr. Quarkblitz          950   950    1050  1200
Axe    The Axe                1150  1150    1250  1400
Robot  The Robot              1050  1050    1150  1300
Devil  Devil Incarnate        1350  1350    1450  1600
=====  =====================  ====  ====  ======  ====

Due to the hardcoded nature of the Wolfenstein 3D engine all the enemies in the
mission packs are just re-skins of enemies from Spear of Destiny, so their
starting hit points are the same.



List of state classes
#####################

The following is a list of all types of actor states in the order they are
defined in the original source. Sticking to that order is not strictly
necessary, but one has to make sure to keep the mapping correct.

``path`` means patrolling around before having spotted the player, ``die`` is
the state of dying when the death animation is being played. ``dead`` is the
state of being acctually dead.

=======  =====
State    Index
=======  =====
stand        0

path1        1
path1s       2
path2        3
path3        4
path3s       5
path4        6

pain         7
pain1        8

shoot1       9
shoot2      10
shoot3      11
shoot4      12
shoot5      13
shoot6      14
shoot7      15
shoot8      16
shoot9      17

chase1      18
chase1s     19
chase2      20
chase3      21
chase3s     22
chase4      23

die1        24
die2        25
die3        26
die4        27
die5        28
die6        29
die7        30
die8        31
die9        32

dead        33

remove      34
=======  =====

:TODO: A list of all actor states for each actor would be too large for this
       document, so it will have its own file.


Table of actor states
#####################

The following tables list the states for every actor class and state class. If a
field is empty, then there is nothing defined for it. In the case of a function
pointer it means the pointer is ``NULL`` and in the case of a sprite the sprite
is just some useless junk sprite.


Guard
=====

===========  ==========  ==============  =======  =======  ============  ==========
State        Can Rotate  Base Sprite     Timeout  Thought  Action        Next State
===========  ==========  ==============  =======  =======  ============  ==========
**stand**    true        SPR_GRD_S_1           0  Stand                  stand

**path1**    true        SPR_GRD_W1_1         20  Path                   path1s
**path1s**   true        SPR_GRD_W1_1          5                         path2
**path2**    true        SPR_GRD_W2_1         15  Path                   path3
**path3**    true        SPR_GRD_W3_1         20  Path                   path3s
**path3s**   true        SPR_GRD_W3_1          5                         path4
**path4**    true        SPR_GRD_W4_1         15  Path                   path1

**pain**     false       SPR_GRD_PAIN_1       10                         chase1
**pain1**    false       SPR_GRD_PAIN_2       10                         chase1

**shoot1**   false       SPR_GRD_SHOOT1       20                         shoot2
**shoot2**   false       SPR_GRD_SHOOT2       20           Shoot         shoot3
**shoot3**   false       SPR_GRD_SHOOT3       20                         chase1
**shoot4**   false                             0                         chase1
**shoot5**   false                             0                         chase1
**shoot6**   false                             0                         chase1
**shoot7**   false                             0                         chase1
**shoot8**   false                             0                         chase1
**shoot9**   false                             0                         chase1

**chase1**   true        SPR_GRD_W1_1         10  Chase                  chase1s
**chase1s**  true        SPR_GRD_W1_1          3                         chase2
**chase2**   true        SPR_GRD_W2_1          8  Chase                  chase3
**chase3**   true        SPR_GRD_W3_1         10  Chase                  chase3s
**chase3s**  true        SPR_GRD_W3_1          3                         chase4
**chase4**   true        SPR_GRD_W4_1          8  Chase                  chase1

**die1**     false       SPR_GRD_DIE_1        15           Death Scream  die2
**die2**     false       SPR_GRD_DIE_2        15                         die3
**die3**     false       SPR_GRD_DIE_3        15                         dead
**die4**     false                             0                         dead
**die5**     false                             0                         dead
**die6**     false                             0                         dead
**die7**     false                             0                         dead
**die8**     false                             0                         dead
**die9**     false                             0                         dead

**dead**     false       SPR_GRD_DEAD          0                         dead
===========  ==========  ==============  =======  =======  ============  ==========


Officer
=======

===========  ==========  ==============  =======  =======  ============  ========
State        Can Rotate  Base Sprite     Timeout  Thought  Action        Next State
===========  ==========  ==============  =======  =======  ============  ========
**stand**    true        SPR_OFC_S_1           0  Stand                  stand

**path1**    true        SPR_OFC_W1_1         20  Path                   path1s
**path1s**   true        SPR_OFC_W1_1          5                         path2
**path2**    true        SPR_OFC_W2_1         15  Path                   path3
**path3**    true        SPR_OFC_W3_1         20  Path                   path3s
**path3s**   true        SPR_OFC_W3_1          5                         path4
**path4**    true        SPR_OFC_W4_1         15  Path                   path1

**pain**     false       SPR_GRD_PAIN_1       10                         chase1
**pain1**    false       SPR_GRD_PAIN_2       10                         chase1

**shoot1**   false       SPR_GRD_SHOOT1        6                         shoot2
**shoot2**   false       SPR_GRD_SHOOT2       20           Shoot         shoot3
**shoot3**   false       SPR_GRD_SHOOT3       10                         chase1
**shoot4**   false                             0                         chase1
**shoot5**   false                             0                         chase1
**shoot6**   false                             0                         chase1
**shoot7**   false                             0                         chase1
**shoot8**   false                             0                         chase1
**shoot9**   false                             0                         chase1

**chase1**   true        SPR_OFC_W1_1         10  Chase                  chase1s
**chase1s**  true        SPR_OFC_W1_1          3                         chase2
**chase2**   true        SPR_OFC_W2_1          8  Chase                  chase3
**chase3**   true        SPR_OFC_W3_1         10  Chase                  chase3s
**chase3s**  true        SPR_OFC_W3_1          3                         chase4
**chase4**   true        SPR_OFC_W4_1          8  Chase                  chase1

**die1**     false       SPR_OFC_DIE_1        11           Death Scream  die2
**die2**     false       SPR_OFC_DIE_2        11                         die3
**die3**     false       SPR_OFC_DIE_3        11                         dead
**die4**     false                             0                         dead
**die5**     false                             0                         dead
**die6**     false                             0                         dead
**die7**     false                             0                         dead
**die8**     false                             0                         dead
**die9**     false                             0                         dead

**dead**     false       SPR_OFC_DEAD          0                         dead
===========  ==========  ==============  =======  =======  ============  ========


SS
====

===========  ==========  =============  =======  =======  ============  ==========
State        Can Rotate  Base Sprite    Timeout  Thought  Action        Next State
===========  ==========  =============  =======  =======  ============  ==========
**stand**    true        SPR_SS_S_1           0  Stand                  stand

**path1**    true        SPR_SS_W1_1         20  Path                   path1s
**path1s**   true        SPR_SS_W1_1          5                         path2
**path2**    true        SPR_SS_W2_1         15  Path                   path3
**path3**    true        SPR_SS_W3_1         20  Path                   path3s
**path3s**   true        SPR_SS_W3_1          5                         path4
**path4**    true        SPR_SS_W4_1         15  Path                   path1

**pain**     false       SPR_SS_PAIN_1       10                         chase1
**pain1**    false       SPR_SS_PAIN_2       10                         chase1

**shoot1**   false       SPR_SS_SHOOT1       20                         shoot2
**shoot2**   false       SPR_SS_SHOOT2       20           Shoot         shoot3
**shoot3**   false       SPR_SS_SHOOT3       10                         chase1
**shoot4**   false       SPR_SS_SHOOT2       10           Shoot         chase1
**shoot5**   false       SPR_SS_SHOOT3       10                         chase1
**shoot6**   false       SPR_SS_SHOOT2       10           Shoot         chase1
**shoot7**   false       SPR_SS_SHOOT3       10                         chase1
**shoot8**   false       SPR_SS_SHOOT2       10           Shoot         chase1
**shoot9**   false       SPR_SS_SHOOT3       10                         chase1

**chase1**   true        SPR_SS_W1_1         10  Chase                  chase1s
**chase1s**  true        SPR_SS_W1_1          3                         chase2
**chase2**   true        SPR_SS_W2_1          8  Chase                  chase3
**chase3**   true        SPR_SS_W3_1         10  Chase                  chase3s
**chase3s**  true        SPR_SS_W3_1          3                         chase4
**chase4**   true        SPR_SS_W4_1          8  Chase                  chase1

**die1**     false       SPR_SS_DIE_1        15           Death Scream  die2
**die2**     false       SPR_SS_DIE_2        15                         die3
**die3**     false       SPR_SS_DIE_3        15                         dead
**die4**     false                            0                         dead
**die5**     false                            0                         dead
**die6**     false                            0                         dead
**die7**     false                            0                         dead
**die8**     false                            0                         dead
**die9**     false                            0                         dead

**dead**     false       SPR_SS_DEAD          0                         dead
===========  ==========  =============  =======  =======  ============  ==========


Dog
====

Dogs have no pain because they have only one hit point.

===========  ==========  =============  =======  =======  ============  ==========
State        Can Rotate  Base Sprite    Timeout  Thought  Action        Next State
===========  ==========  =============  =======  =======  ============  ==========
**stand**    false                            0                         stand

**path1**    true        SPR_DOG_W1_1        20  Path                   path1s
**path1s**   true        SPR_DOG_W1_1         5                         path2
**path2**    true        SPR_DOG_W2_1        15  Path                   path3
**path3**    true        SPR_DOG_W3_1        20  Path                   path3s
**path3s**   true        SPR_DOG_W3_1         5                         path4
**path4**    true        SPR_DOG_W4_1        15  Path                   path1

**pain**     false                           10                         chase1
**pain1**    false                           10                         chase1

**shoot1**   false       SPR_DOG_JUMP1       20                         shoot2
**shoot2**   false       SPR_DOG_JUMP2       20           Shoot         shoot3
**shoot3**   false       SPR_DOG_JUMP3       10                         chase1
**shoot4**   false       SPR_DOG_JUMP2       10           Shoot         chase1
**shoot5**   false       SPR_DOG_W1_1        10                         chase1
**shoot6**   false                           10           Shoot         chase1
**shoot7**   false                           10                         chase1
**shoot8**   false                           10           Shoot         chase1
**shoot9**   false                           10                         chase1

**chase1**   true        SPR_DOG_W1_1        10  Chase                  chase1s
**chase1s**  true        SPR_DOG_W1_1         3                         chase2
**chase2**   true        SPR_DOG_W2_1         8  Chase                  chase3
**chase3**   true        SPR_DOG_W3_1        10  Chase                  chase3s
**chase3s**  true        SPR_DOG_W3_1         3                         chase4
**chase4**   true        SPR_DOG_W4_1         8  Chase                  chase1

**die1**     false       SPR_DOG_DIE_1       15           Death Scream  die2
**die2**     false       SPR_DOG_DIE_2       15                         die3
**die3**     false       SPR_DOG_DIE_3       15                         dead
**die4**     false                            0                         dead
**die5**     false                            0                         dead
**die6**     false                            0                         dead
**die7**     false                            0                         dead
**die8**     false                            0                         dead
**die9**     false                            0                         dead

**dead**     false       SPR_DOG_DEAD         0                         dead
===========  ==========  =============  =======  =======  ============  ==========


Hans Grosse
===========

Hans has no pain and no patrol.

===========  ==========  ===============  =======  =======  ===============  ==========
State        Can Rotate  Base Sprite      Timeout  Thought  Action           Next State
===========  ==========  ===============  =======  =======  ===============  ==========
**stand**    true        SPR_BOSS_W1            0  Stand                     stand

**path1**    true                               0  Path                      path1s
**path1s**   true                               0                            path2
**path2**    true                               0  Path                      path3
**path3**    true                               0  Path                      path3s
**path3s**   true                               0                            path4
**path4**    true                               0  Path                      path1

**pain**     false                              0                            chase1
**pain1**    false                              0                            chase1

**shoot1**   false       SPR_BOSS_SHOOT1       30                            shoot2
**shoot2**   false       SPR_BOSS_SHOOT2       10           Shoot            shoot3
**shoot3**   false       SPR_BOSS_SHOOT3       10           Shoot            chase4
**shoot4**   false       SPR_BOSS_SHOOT2       10           Shoot            chase5
**shoot5**   false       SPR_BOSS_SHOOT3       10           Shoot            chase6
**shoot6**   false       SPR_BOSS_SHOOT2       10           Shoot            chase7
**shoot7**   false       SPR_BOSS_SHOOT3       10           Shoot            chase8
**shoot8**   false       SPR_BOSS_SHOOT1       10                            chase1
**shoot9**   false                              0                            chase1

**chase1**   true        SPR_BOSS_W1           10  Chase                     chase1s
**chase1s**  true        SPR_BOSS_W1            3                            chase2
**chase2**   true        SPR_BOSS_W2            8  Chase                     chase3
**chase3**   true        SPR_BOSS_W3           10  Chase                     chase3s
**chase3s**  true        SPR_BOSS_W3            3                            chase4
**chase4**   true        SPR_BOSS_W4            8  Chase                     chase1

**die1**     false       SPR_BOSS_DIE_1        15           Death Scream     die2
**die2**     false       SPR_BOSS_DIE_2        15                            die3
**die3**     false       SPR_BOSS_DIE_3        15                            dead
**die4**     false                              0                            dead
**die5**     false                              0                            dead
**die6**     false                              0                            dead
**die7**     false                              0                            dead
**die8**     false                              0                            dead
**die9**     false                              0                            dead

**dead**     false       SPR_BOSS_DEAD          0           Start Death Cam  dead
===========  ==========  ===============  =======  =======  ===============  ==========


Dr. Schabbs
===========

===========  ==========  ======================  =======  ==========  ===============  ==========
State        Can Rotate  Base Sprite             Timeout  Thought     Action           Next State
===========  ==========  ======================  =======  ==========  ===============  ==========
**stand**    false       SPR_SCHABB_W1                 0  Stand                        stand

**path1**    false                                     0                               path1s
**path1s**   false                                     0                               path2
**path2**    false                                     0                               path3
**path3**    false                                     0                               path3s
**path3s**   false                                     0                               path4
**path4**    false                                     0                               path1

**pain**     false                                     0                               chase1
**pain1**    false                                     0                               chase1

**shoot1**   false       SPR_SCHABB_SHOOT1            30                               shoot2
**shoot2**   false       SPR_SCHABB_SHOOT2            10              Launch           chase1
**shoot3**   false                                    10                               chase1
**shoot4**   false                                    10                               chase1
**shoot5**   false                                    10                               chase1
**shoot6**   false                                    10                               chase1
**shoot7**   false                                    10                               chase1
**shoot8**   false                                    10                               chase1
**shoot9**   false                                     0                               chase1

**chase1**   false       SPR_SCHABB_W1                10  Boss Chase                   chase1s
**chase1s**  false       SPR_SCHABB_W1                 3                               chase2
**chase2**   false       SPR_SCHABB_W2                 8  Boss Chase                   chase3
**chase3**   false       SPR_SCHABB_W3                10  Boss Chase                   chase3s
**chase3s**  false       SPR_SCHABB_W3                 3                               chase4
**chase4**   false       SPR_SCHABB_W4                 8  Boss Chase                   chase1

**die1**     false       SPR_SCHABB_SCHABB_W1         10              Death Scream     die2
**die2**     false       SPR_SCHABB_SCHABB_W1         10                               die3
**die3**     false       SPR_SCHABB_SCHABB_DIE1       10                               die4
**die4**     false       SPR_SCHABB_SCHABB_DIE2       10                               die5
**die5**     false       SPR_SCHABB_SCHABB_DIE3       10                               die6
**die6**     false                                     0                               dead
**die7**     false                                     0                               dead
**die8**     false                                     0                               dead
**die9**     false                                     0                               dead

**dead**     false       SPR_SCHABB_DEAD               0              Start Death Cam  dead
===========  ==========  ======================  =======  ==========  ===============  ==========


Fake Hitler
===========

===========  ==========  ==============  =======  ==========  ============  ==========
State        Can Rotate  Base Sprite     Timeout  Thought     Action        Next State
===========  ==========  ==============  =======  ==========  ============  ==========
**stand**    false       SPR_FAKE_W1           0  Stand                     stand

**path1**    false                             0                            path1s
**path1s**   false                             0                            path2
**path2**    false                             0                            path3
**path3**    false                             0                            path3s
**path3s**   false                             0                            path4
**path4**    false                             0                            path1

**pain**     false                             0                            chase1
**pain1**    false                             0                            chase1

**shoot1**   false       SPR_FAKE_SHOOT        8              Launch        shoot2
**shoot2**   false       SPR_FAKE_SHOOT        8              Launch        shoot3
**shoot3**   false       SPR_FAKE_SHOOT        8              Launch        shoot4
**shoot4**   false       SPR_FAKE_SHOOT        8              Launch        shoot5
**shoot5**   false       SPR_FAKE_SHOOT        8              Launch        shoot6
**shoot6**   false       SPR_FAKE_SHOOT        8              Launch        shoot7
**shoot7**   false       SPR_FAKE_SHOOT        8              Launch        shoot8
**shoot8**   false       SPR_FAKE_SHOOT        8              Launch        shoot9
**shoot9**   false       SPR_FAKE_SHOOT        8                            chase1

**chase1**   false       SPR_FAKE_W1          10  Fake                      chase1s
**chase1s**  false       SPR_FAKE_W1           3                            chase2
**chase2**   false       SPR_FAKE_W2           8  Fake                      chase3
**chase3**   false       SPR_FAKE_W3          10  Fake                      chase3s
**chase3s**  false       SPR_FAKE_W3           3                            chase4
**chase4**   false       SPR_FAKE_W4           8  Fake                      chase1

**die1**     false       SPR_FAKE_DIE1        10              Death Scream  die2
**die2**     false       SPR_FAKE_DIE2        10                            die3
**die3**     false       SPR_FAKE_DIE3        10                            die4
**die4**     false       SPR_FAKE_DIE4        10                            die5
**die5**     false       SPR_FAKE_DIE5        10                            dead
**die6**     false                             0                            dead
**die7**     false                             0                            dead
**die8**     false                             0                            dead
**die9**     false                             0                            dead

**dead**     false       SPR_FAKE_DEAD         0                            dead
===========  ==========  ==============  =======  ==========  ============  ==========


Mecha Hitler
============

===========  ==========  ================  =======  ==========  ============  ==========
State        Can Rotate  Base Sprite       Timeout  Thought     Action        Next State
===========  ==========  ================  =======  ==========  ============  ==========
**stand**    false       SPR_MECHA_W1            0  Stand                     stand

**path1**    false                               0                            path1s
**path1s**   false                               0                            path2
**path2**    false                               0                            path3
**path3**    false                               0                            path3s
**path3s**   false                               0                            path4
**path4**    false                               0                            path1

**pain**     false                               0                            chase1
**pain1**    false                               0                            chase1

**shoot1**   false       SPR_MECHA_SHOOT1       30                            shoot2
**shoot2**   false       SPR_MECHA_SHOOT2       10              Shoot         shoot3
**shoot3**   false       SPR_MECHA_SHOOT3       10              Shoot         shoot4
**shoot4**   false       SPR_MECHA_SHOOT2       10              Shoot         shoot5
**shoot5**   false       SPR_MECHA_SHOOT3       10              Shoot         shoot6
**shoot6**   false       SPR_MECHA_SHOOT2       10              Shoot         shoot7
**shoot7**   false                               0                            shoot8
**shoot8**   false                               0                            shoot9
**shoot9**   false                               0                            chase1

**chase1**   false       SPR_MECHA_W1           10  Chase       Mecha Sound   chase1s
**chase1s**  false       SPR_MECHA_W1            6                            chase2
**chase2**   false       SPR_MECHA_W2            8  Chase                     chase3
**chase3**   false       SPR_MECHA_W3           10  Chase       Mecha Sound   chase3s
**chase3s**  false       SPR_MECHA_W3            6                            chase4
**chase4**   false       SPR_MECHA_W4            8  Chase                     chase1

**die1**     false       SPR_MECHA_DIE1         10              Death Scream  die2
**die2**     false       SPR_MECHA_DIE2         10                            die3
**die3**     false       SPR_MECHA_DIE3         10              Hitler Morph  dead
**die4**     false                               0                            dead
**die5**     false                               0                            dead
**die6**     false                               0                            dead
**die7**     false                               0                            dead
**die8**     false                               0                            dead
**die9**     false                               0                            dead

**dead**     false       SPR_MECHA_DEAD          0                            dead
===========  ==========  ================  =======  ==========  ============  ==========


Adolf Hitler
============

===========  ==========  =================  =======  ==========  ===============  ==========
State        Can Rotate  Base Sprite        Timeout  Thought     Action           Next State
===========  ==========  =================  =======  ==========  ===============  ==========
**stand**    false                                0  Stand                        stand

**path1**    false                                0                               path1s
**path1s**   false                                0                               path2
**path2**    false                                0                               path3
**path3**    false                                0                               path3s
**path3s**   false                                0                               path4
**path4**    false                                0                               path1

**pain**     false                                0                               chase1
**pain1**    false                                0                               chase1

**shoot1**   false       SPR_HITLER_SHOOT1       30                               shoot2
**shoot2**   false       SPR_HITLER_SHOOT2       10              Shoot            shoot3
**shoot3**   false       SPR_HITLER_SHOOT3       10              Shoot            shoot4
**shoot4**   false       SPR_HITLER_SHOOT2       10              Shoot            shoot5
**shoot5**   false       SPR_HITLER_SHOOT3       10              Shoot            shoot6
**shoot6**   false       SPR_HITLER_SHOOT2       10              Shoot            shoot7
**shoot7**   false                                0                               shoot8
**shoot8**   false                                0                               shoot9
**shoot9**   false                                0                               chase1

**chase1**   false       SPR_HITLER_W1            6  Chase                        chase1s
**chase1s**  false       SPR_HITLER_W1            4                               chase2
**chase2**   false       SPR_HITLER_W2            2  Chase                        chase3
**chase3**   false       SPR_HITLER_W3            6  Chase                        chase3s
**chase3s**  false       SPR_HITLER_W3            4                               chase4
**chase4**   false       SPR_HITLER_W4            2  Chase                        chase1

**die1**     false       SPR_HITLER_W1            1              Death Scream     die2
**die2**     false       SPR_HITLER_W1           10                               die3
**die3**     false       SPR_HITLER_DIE1         10              Hitler Morph     die4
**die4**     false       SPR_HITLER_DIE2         10                               die5
**die5**     false       SPR_HITLER_DIE3         10                               die6
**die6**     false       SPR_HITLER_DIE4         10                               die7
**die7**     false       SPR_HITLER_DIE5         10                               die8
**die8**     false       SPR_HITLER_DIE6         10                               die9
**die9**     false       SPR_HITLER_DIE7         10                               dead

**dead**     false       SPR_HITLER_DEAD          0              Start Death Cam  dead
===========  ==========  =================  =======  ==========  ===============  ==========


Mutant
======

===========  ==========  ==============  =======  ==========  ============  ==========
State        Can Rotate  Base Sprite     Timeout  Thought     Action        Next State
===========  ==========  ==============  =======  ==========  ============  ==========
**stand**    true        SPR_MUT_S_1           0  Stand                     stand

**path1**    true        SPR_MUT_W1_1         20  Path                      path1s
**path1s**   true        SPR_MUT_W1_1          5                            path2
**path2**    true        SPR_MUT_W2_1         15  Path                      path3
**path3**    true        SPR_MUT_W3_1         20  Path                      path3s
**path3s**   true        SPR_MUT_W3_1          5                            path4
**path4**    true        SPR_MUT_W4_1         15  Path                      path1

**pain**     false       SPR_MUT_PAIN_1       10                            chase1
**pain1**    false       SPR_MUT_PAIN_2       10                            chase1

**shoot1**   false       SPR_MUT_SHOOT1        6                            shoot2
**shoot2**   false       SPR_MUT_SHOOT2       20              Shoot         shoot3
**shoot3**   false       SPR_MUT_SHOOT3       10              Shoot         shoot4
**shoot4**   false       SPR_MUT_SHOOT4       20              Shoot         shoot5
**shoot5**   false                             0              Shoot         shoot6
**shoot6**   false                             0              Shoot         chase1
**shoot7**   false                             0                            chase1
**shoot8**   false                             0                            chase1
**shoot9**   false                             0                            chase1

**chase1**   true        SPR_MUT_W1_1         10  Chase                     chase1s
**chase1s**  true        SPR_MUT_W1_1          3                            chase2
**chase2**   true        SPR_MUT_W2_1          8  Chase                     chase3
**chase3**   true        SPR_MUT_W3_1         10  Chase                     chase3s
**chase3s**  true        SPR_MUT_W3_1          3                            chase4
**chase4**   true        SPR_MUT_W4_1          8  Chase                     chase1

**die1**     false       SPR_MUT_DIE_1         7              Death Scream  die2
**die2**     false       SPR_MUT_DIE_2         7                            die3
**die3**     false       SPR_MUT_DIE_3         7                            die4
**die4**     false       SPR_MUT_DIE_4         7                            dead
**die5**     false                             0                            dead
**die6**     false                             0                            dead
**die7**     false                             0                            dead
**die8**     false                             0                            dead
**die9**     false                             0                            dead

**dead**     false       SPR_MUT_DEAD          0                            dead
===========  ==========  ==============  =======  ==========  ============  ==========


Blinky
======

===========  ==========  =============  =======  =======  ======  ==========
State        Can Rotate  Base Sprite    Timeout  Thought  Action  Next State
===========  ==========  =============  =======  =======  ======  ==========
**stand**    false                            0                   stand

**path1**    false                            0                   path1s
**path1s**   false                            0                   path2
**path2**    false                            0                   path3
**path3**    false                            0                   path3s
**path3s**   false                            0                   path4
**path4**    false                            0                   path1

**pain**     false                            0                   chase1
**pain1**    false                            0                   chase1

**shoot1**   false                            0                   shoot2
**shoot2**   false                            0                   shoot3
**shoot3**   false                            0                   shoot4
**shoot4**   false                            0                   shoot5
**shoot5**   false                            0                   shoot6
**shoot6**   false                            0                   chase1
**shoot7**   false                            0                   shoot8
**shoot8**   false                            0                   shoot9
**shoot9**   false                            0                   chase1

**chase1**   false       SPR_BLINKY_W1       10  Ghosts           chase2
**chase1s**  false                            0                   chase2
**chase2**   false       SPR_BLINKY_W2       10  Ghosts           chase1
**chase3**   false                            0                   chase3s
**chase3s**  false                            0                   chase4
**chase4**   false                            0                   chase1

**die1**     false                           10                   die2
**die2**     false                           10                   die3
**die3**     false                           10                   dead
**die4**     false                            0                   dead
**die5**     false                            0                   dead
**die6**     false                            0                   dead
**die7**     false                            0                   dead
**die8**     false                            0                   dead
**die9**     false                            0                   dead

**dead**     false                            0                   dead
===========  ==========  =============  =======  =======  ======  ==========


Clyde
=====

===========  ==========  ============  =======  =======  ======  ==========
State        Can Rotate  Base Sprite   Timeout  Thought  Action  Next State
===========  ==========  ============  =======  =======  ======  ==========
**stand**    false                           0                   stand

**path1**    false                           0                   path1s
**path1s**   false                           0                   path2
**path2**    false                           0                   path3
**path3**    false                           0                   path3s
**path3s**   false                           0                   path4
**path4**    false                           0                   path1

**pain**     false                           0                   chase1
**pain1**    false                           0                   chase1

**shoot1**   false                           0                   shoot2
**shoot2**   false                           0                   shoot3
**shoot3**   false                           0                   shoot4
**shoot4**   false                           0                   shoot5
**shoot5**   false                           0                   shoot6
**shoot6**   false                           0                   chase1
**shoot7**   false                           0                   shoot8
**shoot8**   false                           0                   shoot9
**shoot9**   false                           0                   chase1

**chase1**   false       SPR_CLYDE_W1       10  Ghosts           chase2
**chase1s**  false                           0                   chase2
**chase2**   false       SPR_CLYDE_W2       10  Ghosts           chase1
**chase3**   false                           0                   chase3s
**chase3s**  false                           0                   chase4
**chase4**   false                           0                   chase1

**die1**     false                          10                   die2
**die2**     false                          10                   die3
**die3**     false                          10                   dead
**die4**     false                           0                   dead
**die5**     false                           0                   dead
**die6**     false                           0                   dead
**die7**     false                           0                   dead
**die8**     false                           0                   dead
**die9**     false                           0                   dead

**dead**     false                           0                   dead
===========  ==========  ============  =======  =======  ======  ==========


Pinky
=====

===========  ==========  ============  =======  =======  ======  ==========
State        Can Rotate  Base Sprite   Timeout  Thought  Action  Next State
===========  ==========  ============  =======  =======  ======  ==========
**stand**    false                           0                   stand

**path1**    false                           0                   path1s
**path1s**   false                           0                   path2
**path2**    false                           0                   path3
**path3**    false                           0                   path3s
**path3s**   false                           0                   path4
**path4**    false                           0                   path1

**pain**     false                           0                   chase1
**pain1**    false                           0                   chase1

**shoot1**   false                           0                   shoot2
**shoot2**   false                           0                   shoot3
**shoot3**   false                           0                   shoot4
**shoot4**   false                           0                   shoot5
**shoot5**   false                           0                   shoot6
**shoot6**   false                           0                   chase1
**shoot7**   false                           0                   shoot8
**shoot8**   false                           0                   shoot9
**shoot9**   false                           0                   chase1

**chase1**   false       SPR_PINKY_W1       10  Ghosts           chase2
**chase1s**  false                           0                   chase2
**chase2**   false       SPR_PINKY_W2       10  Ghosts           chase1
**chase3**   false                           0                   chase3s
**chase3s**  false                           0                   chase4
**chase4**   false                           0                   chase1

**die1**     false                          10                   die2
**die2**     false                          10                   die3
**die3**     false                          10                   dead
**die4**     false                           0                   dead
**die5**     false                           0                   dead
**die6**     false                           0                   dead
**die7**     false                           0                   dead
**die8**     false                           0                   dead
**die9**     false                           0                   dead

**dead**     false                           0                   dead
===========  ==========  ============  =======  =======  ======  ==========


Inky
====

===========  ==========  ===========  =======  =======  ======  ==========
State        Can Rotate  Base Sprite  Timeout  Thought  Action  Next State
===========  ==========  ===========  =======  =======  ======  ==========
**stand**    false                          0                   stand

**path1**    false                          0                   path1s
**path1s**   false                          0                   path2
**path2**    false                          0                   path3
**path3**    false                          0                   path3s
**path3s**   false                          0                   path4
**path4**    false                          0                   path1

**pain**     false                          0                   chase1
**pain1**    false                          0                   chase1

**shoot1**   false                          0                   shoot2
**shoot2**   false                          0                   shoot3
**shoot3**   false                          0                   shoot4
**shoot4**   false                          0                   shoot5
**shoot5**   false                          0                   shoot6
**shoot6**   false                          0                   chase1
**shoot7**   false                          0                   shoot8
**shoot8**   false                          0                   shoot9
**shoot9**   false                          0                   chase1

**chase1**   false       SPR_INKY_W1       10  Ghosts           chase2
**chase1s**  false                          0                   chase2
**chase2**   false       SPR_INKY_W2       10  Ghosts           chase1
**chase3**   false                          0                   chase3s
**chase3s**  false                          0                   chase4
**chase4**   false                          0                   chase1

**die1**     false                         10                   die2
**die2**     false                         10                   die3
**die3**     false                         10                   dead
**die4**     false                          0                   dead
**die5**     false                          0                   dead
**die6**     false                          0                   dead
**die7**     false                          0                   dead
**die8**     false                          0                   dead
**die9**     false                          0                   dead

**dead**     false                          0                   dead
===========  ==========  ===========  =======  =======  ======  ==========
