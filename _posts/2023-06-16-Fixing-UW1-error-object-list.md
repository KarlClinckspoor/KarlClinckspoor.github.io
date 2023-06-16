# Solving "Underworld Internal Error - Problems in object list" without resorting to nukes

## Introduction

It's possible that during one of your playthroughs of Ultima Underworld 1, you were faced with this problem. Reading online, you noticed it has quite a nasty reputation and can lead to softlocks, requiring you to revert to a previous save or potentially having to start over. Neither option is very appetizing, so you look for the possible origin of the bug and solutions. Many people tell you it can be triggered if there are too many objects in a level, which happens if you, for example, farm enemy spawns or hoard items in a level, and that the solution is to chuck everything in water or blow everything up with fireballs and cheats/exploits. It works, the bug disappears, and you continue to play happy, but a bit annoyed.

While it is true, there is a 1022 object limit in a level (254 npcs/monsters and 764 items); this bug can happen way before the item list is full. While nuking *can* also work to recover these files, it's still less than ideal. With this post, I'll attempt to provide instructions to fix this bug yourself using a hex editor.

![Message](/assets/content_posts/2023-06-16-Fixing-UW1-error-object-list/Message.png)

## Requirements

For this tutorial, we'll require:

* [a corrupted save](https://1drv.ms/f/s!As-yStiKjmVHmfMdsA7CR_IvfgrTZw?e=vDvdcI) that triggers the error when you go from Lvl1 to Lvl2 by walking forwards a bit. Copy the contents to the folder `SAVE1` in your ultima underworld 1 installation. Note that this guide requires access to the files of the game, so you'll need to extract `game.gog` with any appropriate tool (such as 7zip), then configure dosbox. ([Direct link to LEV.ARK](/assets/content_posts/2023-06-16-Fixing-UW1-error-object-list/))
* [A level editor made by krokots](https://www.gog.com/forum/ultima_series/ultima_underworld_editor_release/post1) that has a bunch of useful features, but can't fix the bug itself for us.
* [A tool to dump all the data in `LEV.ARK` called `uwdump.exe`](https://github.com/vividos/UnderworldAdventures/releases/tag/version-0.10.0-pina-colada). Download the zipped file, extract it somewhere, go into tools and copy `uwdump.exe` to the same directory as `uw.exe`
* A hex editor, such as [HxD](https://mh-nexus.de/en/hxd/), to edit the files.
* The Windows calculator in Programmer mode.
* Some patience

## A primer on `LEV.ARK`

The file `LEV.ARK` contains all important level data of your playthrough, like the level layout, item locations, map and automap data. It's quite complex ([see `uw-formats.txt` for more info](https://github.com/vividos/UnderworldAdventures/blob/main/uwadv/docs/uw-formats.txt)), but I'll walk you through briefly about the layout of this file.

It is divided into blocks, which can be subdivided into more blocks, and so on. First, there is a header 542 bytes long that contains the offsets to the other blocks. An offset is essentially how much along the file you have to traverse to reach, in this case, a block. After the header, there are 9 blocks that control the level layout and object data of each of the 9 levels, then loads more blocks with other stuff. We're interested in these first 9 blocks.

Conveniently, they have a fixed size of 31752 (0x7C08) bytes. They start with the tile map. There are 64 * 64 tiles in a level, and each tile occupies 4 bytes, giving this subblock a size of 16384. Then there's another subblock with mobile object data (npcs, etc) that is 256 items long, and each entry is 27 bytes, giving a size of 6912 bytes. Then there's the static object data (swords, candles, etc), 768 items long, each with 8 bytes, for a total of 6144 bytes. Then there's some item allocation info, an area with unknown info, and it finishes with two bytes: `uw`. Very cute.

With this info, you can calculate the position of the n-th static object or m-th tile of any level. For example, static object 400 of level 6 will give you:

    542 (header) + 5 * 31752 (5 levels before lvl6) + 16384 (skipping the tiles) + 6912 (skipping mobile object) + 400 * 8 (400th object, each is 8 bytes long) = 185798

Note that we considered the 400th static object. If you see the tools forward, they give you the nth *game object* (mobile + static), so you have to subtract 256 from those values to find the static object position.

In `LEV.ARK`, there are two places that refer to whether an item should exist or not. First, objects can be placed in a tile, they can be placed inside a container, an NPC inventory, or you can have some invisible objects, like traps, that can link to other objects or traps. Second, there's a number that tells the game how many objects there are in the level, and a list that controls which objects are vacant, which tells the game should store new objects when they're created (e.g. when you drop them). However, for some reason, it is possible for an object to *not* be referenced by anything and *not* be flagged as vacant, so it's in a limbo. This causes the error message because the game is confused. This is the type of bug we'll be fixing today.

## Finding the culprit

Open a terminal window in the folder with `uwdump.exe` and run the following command:

`.\uwdump dump '.\SAVE1\LEV.ARK' > dump1.txt`

Then open `dump1.txt` using a tool like `notepad++`. This file is composed of a few regions, and it can look very intimidating, so we'll go slowly. First, open a search menu and look for `linkref=0000`. This property means who is referencing a particular object, and 0000 means no-one is referencing it. Your first match will be in line 153

    0001: [207f bf10 0000 0000] id=007f link=0000 flags=0 invis=0 ench=0 is_quant=0 ref=00 linkref=0000 tile=ff/ff [xpos=5 ypos=7 heading=2 zpos=10] [quality=00 owner=00 sp_link =0000] name=an_adventurer

This is because object slot 1 is always reserved for player info, and isn't important to us. We're looking for situations where `linkref=0000` is found but not in item 0001. This is the situation I described above, where an item isn't referenced by anything but its slot wasn't marked as free.

Continue searching and you'll reach line 2223:

    01f0: [8012 2b80 0016 0200] id=0012 link=0000 flags=0 invis=0 ench=0 is_quant=1 ref=00 linkref=0000 tile=ff/ff [xpos=1 ypos=2 heading=3 zpos=00] [quality=16 owner=00 quantity=0008] name=an_arrow

Apparently, there's a bundle of 8 arrows that aren't referenced. If you scroll up a bit, you'll see in line 1692 (`dumping infos for level 1 (0x01)`) that this refers to "level 1", which is actually the dwarven level (lvl 2 in game — we're counting from 0).

`01f0` is hex for 496, and means static object 240. To fix this, we'll have to modify some tile to refer to this object.

## Figuring out where to put the object

We'll use the level editor to find a suitable spot to put the item. It has a few quirks though. To use it to edit the contents of the folder `SAVE1`, you have to copy over `LEV.ARK` from `SAVE1` into `DATA` (backup your existing file there). Then go to `File`, `UW File path`, select `LEV.ARK` from the `DATA` folder, then `File`->`Load`. Go to lvl 2 using the tool in the bottom-left corner, then click `Object mode` and change to `Tile mode`. Since we're right before the staircase that leads to the upper-left corner of the map, close to the slugs with the moonstone. Let's select the tile at the intersection as the place to put our item.

![Editor.png](/assets/content_posts/2023-06-16-Fixing-UW1-error-object-list/Guide.PNG)

In the bottom-right corner, you'll see that there's a part written `File : A43E`. Conveniently, this is the absolute offset in the `LEV.ARK` file where the data of this tile starts at, in hex. In decimal, this is equivalent to 42046. We can confirm this with some math. This is tile X=6, Y=38 (also from the editor). Since the tile grid is 64x64, we can get the number of this tile using `38 * 64 + 6 = 2438`. Using similar math as above, we confirm this offset with:

    542 (header) + 1 * 31752 (1 level) + 2438 * 4 (2438 tiles) = 42046

It's important that you select a tile that has no objects in it.

Out of curiosity, you can go to the level objects viewer and see that there's, in fact, no item 496 (`Objects`->`Level objects`).

## Altering the tile hex data

You can close the editor now. Open `LEV.ARK` from the `SAVE1` folder using HxD. I recommend you backup your save before we continue, just in case. Click on `Search`->`Go To`, and type the offset we had. If you copy the one from the editor, be sure to select the `hex` option, and set `offset relative to` to `begin`.

![hxd_goto](/assets/content_posts/2023-06-16-Fixing-UW1-error-object-list/hxd_goto.PNG)

Your cursor will get placed at that offset. If you look at the offset at the left, in blue, it's `0000A430` and above it's `OE`. Joining those you get `A43E`.

![hxd_goto_tile](/assets/content_posts/2023-06-16-Fixing-UW1-error-object-list/hxd_goto_tile.PNG)

The tile data is `C1 08 20 00`. Each one of this is a byte, so 4 bytes in total (32 bits), or two `short`s of 16 bits each. We'll need to modify this to point it to the object we want, number 496.

Fortunately, [`uw-formats.txt`](https://github.com/vividos/UnderworldAdventures/blob/fb4b846f0262073ef879d92fff754dfe9ccc6a5e/uwadv/docs/uw-formats.txt#L1552) describes how to do this. Here's the required except, from section 4.2.

> For each tile there are two Int16 that describe a
> tile's properties. The map's origin is at the lower left tile, going to the
> right, each line in turn.
> 
> The two Int16 values can be split into bits:
> 
> 0000 tile properties / flags:
> 
>       bits     len  description
>        0- 3    4    tile type (0-9, see below)
>        4- 7    4    floor height
>        8       1    unknown (?? special light feature ??) always 0 in uw1
>        9       1    0, never used in uw1
>       10-13    4    floor texture index (into texture mapping)
>       14       1    when set, no magic is allowed to cast/to be casted upon
>       15       1    door bit (when 1, a door is present)
>                     Tiles with this bit set have a door, but not every tile
>                     with a door has this bit set. Perhaps this bit tells if
>                     an NPC can open the door?
> 
> 0002 tile properties 2 / object list link
> 
>       bits     len  description
>        0- 5    6    wall texture index (into texture mapping)
>        6-15    10   first object in tile (index into object list)

We are interested in the last 10 bits, which refer to the `first object in tile (index into object list)`. To deal with this in an easier way, let's copy this over to notepad. Select the 4 bytes of the tile in HxD and, on the right, there's a panel called `Data inspector` with a bunch of numbers, `Binary`, `Uint24`, etc. There's one line written `UInt32` with the value `2099393`. Copy this to clipboard, making sure that the `Byte order` at the bottom is `Little endian`[^1], and paste this in the windows calculator, making sure you're in the `DEC` data type.

![calculator](/assets/content_posts/2023-06-16-Fixing-UW1-error-object-list/calculator.PNG)

This will convert the number into several representation types, including binary. Copy the number `0010 0000 0000 1000 1100 0001` number over to notepad. This number looks a bit short. Remember it's supposed to be 4 bytes long, or 32 bits. This is because the leading zeros were removed. Add the missing zeros to reach 32 bits (`0000 0000 0010 0000 0000 1000 1100 0001`). Remember that the 10 last bits represent the number of the tile. We have to set this to 496. Convert this number to binary (`0001 1111 0000`) and truncate it to 10 digits (`01 1111 0000`). Paste it below the number above and align the spaces, like this:
    
    ____ ____ __
    0000 0000 0010 0000 0000 1000 1100 0001
    0111 1100 00

Now, replace bits above with the bits below (equivalent to an `OR` operation)

    0111 1100 0010 0000 0000 1000 1100 0001
    
Copy this from notepad and paste into the calculator (within the `BIN` data type) and copy the decimal number (`2082474177`) and paste it into the `UInt32` field in `HxD`. This will update the data on the left, setting it red. The bytes will have changed to `C1 08 20 7C`.

If you save this modified `LEV.ARK` in HxD and then load the game in Dosbox, when you go down the stairs, you'll notice the bug message won't appear anymore. Congrats! Bug fixed. However, you'll notice the arrows are unreachable and barely visible.

![arrows1](/assets/content_posts/2023-06-16-Fixing-UW1-error-object-list/arrows1.png)

This is because the object data controls its height in the tile. To fix this, we'd need to modify the object data. If you came here just to fix the bug, then you're done! What we're exploring now is just to leave no loose end.

## Altering the object hex data.

To find the object position, we won't have the same convenience as before, with the editor. We can use math or we can search for the sequence of bytes provided by `uwdump`.

Since this is object 496, static object 240, we do this calculation to find its offset:

    542 (header) + 1 * 31752 (skip 1 level block) + 16384 (skipping all tiles) + 6912 (skipping mobile object data + 240 * 8 = 57510 (0xE0A6)

If you want to use the buffer provided by `uwdump`, first, a warning. It's technically possible to have the same sequence of bytes somewhere else in the file. You could change that and not modify what you want. That aside, first, you need to swap the bytes because of the endian order, like so:

    8012 2b80 0016 0200 (`uwdump`)
     \/   \/   \/   \/
    1280 802b 1600 0002
    
In HxD, go to `Search`->`Find`, then in `Hex-values`, and paste `1280 802b 1600 0002` and click `OK`. You'll get transported to the first match location.

And yeah, it's the same as what we calculated before!

![hxd_object](/assets/content_posts/2023-06-16-Fixing-UW1-error-object-list/hxd_object.PNG).

To modify this, we'll need to check again [`uw-formats.txt`](https://github.com/vividos/UnderworldAdventures/blob/fb4b846f0262073ef879d92fff754dfe9ccc6a5e/uwadv/docs/uw-formats.txt#L1616), section 4.3.

> The "general object info" block looks as following:
> 
>         bits  size  field      description
> 
> 0000 objid / flags
> 0- 8   9   "item_id"   Object ID (see below)
> 9-12   4   "flags"     Flags
> 12   1   "enchant"   Enchantment flag (enchantable objects only)
> 13   1   "doordir"   Direction flag (doors)
> 14   1   "invis"     Invisible flag (don't draw this object)
> 15   1   "is_quant"  Quantity flag (link field is quantity/special)
> OR
> 9-15       texture number
> 
>         Note: some objects don't have flags and use the whole lower byte as a
>         texture number (gravestone, picture, lever, switch, shelf, bridge, ..)
> 
> 0002 position
> 0- 6   7   "zpos"      Object Z position (0-127)
> 7- 9   3   "heading"   Heading (*45 deg)
> 10-12   3   "ypos"      Object Y position (0-7)
> 13-15   3   "xpos"      Object X position (0-7)
> 
> 0004 quality / chain
> 0- 5   6   "quality"   Quality
> 6-15   10  "next"      Index of next object in chain
> 
> 0006 link / special
> 0- 5   6   "owner"     Owner / special
> 6-15   10  (*)         Quantity / special link / special property
 
We are interested in the `zpos` of the object, its height. This is in position 0002, or the second short, in the initial bytes. We're placing it on floor height, so check the tile height in the editor (it's 12) and multiply this by 8 to get the value we need to set here (96). The reason for this is explained elsewhere in `uw-formats.txt`.

As before, copy the object data as a number (`144115283294978066`), but this time as a UInt64, since it's 64 bytes long, and paste it into notepad in the binary notation (`0010 0000 0000 0000 0000 0001 0110 0010 1011 1000 0000 1000 0000 0001 0010`). We want the beginning of the second short, so, from the right to the left, we count 16 bits and select the next 7, which is the length of `zpos`.

                                                  ___ ____ xxxx xxxx xxxx xxxx             
    0010 0000 0000 0000 0000 0001 0110 0010 1011 1000 0000 1000 0000 0001 0010

The rest is totally irrelevant. This object has a height of 0, which is why it appeared so far away. To set it to 96 (`110 0000`), do as before. You'll get

      0010 0000 0000 0000 0000 0001 0110 0010 1011 1000 0000 1000 0000 0001 0010  
                                                    110 0000                      |
    = 0010 0000 0000 0000 0000 0001 0110 0010 1011 1110 0000 1000 0000 0001 0010  

Which is `144115283301269522`. Paste this number back in HxD and the data will modify to `12 80 E0 2B 16 00 00 02`. Save this again, and reload the level. Here's the result:

![arrows2](/assets/content_posts/2023-06-16-Fixing-UW1-error-object-list/arrows2.png)

Congrats! Give yourself a pat on the back, you've conquered the dreaded object list error!

[^1]: Big endian and little endian refer to the meaning attributed to the bits. Consider the number 1234. 4 is the least significant number, while 1 is the most significant, since it means 1000. In big endian notation, 1234 would be "stored" in the sequence 1234, because it starts with the big end, and in little endian notation, it'd be stored as 4321.
