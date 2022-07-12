DayZ Player Database Protocol

Player data is saved in Players.db. The database consists of 3 tables. Table Players
is where all the player data is stored. The table consists of 4 fields, Id, Alive, UID, and Data.
The Data field is a blob containing a hex string. The hex string is translated as below:

02 00 B9 B0 6C 46 5D C5 6B 41 4E 15 54 46 07 1B 				// 16 bytes of location data

| unknown | PosX        | unknown     | PosY        | unknown |
|---------|-------------|-------------|-------------|---------|
| 02 00   | B9 B0 6C 46 | 5D C5 6B 41 | 4E 15 54 46 | 07 1B   |


0E 53 75 72 76 69 76 6F 72 46 5F 4A 75 64 79 					// 14 bytes SurvivorF_Judy, Likely the player model

00 01 00 00 5A 00 00 00 00 00 05 00                             // 12 bytes likely containing model customization

### From here, player data is stored in a length prefix stream format followed by a 4 byte value.
`04 64 69 73 74 A1 CC 0A 44`

| length | data        | ASCII | 4 byte value |
|--------|-------------|-------|--------------|
| 04     | 64 69 73 74 | dist  | A1 CC 0A 44  |

#### The above contains total distance traveled for the character. I've ascertained that the 4 byte value is a little endian float value.


`0E 70 6C 61 79 65 72 73 5F 6B 69 6C 6C 65 64 00 00 00 00`

| length | data                                      | ASCII           | 4 byte value |
|--------|-------------------------------------------|-----------------|--------------|
| 0E     | 70 6C 61 79 65 72 73 5F 6B 69 6C 6C 65 64 | players_killed  | 00 00 00 00  |



`0F 69 6E 66 65 63 74 65 64 5F 6B 69 6C 6C 65 64 00 00 00 00` 


| length | data                                          | ASCII           | 4 byte value |
|--------|-----------------------------------------------|-----------------|--------------|
| 0F     | 69 6E 66 65 63 74 65 64 5F 6B 69 6C 6C 65 64  | infected_killed | 00 00 00 00  |


`08 70 6C 61 79 74 69 6D 65 A9 36 2F 44`


| length | data                                    | ASCII    | 4 byte value |
|--------|-----------------------------------------|----------|--------------|
| 08     | 08 70 6C 61 79 74 69 6D 65 A9 36 2F 44  | playtime | A9 36 2F 44  |


`14 6C 6F 6E 67 65 73 74 5F 73 75 72 76 69 76 6F 72 5F 68 69 74`


| length | data                                                                    | ASCII                | 4 byte value |
|--------|-------------------------------------------------------------------------|----------------------|--------------|
| 14     | 6C 6F 6E 67 65 73 74 5F 73 75 72 76 69 76 6F 72 5F 68 69 74 00 00 00 00 | longest_survivor_hit | A9 36 2F 44  |



### This is the beginning of a 170 byte blob of unknown data. I assume this is where the character's HP, Hydration, Bleeding, Sickness, etc. are stored.
`81 00 00 00 3B 84 6F E7 E2 1E 41 17 94 8D 0E
F2 EA 4E 14 81 58 00 00 00 C9 FF FF C9 FF FF C2
C9 FF FF C2 C9 FF FF C7 C9 FF FF C9 FF FF C7 C9
FF FF C9 FF FF C7 C9 FF FF C9 FF FF C7 C9 FF FF
C9 FF FF C7 C9 FF FF C9 FF FF C7 C9 FF FF C9 FF
FF C7 C9 FF FF C9 FF FF C7 C9 FF FF C9 FF FF C7
C9 FF FF C9 FF FF C7 C9 FF FF C9 FF FF C7 C9 FF
FF 2E 00 00 00 00 00 00 00 00 C8 64 54 C7 00 CA
32 54 A8 43 CA 2F 3D 00 44 CA 00 40 1C 45 CA A4
C0 C1 42 C7 08 C7 C7 00 00 80 00 D2 04 53 5F 50
00 00 00 04 00 00 00 D9 01 00 00`

