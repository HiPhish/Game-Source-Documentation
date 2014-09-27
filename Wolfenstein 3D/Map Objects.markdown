Map object bytes
================

The following is a list of bytes and their corresponding map objects. Some objects are walls and only used on the *Architecture* map, others are decoration objects and used on the *Objects* map. No two objects have the same byte even though they always appear on different maps. The list was adapted from the "MAPDATA.DEF" file of the *Nmap* editor by Nathan Lineback. See below for acknowledgements.

Codes for Wolfenstein 3D
------------------------

	0x0000    Nothing
	0x0001    Grey stone 1
	0x0002    Grey stone 2
	0x0003    Grey w/banner
	0x0004    Grey w/hitler
	0x0005    Cell
	0x0006    Grey w/eagle
	0x0007    Cell w/skeleton
	0x0008    Blue stone 1
	0x0009    Blue stone 2
	0x000a    Wood w/eagle
	0x000b    Wood w/hitler
	0x000c    Wood panelling
	0x000d    Entrance to level
	0x000e    Steel w/signs
	0x000f    Steel plate
	0x0010    Landscape
	0x0011    Red brick
	0x0012    Red w/wreath
	0x0013    Purple wall
	0x0014    Red w/shield
	0x0015    Elevator <>
	0x0016    Elevator 2 ^v
	0x0017    Wood w/symbol
	0x0018    *stone w/yellow stf
	0x0019    Purple w/blood
	0x001a    *stone w/yellow2
	0x001b    Grey stone 3
	0x001c    *Grey stone w/signs
	0x001D    *Tan design
	0x001E    *Tan W blood
	0x001F    *Tan W blood 2
	0x0020    *Tan W blood 3
	0x0021    *Wblock/Hitler
	0x0022    *Blueblock/Face
	0x0023    *White block
	0x0024    *Bblock W symbol
	0x0025    *Wblock W Bit
	0x0026    *R/W/Y/bricks
	0x0027    *Wbrick W crack
	0x0028    *Blue Block
	0x0029    *Blue stone/sign
	0x002A    *Tan pannel
	0x002B    *Wblock w map
	0x002C    *Tan stone
	0x002D    *Tan stone2
	0x002E    *Tan pannel 2
	0x002F    *Tan Pannel/flag
	0x0030    *Plasterboard
	0x0031    *Wblock w Pic
	0x005a    Door
	0x005b    Door
	0x005c    Locked door
	0x005d    Locked door
	0x005E    Locked door 2 Vertical
	0x005F    Locked door 2 Horizontal
	0x0060    Fixed door Vertical
	0x0061    Fixed door Horizontal
	0x0062    Fixed door Vertical
	0x0063    Fixed door Horizontal
	0x0064    Elevator entrance Vertical
	0x0065    Elevator entrance Horizontal
	0x006A    Stasis Area 6A
	0x006C    Area 6C
	0x006D    Area 6D
	0x006E    Area 6E
	0x006F    Area 6F
	0x0070    Area 70
	0x0071    Area 71
	0x0072    Area 72
	0x0073    Area 73
	0x0074    Area 74
	0x0075    Area 75
	0x0076    Area 76
	0x0077    Area 77
	0x0078    Area 78
	0x0079    Area 79
	0x007A    Area 7A
	0x007B    Area 7B
	0x007C    Area 7C
	0x007D    Area 7D
	0x007E    Area 7E
	0x007F    Area 7F
	0x0080    Area 80
	0x0081    Area 81
	0x0082    Area 82
	0x0083    Area 83
	0x0084    Area 84
	0x0085    Area 85
	0x0086    Area 86
	0x0087    Area 87
	0x0088    Area 88
	0x0089    Area 89
	0x008A    Area 8A
	0x008B    Area 8B
	0x008C    Area 8C
	0x008D    Area 8D
	0x008E    Area 8E
	0x008F    Area 8F
	0x0100    Nothing
	0x0113    Start, face N
	0x0114    Start, face E           
	0x0115    Start, face S          
	0x0116    Start, face W           
	0x0117    Water                      
	0x0118    Oil drum                   
	0x0119    Table & Chairs             
	0x011A    Floor lamp                 
	0x011B    Shandelere                 
	0x011C    Hanging skeleton           
	0x011D    Dog food                   
	0x011E    Column                     
	0x011F    Potted tree                
	0x0120    Skeleton                   
	0x0121    Washbasin                  
	0x0122    Potted bush                
	0x0123    Vase                       
	0x0124    Table                      
	0x0125    Overhead lamp              
	0x0126    Pot & Pan rack             
	0x0127    Armor                      
	0x0128    Cage, empty                
	0x0129    Cage, with skeleton
	0x012A    Bones
	0x012B    Key 1 (yellow)
	0x012C    Key 2 (grey)
	0x012D    Bed
	0x012E    Pot
	0x012F    Food
	0x0130    First aid kit
	0x0131    Ammunition
	0x0132    Machine gun
	0x0133    Huge gun
	0x0134    Cross
	0x0135    Cup
	0x0136    Jewels
	0x0137    Crown
	0x0138    Sphere / life
	0x0139    Bloody bones
	0x013A    Brown barrel
	0x013B    Well with water
	0x013C    Well empty
	0x013D    Blood
	0x013E    Flag
	0x013F    *Call Apogee
	0x0140    *Trash 1
	0x0141    *Trash 2
	0x0142    *Trash 3
	0x0143    *Pot/Pan Rack2
	0x0144    *Oven
	0x0145    *Spear Rack
	0x0146    *Vines
	0x015A    Move East
	0x015B    Move North-East
	0x015C    Move ^
	0x015D    Move <\
	0x015E    Move <
	0x015F    Move </
	0x0160    Move v
	0x0161    Move \>
	0x0162    Secret door
	0x0163    End of game trigger
	0x016C    Guard 1 East
	0x016D    Guard 1 ^
	0x016E    Guard 1 <
	0x016F    Guard 1 v
	0x0170    Guard 1 > moving
	0x0171    Guard 1 ^ moving
	0x0172    Guard 1 < moving
	0x0173    Guard 1 v moving
	0x0174    *W Guard 1 East
	0x0175    *W Guard 1 ^
	0x0176    *W Guard 1 <
	0x0177    *W Guard 1 v
	0x0178    *W Guard 1 > moving
	0x0179    *W Guard 1 ^ moving
	0x017A    *W Guard 1 < moving
	0x017B    *W Guard 1 v moving
	0x017C    Dead guard
	0x017E    Officer 1  East
	0x017F    Officer 1  ^
	0x0180    Officer 1  <
	0x0181    Officer 1  v
	0x0182    Officer 1  > moving
	0x0183    Officer 1  ^ moving
	0x0184    Officer 1  < moving
	0x0185    Officer 1  v moving
	0x018A    Dog 1 East
	0x018B    Dog 1 ^
	0x018C    Dog 1 <
	0x018D    Dog 1 v
	0x0190    Guard 2 East
	0x0191    Guard 2 ^
	0x0192    Guard 2 <
	0x0193    Guard 2 v
	0x0194    Guard 2 East moving
	0x0195    Guard 2 ^ moving
	0x0196    Guard 2 < moving
	0x0197    Guard 2 v moving
	0x0198    *W Guard 2 East
	0x0199    *W Guard 2 ^
	0x019A    *W Guard 2 <
	0x019B    *W Guard 2 v
	0x019C    *W Guard 2 East moving
	0x019D    *W Guard 2 ^ moving
	0x019E    *W Guard 2 < moving
	0x019F    *W Guard 2 v moving
	0x01A0    *Floating guy
	0x01A2    Officer 2 East
	0x01A3    Officer 2 ^
	0x01A4    Officer 2 <
	0x01A5    Officer 2 v
	0x01A6    Officer 2 > moving
	0x01A7    Officer 2 ^ moving
	0x01A8    Officer 2 < moving
	0x01A9    Officer 2 v moving
	0x01AE    Dog 2 East
	0x01AF    Dog 2 ^
	0x01B0    Dog 2 <
	0x01B1    Dog 2 v
	0x01B2    *Robo Hitler
	0x01B3    *General
	0x01B4    Guard 3 East
	0x01B5    Guard 3 ^
	0x01B6    Guard 3 <
	0x01B7    Guard 3 v
	0x01B8    Guard 3 East moving
	0x01B9    Guard 3 ^ moving
	0x01BA    Guard 3 < moving
	0x01BB    Guard 3 v moving
	0x01BC    *W Guard 3 East
	0x01BD    *W Guard 3 ^
	0x01BE    *W Guard 3 <
	0x01BF    *W Guard 3 v
	0x01C0    *W Guard 3 East moving
	0x01C1    *W Guard 3 ^ moving
	0x01C2    *W Guard 3 < moving
	0x01C3    *W Guard 3 v moving
	0x01C4    *Dr Schabbs
	0x01C5    *BIG Guardet!
	0x01C6    Officer 3 East
	0x01C7    Officer 3 ^
	0x01C8    Officer 3 <
	0x01C9    Officer 3 v
	0x01CA    Officer 3 East moving
	0x01CB    Officer 3 ^ moving
	0x01CC    Officer 3 < moving
	0x01CD    Officer 3 v moving
	0x01D2    Dog 3 >
	0x01D3    Dog 3 ^
	0x01D4    Dog 3 <
	0x01D5    Dog 3 v
	0x01D6    Hans Grösse (boss)
	0x01D7    *Big officer
	0x01D8    *Zombie 1 East
	0x01D9    *Zombie 1 ^
	0x01DA    *Zombie 1 <
	0x01DB    *Zombie 1 v
	0x01DC    *Zombie 1 East
	0x01DD    *Zombie 1 ^
	0x01DE    *Zombie 1 <
	0x01DF    *Zombie 1 v
	0x01E0    Blinky (red ghost)
	0x01e1    Clyde (orange ghost
	0x01E2    Pinky (pink ghost)
	0x01E3    Inky (blue ghost)
	0x01EA    *Zombie 2 East
	0x01EB    *Zombie 2 ^
	0x01EC    *Zombie 2 <
	0x01ED    *Zombie 2 v
	0x01EE    *Zombie 2 East
	0x01EF    *Zombie 2 ^
	0x01F0    *Zombie 2 <
	0x01F1    *Zombie 2 v
	0x01FC    *Zombie 3 East
	0x01FD    *Zombie 3 ^
	0x01FE    *Zombie 3 <
	0x01FF    *Zombie 3 v
	0x0200    *Zombie 3 East
	0x0201    *Zombie 3 ^
	0x0202    *Zombie 3 <
	0x0203    *Zombie 3 v

Codes for Spear of Destiny
--------------------------
Spear of destiny uses a slightly different table.

	0x0000    Nothing
	0x0001    Grey stone 1
	0x0002    Grey stone 2
	0x0003    Grey w/banner
	0x0004    Grey w/hitler
	0x0005    Cell
	0x0006    Grey w/eagle
	0x0007    Cell w/skeleton
	0x0008    Blue stone 1
	0x0009    Blue stone 2
	0x000a    Wood w/eagle
	0x000b    Wood w/hitler
	0x000c    Wood panelling
	0x000d    Entrance to level
	0x000e    Steel w/signs
	0x000f    Steel plate
	0x0010    Landscape
	0x0011    Red brick
	0x0012    Red w/wreath
	0x0013    Purple wall
	0x0014    Red w/shield
	0x0015    Elevator <>
	0x0016    Elevator 2 ^v
	0x0017    Wood w/symbol
	0x0018    stone w/yellow stf
	0x0019    Purple w/blood
	0x001a    stone w/yellow2
	0x001b    Grey stone 3
	0x001c    Grey stone w/signs
	0x001D    Tan design
	0x001E    Tan W blood
	0x001F    Tan W blood 2
	0x0020    Tan W blood 3
	0x0021    Wblock/Hitler
	0x0022    Blueblock/Face
	0x0023    White block
	0x0024    Bblock W symbol
	0x0025    Wblock W Bit
	0x0026    R/W/Y/bricks
	0x0027    Wbrick W crack
	0x0028    Blue Block
	0x0029    Blue stone/sign
	0x002A    Tan pannel
	0x002B    Wblock w map
	0x002C    Tan stone
	0x002D    Tan stone2
	0x002E    Tan pannel 2
	0x002F    Tan Pannel/flag
	0x0030    Plasterboard
	0x0031    Wblock w Pic
	0x0032    Rock wall 1
	0x0033    Rock wall 2
	0x0034    Rock /w banner
	0x0035    Rock /w reath
	0x0036    Grey marble 1
	0x0037    Grey marble 2
	0x0038    Hot wall
	0x0039    Grey flat marble
	0x003A    Stone fence 1
	0x003B    Stone fence 2
	0x003C    Elevator decoration
	0x003D    White wall
	0x003E    Brown Marble
	0x003F    Purple Brick
	0x005a    Door
	0x005b    Door
	0x005c    Locked door
	0x005d    Locked door
	0x005E    Locked door 2 |
	0x005F    Locked door 2 -
	0x0060    Fixed door |
	0x0061    Fixed door -
	0x0062    Fixed door |
	0x0063    Fixed door -
	0x0064    Elevator entrance |
	0x0065    Elevator entrance -
	0x006A    Stasis Area 6A
	0x006C    Area 6C
	0x006D    Area 6D
	0x006E    Area 6E
	0x006F    Area 6F
	0x0070    Area 70
	0x0071    Area 71
	0x0072    Area 72
	0x0073    Area 73
	0x0074    Area 74
	0x0075    Area 75
	0x0076    Area 76
	0x0077    Area 77
	0x0078    Area 78
	0x0079    Area 79
	0x007A    Area 7A
	0x007B    Area 7B
	0x007C    Area 7C
	0x007D    Area 7D
	0x007E    Area 7E
	0x007F    Area 7F
	0x0080    Area 80
	0x0081    Area 81
	0x0082    Area 82
	0x0083    Area 83
	0x0084    Area 84
	0x0085    Area 85
	0x0086    Area 86
	0x0087    Area 87
	0x0088    Area 88
	0x0089    Area 89
	0x008A    Area 8A
	0x008B    Area 8B
	0x008C    Area 8C
	0x008D    Area 8D
	0x008E    Area 8E
	0x008F    Area 8F
	0x0100    Nothing
	0x0113    Start, face north
	0x0114    Start, face east
	0x0115    Start, face south
	0x0116    Start, face west
	0x0117    Water
	0x0118    Oil drum
	0x0119    Table & Chairs
	0x011A    Floor lamp
	0x011B    Shandelere
	0x011C    Hanging skeleton
	0x011D    Dog food
	0x011E    Column
	0x011F    Potted tree
	0x0120    Skeleton
	0x0121    Skulls on stick
	0x0122    Potted bush
	0x0123    Vase
	0x0124    Table
	0x0125    Overhead lamp
	0x0126    Cage W guts
	0x0127    Armor
	0x0128    Cage, empty
	0x0129    Cage, with skeleton
	0x012A    Bones
	0x012B    Key 1 (yellow)
	0x012C    Key 2 (grey)
	0x012D    Cage w skulls
	0x012E    Pot
	0x012F    Food
	0x0130    First aid kit
	0x0131    Ammunition
	0x0132    Machine gun
	0x0133    Huge gun
	0x0134    Cross
	0x0135    Cup
	0x0136    Jewels
	0x0137    Crown
	0x0138    Sphere / life
	0x0139    Bloody bones
	0x013A    Brown barrel
	0x013B    Well with water
	0x013C    Well empty
	0x013D    Blood
	0x013E    Flag
	0x013F    Overhead lamp2
	0x0140    Trash 1
	0x0141    Trash 2
	0x0142    Trash 3
	0x0143    Animal skull stick
	0x0144    Pool of guts
	0x0145    Demon statue
	0x0146    Vines
	0x0147    Tan pillar
	0x0148    Ammo Box
	0x0149    Truck
	0x014A    Spear
	0x015A    Move >
	0x015B    Move />
	0x015C    Move ^
	0x015D    Move <\
	0x015E    Move <
	0x015F    Move </
	0x0160    Move v
	0x0161    Move \>
	0x0162    Secret door
	0x0163    End of game trigger
	0x016A    WEIRD THING
	0x016B    LAST SOD GUY
	0x016C    Guard 1 >
	0x016D    Guard 1 ^
	0x016E    Guard 1 <
	0x016F    Guard 1 v
	0x0170    Guard 1 > moving
	0x0171    Guard 1 ^ moving
	0x0172    Guard 1 < moving
	0x0173    Guard 1 v moving
	0x0174    W Guard 1 >
	0x0175    W Guard 1 ^
	0x0176    W Guard 1 <
	0x0177    W Guard 1 v
	0x0178    W Guard 1 > moving
	0x0179    W Guard 1 ^ moving
	0x017A    W Guard 1 < moving
	0x017B    W Guard 1 v moving
	0x017C    Dead guard
	0x017D    SOD GUY1
	0x017E    Officer 1  >
	0x017F    Officer 1  ^
	0x0180    Officer 1  <
	0x0181    Officer 1  v
	0x0182    Officer 1  > moving
	0x0183    Officer 1  ^ moving
	0x0184    Officer 1  < moving
	0x0185    Officer 1  v moving
	0x018A    Dog 1 >
	0x018B    Dog 1 ^
	0x018C    Dog 1 <
	0x018D    Dog 1 v
	0x018E    SOD GUY4
	0x018F    SOD GUY2
	0x0190    Guard 2 >
	0x0191    Guard 2 ^
	0x0192    Guard 2 <
	0x0193    Guard 2 v
	0x0194    Guard 2 > moving
	0x0195    Guard 2 ^ moving
	0x0196    Guard 2 < moving
	0x0197    Guard 2 v moving
	0x0198    W Guard 2 >
	0x0199    W Guard 2 ^
	0x019A    W Guard 2 <
	0x019B    W Guard 2 v
	0x019C    W Guard 2 > moving
	0x019D    W Guard 2 ^ moving
	0x019E    W Guard 2 < moving
	0x019F    W Guard 2 v moving
	0x01A0    Floating guy
	0x01A1    SOD GUY3
	0x01A2    Officer 2 >
	0x01A3    Officer 2 ^
	0x01A4    Officer 2 <
	0x01A5    Officer 2 v
	0x01A6    Officer 2 > moving
	0x01A7    Officer 2 ^ moving
	0x01A8    Officer 2 < moving
	0x01A9    Officer 2 v moving
	0x01AE    Dog 2 >
	0x01AF    Dog 2 ^
	0x01B0    Dog 2 <
	0x01B1    Dog 2 v
	0x01B2    Robo Hitler
	0x01B3    General
	0x01B4    Guard 3 >
	0x01B5    Guard 3 ^
	0x01B6    Guard 3 <
	0x01B7    Guard 3 v
	0x01B8    Guard 3 > moving
	0x01B9    Guard 3 ^ moving
	0x01BA    Guard 3 < moving
	0x01BB    Guard 3 v moving
	0x01BC    W Guard 3 >
	0x01BD    W Guard 3 ^
	0x01BE    W Guard 3 <
	0x01BF    W Guard 3 v
	0x01C0    W Guard 3 > moving
	0x01C1    W Guard 3 ^ moving
	0x01C2    W Guard 3 < moving
	0x01C3    W Guard 3 v moving
	0x01C4    Dr Schabbs
	0x01C5    BIG Guardet!
	0x01C6    Officer 3 >
	0x01C7    Officer 3 ^
	0x01C8    Officer 3 <
	0x01C9    Officer 3 v
	0x01CA    Officer 3 > moving
	0x01CB    Officer 3 ^ moving
	0x01CC    Officer 3 < moving
	0x01CD    Officer 3 v moving
	0x01D2    Dog 3 >
	0x01D3    Dog 3 ^
	0x01D4    Dog 3 <
	0x01D5    Dog 3 v
	0x01D6    BIG Guard!
	0x01D7    Big officer
	0x01D8    Zombie 1 >
	0x01D9    Zombie 1 ^
	0x01DA    Zombie 1 <
	0x01DB    Zombie 1 v
	0x01DC    Zombie 1 > moving
	0x01DD    Zombie 1 ^ moving
	0x01DE    Zombie 1 < moving
	0x01DF    Zombie 1 v moving
	0x01E0    Red Ghost
	0x01E1    Orange Ghost
	0x01E2    Light red Ghost
	0x01E3    Blue Ghost
	0x01EA    Zombie 2 >
	0x01EB    Zombie 2 ^
	0x01EC    Zombie 2 <
	0x01ED    Zombie 2 v
	0x01EE    Zombie 2 > moving
	0x01EF    Zombie 2 ^ moving
	0x01F0    Zombie 2 < moving
	0x01F1    Zombie 2 v moving
	0x01FC    Zombie 3 >
	0x01FD    Zombie 3 ^
	0x01FE    Zombie 3 <
	0x01FF    Zombie 3 v
	0x0200    Zombie 3 > moving
	0x0201    Zombie 3 ^ moving
	0x0202    Zombie 3 < moving
	0x0203    Zombie 3 v moving

Ackgnowledgements
-----------------
NMAP Wolf3d map editor by Nathan Lineback [http://toastytech.com/files/nmap.html]
