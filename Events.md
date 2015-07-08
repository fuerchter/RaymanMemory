# Event documentation
(All addresses and offsets are in hexadecimal)

## Rayman
Rayman himself is stored (at **1f61a0**) in a similar way to the events, so the data from the Main Events list can be applied to him as well.  
Rayman's hitbox can be read in the following way:

|address|bytesize|description|
|---|---|---|
|1f9a10|2|x|
|1f9a28|2|y|
|1f9a08||width|
|1f84c8||height|

**Offset 40** of Rayman's properties is used in the assembly to calculate the tile on the map, Rayman is currently standing on (tileAddress being what is stored at **1f4438**): tileAddress+sll(off40, 1)

## Hitbox intersections
At _13fd7c_ in the assembly code some hitbox intersections are tested. _r4-r7_ usually seem to contain Rayman here (or his fist) with _r4/r5_ being the position, _r6/r7_ being the size.

## Animation 1 (and it's hitbox)
There is a list of single frames of an animation starting at **4316c** (An example would be the tings in Anguish lagoon first screen, which start at **43680**). It uses the address at **offset 0** of each event to get the right element from the list.

length: 20 bytes

|offset|bytesize|description|
|---|---|---|
|05||width (of sprite)|
|06||height|
|07||hitbox width (not used in a script yet)|
|08||hitbox height|
|09||used in the assembly code to calculate the position of a hitbox (not 100% sure of it's meaning though)|
|0c|2|extra coloring?|
|0e||choose tileset|
|0f||reverse colors?|
|10||x-offset (within tileset)|
|11||y-offset|

## Animation 2 (and it's hitbox)
Similar to the previous one, each frame of an animation is represented with the address at **offset 4** of an event (this is combined with **offset 54** in the following way (in the assembly): off4+sll(sll(off54, 1)+off54, 2))  
This only represents the base of the animation though. So at the address that results from that calculation you can find 3*4 bytes. The first 4 bytes point to the following:  
length: 4 bytes

|offset|bytesize|description|
|---|---|---|
|0||flipping direction|
|1||x-offset (moves the sprite at the current frame)|
|2||y-offset|
|3||sprite index|

The next 4 bytes point to a hitbox (and you can add sll(off55, 2) of an event to it, to get the current frame, this is done at *140804* in the assembly):  
length: 4 bytes

|offset|bytesize|description|
|---|---|---|
|0||x-offset|
|1||y-offset|
|2||width|
|3||height|

If an event is flipped horizontally, the x-position of the hitbox will change. Instead of adding the x-offset of the hitbox to the event's position, you will need to do the following calculation (similar to *147374*): xPosition+sll(off52, 1)-xOffset-width


## Other properties
Some properties for certain event types are (assumed) to be where **offset c** of an event is pointing to (tings tend to be at **43db8** for example).

length: 8 bytes

|offset|bytesize|description|
|---|---|---|
|2||animation index (ALSO AFFECTS HITBOX!) (overwrites **54** in event)|
|4||overwrites **58** in event|
|6||plays sound effects|

## Hitboxes
**1c1a94** contains a list of hitboxes. These are not all hitboxes yet, afaik, but I'm trying to find more (example: **1c1b34** is the Anguish lagoon first screen goal sign). 1c1a94+sll(off48, 3) is how it seems to be calculated in the assembly (some events even have 00 at **offset 48**).

length: 8 bytes

|offset|bytesize|description|
|---|---|---|
|0|2*2|offset of the hitbox (depends on which way the event is facing?)|
|4||width|
|5||height|
|6||can disable hitbox depending on its value|

## Active events
**1e5428** is a list of all events that are active on the current frame (one byte with the index of the event, after that one byte with 00).

## Main events list
**1d7ae0** holds a pointer to the list of events. **1d7ae4** is the number of events on the current screen.

length: 112 bytes

|offset|bytesize|description|
|---|---|---|
|00|4|might point to animation|
|04|4|might point to animation (points to addresses, which all point to after **00/0c**?)|
|08|4|magic value|
|0c|4|might point to animation (uses **56, 58, 63** in its calculations at _~150ecc_ in the assembly)|
|10|4|?|
|14|4|magic value (not always present here?)|
|18|4||
|1c|2*2|position|
||||
|20|1|index|
||1||
|22|2*2|camera position relative to the event's position (signed?)|
|26|2||
||||
|28|2*2|position when it respawns (based on **22** and/or whenever it's inactive)|
|2c|2*2|speed|
||||
||4*4||
||||
||2||
|42|2|invincibility (on Rayman at least)|
||||
||2||
|46|2|transformation (related to scrolling somehow)|
||||
|48|2|first byte: hitbox|
||1||
||1||
||||
||4||
||||
||2||
|52|2|offset (x and y one byte each, mainly used when the event flips horizontally?)|
||||
|54|1|animation|
|55|1|animation counter|
|56|1|animation (can change events AND HITBOX)|
||1||
||||
|58|1|animation index? (change to 13 to change a ting to a one up e.g., including it's hitbox) tings don't necessarily have the same (18, 19, 1a, 1b, 1c, 1d)|
||1||
||2||
||||
||1||
|5d|1|becomes 1 on rayman/ting collision|
|5e|1|y-offset/hitline for platforms (not sure how it works)|
||1||
||||
|60|1|hp|
||1||
||1||
|63|1|lower this value to get an explosion (or change a flower to other events like a lily or butterfly e.g.), can display boss healthbar, can make events fly down/up<br>can switch between effects/behaviors like doing damage (93), giving a p<br>ting: 8e for one up<br>one up: 2a for road sign, 0c for hunter?<br>cage: 8e for one up (without collecting it though)<br><br>_14403c_ in code checks type?<br>compared with 32, e3 and a5 (both are hurting?)|
||||
||1||
|65|1|changes depending on whether rayman is close (doesn't seem to sync up with activity though)|
||2||
||||
||2||
|6a|1|event only visible in 1-7 range|
||1||
||||
|||loads all 4 bytes in assembly for this at once:|
||1||
|6d|1|flipping a sprite (horizontally), if anding it with 0x40 results in 0 it is facing left, otherwise it's facing right|
||2||