Of the 170 byte block, these places represent the follow.
ie. UnkBlock[33] is used to store blood Blood.

I am still unsure what the developers deal is here. It seems like for some values they use a byte, then uint16, and then unint32 on others. It doesn't make a whole lot of sense.

	Blood [33]
	Health [106, 107]
	Hunger [127, 128, 129, 130]
	Hydration [132, 133, 134, 135]
	Bleeding [153]


## Items begin after the 170 byte block:

#### Items conform to the same protocol as above. The item is prefixed by item name length with follow on item value data.


T S h i r t _ B l a c k

`0C 54 53 68 69  72 74 5F 42 6C 61 63 6B 	
00 00 FF FF FF FF 04 42 6F 64 79 23 00 00 00 20
DF A1 81 28 65 43 3E AC EF 22 E7 5A BD 03 20 03
00 00 00 C9 23 77 08 00 00 00 01 FF 01 08 C9 8B
0C 00 05 00 00 00 49 00 00 00`


B a n d a g e D r e s s i n g

`0F 42 61 6E 64 61 67 65 44 72 65 73 73 69 6E 67 
00 00 00 00 00 00 05 63 61 72 67 6F 25 00
00 00 76 FF 3C C1 54 78 45 CA 9D 9D 6A 7F 88 8F
DB 5C 00 00 00 00 0D 00 00 00 01 FF 01 09 CA 00
00 00 40 C9 8B 0C 00 00 00 00 00 59 00 00 00`


C h e m l i g h t _ Y e l l o w

`10 43 68 65 6D 6C 69 67 68 74 5F 59 65 6C 6C 6F 77 
00 00 00 00 01 00 05 63 61 72 67 6F 34 00 00 00
42 4C 34 EE DE 8C 43 A0 97 13 57 19 30 04 02 59
03 00 00 00 C9 AE 8E 19 00 00 00 CA 00 C0 28 46
00 00 00 00 00 00 00 01 FF 01 09 CA 00 00 C8 42
C9 8B 0C 00 00 00 00 00 46 00 00 00`

## I've been able to ascertain the below:
```
04 50 65 61 72
00 00 00 00 02 
00 05 63 61 72 
67 6F 2D 00 00
00 C8 28 42 EA 
FC 83 4D D9 A1 
19 47 F3 CC 44 
36 33 03 00 00 
00 C9 E3 89 12 
00 00 00 01 FF 
01 08 C9 8B 0C 
00 01 00 00 00 
CA BA A1 2D 47 
01 00 00 00 00 
3D 00 00 00
```

| length | data        | ASCII |
|--------|-------------|-------|
| 04     | 50 65 61 72 | Pear  |


### After the item name comes the follow-on data. I've broken this up in to five byte rows but only for readability for what I currently know.
### The first five bytes seem to relate to the container they are in. IE a shirt. The first two bytes are unknown. The third byte determines which row (starting with 0) the item belongs to in the container. The fourth byte is unknown. The fifth byte determines which column of the container the item belongs in.


| Unk | Unk | Row | Unk   | Col |
|-----|-----|-----|-------|-----|
| 00  | 00  | 00  | 00    | 02  | 


P e a r

`04 50 65 61 72 
00 00 00 00 02 00 05 63 61 72 67 6F 2D 00 00
00 C8 28 42 EA FC 83 4D D9 A1 19 47 F3 CC 44 36
33 03 00 00 00 C9 E3 89 12 00 00 00 01 FF 01 08
C9 8B 0C 00 01 00 00 00 CA BA A1 2D 47 01 00 00 00 00 3D 00 00 00`

L o c k p i c k

`08 4C 6F 63 6B 70 69 63 6B 			
00 00 00 00 03 00 05 63 61 72 67 6F 20 00 00 00 C7
3B 0A 04 E4 7D 47 A9 BC 6E 8A CA E2 FE 9C 28 00
00 00 00 08 00 00 00 01 FF 01 08 C9 8B 0C 00 00
00 00 00 5D 00 00 00`

