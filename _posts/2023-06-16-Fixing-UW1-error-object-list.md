# Solving "Underworld Internal Error - Problems in object list" without resorting to nukes

## Introduction

It's possible that during one of your playthroughs of Ultima Underworld 1, you were faced with this problem. Reading online, you noticed it has quite a nasty reputation and can lead to softlocks, requiring you to revert to a previous save or potentially having to start over. Neither option is very appetizing, so you look for the possible origin of the bug and solutions. Many people tell you it can be triggered if there are too many objects in a level, which happens if you, for example, farm enemy spawns or hoard items in a level, and that the solution is to chuck everything in water or blow everything up with fireballs and cheats/exploits. It works, the bug disappears, and you continue to play happy, but a bit annoyed.

While it is true, there is a 1022 object limit in a level (254 npcs/monsters and 764 items); this bug can happen way before the item list is full. While nuking *can* also work to recover these files, it's still less than ideal. With this post, I'll attempt to provide instructions to fix this bug yourself using a hex editor.

![Message](/assets/content_posts/2023-06-16-Fixing-UW1-error-object-list/Message.png)

## Requirements

For this tutorial, we'll require:

* [a corrupted save](https://1drv.ms/f/s!As-yStiKjmVHmfMdsA7CR_IvfgrTZw?e=vDvdcI) that triggers the error when you go from Lvl1 to Lvl2 by walking forwards a bit. Copy the contents to the folder `SAVE1` in your ultima underworld 1 installation. Note that this guide requires access to the files of the game, so you'll need to extract `game.gog` with any appropriate tool (such as 7zip), then configure dosbox.
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

Out of curiosity, you can go to the level objects viewer and see that there's, in fact, no item 496 (`Objects`->`Level objects`).

## Altering the hex data

You can close the editor now. Open `LEV.ARK` from the `SAVE1` folder using HxD. I recommend you backup your save before we continue, just in case. Click on `Search`->`Go To`, and type the offset we had. If you copy the one from the editor, be sure to select the `hex` option, and set `offset relative to` to `begin`.

![hxd_goto](/assets/content_posts/2023-06-16-Fixing-UW1-error-object-list/hxd_goto.PNG)