M a g _ I J 7 0 _ 8 R n d

`0D 4D 61 67 5F 49 4A 37 30 5F 38 52 6E 64				

00 01 02 00 00 00 05 63 61 72 67
6F 3B 00 00 00 7E BE 41 F2 F4 6D 4C A2 A9 C8 E3
B4 F5 D3 E1 BB 00 00 00 00 04 00 00 00 01 FF 00
00 85 D2 4A 07 8D D5 C7 D2 4A 07 8D D5 C7 D2 4A
07 8D D5 C7 D2 4A 07 8D D5 C7 D2 4A 07 8D D5 C7
00 00 00 00 A4 00 00 00`

CanvasPantsMidi_Red

`13 43 61 6E 76 61 73 50 61 6E 74 73 4D 69 64 69 5F 52 65 64 

00 00 FF FF
FF FF 04 4C 65 67 73 23 00 00 00 CD 50 F4 46 2C
43 49 8D 80 E3 DE 16 31 9D 67 19 03 00 00 00 C9
BF A4 08 00 00 00 01 FF 01 08 C9 8B 0C 00 01 00
00 00 56 00 00 00`


LargeGasCanister


`10 4C 61 72 67 65 47 61 73 43 61 6E 69 73 74 65 72 					

00 00 00 00 00 00 05 63 61
72 67 6F 31 00 00 00 DF 4B 83 15 CF B6 46 47 9D
1B FA E9 D7 7B 6A 25 00 00 00 00 19 00 00 00 CA
00 00 48 44 01 01 00 00 00 00 00 01 FF 01 09 CA
00 00 A0 41 C9 8B 0C 00 00 00 00 00 48 00 00 00`


HikingBoots_Black

`11 48 69 6B 69 6E 67 42 6F 6F 74 73 5F 42 6C 61 63 6B 	

00 00 FF FF FF FF 04 46 65 65 74 23 00 00
00 6F A9 30 5F 26 A8 40 7D 82 B8 12 69 A5 98 BB
26 03 00 00 00 C9 74 FE 08 00 00 00 01 FF 01 08
C9 8B 0C 00 00 00 00 00 14 01 00 00`

 DryBag_Blue

`0B 44 72 79 42 61 67 5F 42 6C 75 65		

00 00 FF FF FF FF 04 42
61 63 6B 23 00 00 00 25 36 B7 C9 2D 92 4A F7 BA
93 F3 AA 2F E4 52 77 03 00 00 00 C9 32 B3 08 00
00 00 01 FF 01 08 C9 8B 0C 00 03 00 00 00 44 00
00 00`

BakedBeansCan

`0D 42 61 6B 65 64 42 65 61 6E 73 43 61 6E		

00 00 00 00 00 00 05 63 61 72 67 6F 22 00 00 00
B8 43 2C EE 07 2D 47 1D BA 28 5F 76 D6 21 7D 30
00 00 00 00 0A 00 00 00 01 FF 01 08 C9 8B 0C 00
C7 00 00 00 00 00 3C 00 00 00`

Crowbar

`07 43 72 6F 77 62 61 72			

61 72 00 00 00 00 02 00 05 63 61 72 67 6F 20 00
00 00 28 D5 25 90 F0 7B 43 9A AB 2B B7 3D D2 E0
6A DF 00 00 00 00 08 00 00 00 01 FF 01 08 C9 8B
0C 00 00 00 00 00 46 00 00 00`

AmmoBox_380_35rnd

`11 41 6D 6D 6F 42 6F 78 5F 33 38 30 5F 33 35 72 6E 64 	

00 00 00 00 03 00 05 63 61 72 67 6F 20 00 00 00
5A C2 42 EA 32 BF 49 4D 84 89 B9 31 3E E1 80 9C
00 00 00 00 08 00 00 00 01 FF 01 08 C9 8B 0C 00
00 00 00 00`
