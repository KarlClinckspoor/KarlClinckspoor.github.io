---
layout: post
title: My story with the Ultima series
description: My story with the Ultima series
author: Karl Jan Clinckspoor
tags:
- ultima
- games
- personal
---

In this post, I'm sharing my personal story with the Ultima series. This seems like a weird 
theme for a post, especially on a "CV"-type blog, but whatever, I wanted to share something. 
Maybe it's nostalgia speaking.

Disclaimer: This recollection is the best version I could remember. The actual facts might be
different, so there might be some inaccuracies and discrepancies.

Growing up in the early 90s, I had limited access to computers. Not only were they expensive, but I
had absolutely no idea how to use DOS, configure memory, and couldn't read English very well. This
limited my capacity to play games on the computer, so I played mostly on my SNES and N64. My father
bought a collection CD, maybe around 95-96, with a ton of
really good games (perhaps a type of EA classics?). In it, there were 3 ultima games, Ultima
Underworld, Ultima VII and Worlds of Ultima: Savage Empire. Other notable examples are Chuck
Yeager's Air Combat, Shadowcaster, Populous and Space Hulk. We had some other games before, most 
on floppies, but this CD was an absolute gold mine.

My father knew English the most in our home, so he could read the manuals and install stuff. 
These were DOS games, so you had to quit out of Windows and restart in DOS mode. There, you had 
to type the commands to install and run the games. After installing, he would leave handwritten 
notes on the margins with what commands you had to type to run the games.

Still, the complexity of these games was too much for kid me. I think the only one I could play 
was Air Combat, because we had a joystick, and the interface was relatively easy. Since many 
missions started and ended while airborne, no difficult takeoffs or landings were necessary. And 
there were cheats for infinite ammo and no damage. The other games I just watched my father play.

I have very little memory of this. The only game I remember him playing was Ultima Underworld. 
Specifically, I remember the creepy skull that appeared when you died, and the creepy blue face 
of Garamon during the intro. He never got very far, at most level 3 I think (from a talk a few 
years later).

These were good times, and I was innocent, and very very bad at videogames. Morton's castle (nr 2)
in Super Mario World was too tough for me. Zelda Ocarina of Time was a matter of exhaustive trial
and error (couldn't read) and playground rumors. Sometimes, someone's cousin or uncle would get past
an obstacle and then pass the knowledge to the younger generation. I remember studying and
theorycrafting about a ton of aspects of these games. I think that in Zelda, there's some passage a
Kokiri kid says that suggests a curse befalls the forest when a Kokiri leaves, or something to that
effect. This got distorted by the game of telephone of the playground, and a kid was 100% sure that,
when you leave the forest as a kid, after beating the Great Deku Tree, you doomed the forest.
Actually what happened was that the day-night cycle advanced in Hyrule Field, and the creepy
Stalkids appeared at night. We were deathly afraid of these skeletons, so we couldn't leave the
forest anymore. To top it off, it was dark, because it was nighttime, so it looked cursed. We were
convinced we had doomed the land. I remember one day just braving the skeletons and running past
them. I even visited this friend of mine and showed him how to get out. He almost took the
controller out of my hands because of how frightened he was. 

You can notice these stories are mostly about console games. My overall story playing PC Games, 
especially Ultima, is mostly a lonely one. I never really got into multiplayer games all that 
much. And I'm the only person I know that knows of Ultima, so I don't really have people to talk 
about that.

Anyway, going forward a few years, after 2000. We had moved and upgraded our PC. The internet 
was starting to catch up. We had that really really slow dial-up internet that cost a fortune. 
My knowledge in English, due to study at school, sheer hard-headedness and determination while 
playing videogames, was growing. I could, and did, read stuff in English for pleasure. This is 
when that collection CD comes into play. It came with a thick manual, which I treasure dearly, 
containing details from all the games included in the CD[^1]. In those times, game manuals were 
*good*. They came with background story, lore, and often a lot of vital info that wasn't 
available in the games themselves, in a sort of crude copy-protection scheme. Shame the CD didn't 
come with the goodies of each game, a staple especially of Origin Systems games. 

I would read and re-read the sections of the Ultima games. Savage empire has a great "Indiana Jones"
-esque introduction, while Ultima VII had a book that told the story of the world through the lens
of the Fellowship, trying to make them less perverse at first glance. Reading all of this game me
great pangs of nostalgia. At this time, however, Windows compatibility with DOS was severely
lacking. No DOSBox[^3], and Windows XP, which came out in 2001, had ditched the capacity to reboot
in DOS mode. Being a complete noob, and a kid, I wouldn't dare to try to downgrade the computer to
Windows 98 just to play games. The thought never even crossed my mind.[^2] So the only thing I 
could do was to fantasize about these worlds. Ultima Underworld, especially, was my obsession.

I remember reading about a fix to run Ultima Underworld on NT systems (like XP). I still have 
the file, or what I think it the file, curiously. Here's the readme file of `uw2nt.exe`.

    ---------------------------------------------------------------------------
        uw2wnt.exe  version 1.1
    
        patch makes ultima underworld I und II running with windows nt
    
        according to the problem solution by Moscow Dragon
        see following copy of news entry
    
    --------------------- begin of news entry -------------------------------
    From        : Moscow Dragon (maxim__s@mtu-net.ru)
    Subject     : UW2 on NT: looks like solved!
    Newsgroups  : rec.games.computer.ultima.series
    Date        : 2001-12-23 15:06:51 PST
    
        Patching    :
        - make a copy of the original UW2.EXE, for a case if something will go
          wrong.
        - open UW2.EXE in your favourite hex editor like HIEW (or use some UNIX
          tool).
        - go to offset 0x24719
        - you will see the bytes of:
    
          FA 52 BA 03 00 E4 64 A8 02
    
          If you see some other bytes, do not proceed. Looks like you have some
          other build of UW2 then me.
        - patch the FA byte to C3
        - save and exit
    
        Technical details:
        - UW2 has a function which sends "set LED indicators" command to the
          keyboard ports, bypassing BIOS.
        - this way of accessing ports is incompatible with NTVDM and hangs it.
          NTVDM waits forever on some Win32 event as a result of the DOS app
          reading the port.
        - the patch switches the function away at all, C3 is RETN
        - I do not know whether the function is vital for UW2. Probably not.
        - for now, I also do not know whether the function called many times from
          UW2 or only during the introduction.
    
        Max
    
    --------------------- end of news entry -----------------------------------
    
    I found out that the offset is dependent on the Ultima Underworld version,
    but Moscow Dragon's solution should work with all versions and even with
    Ultima Underworld 1 too. Just search for the pattern and change the first
    byte according to Moscow Dragon's description.
    
    Many people don't own an hex editor or are not familiar with the operation
    of this tool. For all this folks I've created this patch which will do the
    necessary work automatically. BTW - should work means that I wasn't able
    to test this with all Ultima Underworld versions and with all Windows NT
    versions. My Ultima Underworld 2 version is different from Moscow Dragon's
    (not the same offset) but this patch program works fine for both of my
    Ultima Underworlds and with my Windows 2000 (SP2). Can't say anything about
    Windows XP but Windows 2000 is actually Windows NT 5.0 and Windows XP is
    Windows NT 5.1. So it should work, but who knows.
    
    Put the patch program uw2wnt.exe into the Ultima Underworld directory and
    start it. uw2wnt will look for UW.EXE and UW2.EXE and patch it so you can
    use it with Windows NT (2000, XP). It will also create a backup of the
    original file which is called UW.EXE.BAK or UW2.EXE.BAK. The patch program
    will shelter the back up file from overwriting by setting the read only
    attribute.
    
    After patching you must modify the properties of UW.EXE respectively
    UW2.EXE. Select the Memory tab (hope this is the correct term, I own
    the German version and my tab is called 'Speicher' which means 'Memory').
    Select the Expansion Memory (EMS) listbox and choose 8192. Then select
    the Screen tab and activate the Fullscreen radio button (again I can only
    guess the English terms, just secure that Ultima Underworld don't start
    in a window).
    
    Run Ultima Underworld 1 and 2 with VDMS for sound and music. Get VDMS for
    free, visit http://www.ece.mcgill.ca/~vromas/vdmsound .
    
    Now have fun with this patch and let me know if you like it ;-)
    
    
        development : Sir Cabirus Dragon aka Frank Wolter
        email       : SirCabirus@gmx.net
        homepage    : http://www.SirCabirus.com
      ---------------------------------------------------------------------------

As you can imagine, a lot of those terms went right over my head (*Hex editor*?). I found this tool
and ran it on my copy of UW, and even used `VDMS`, that tool to add sound and music, but I couldn't
get it to work. I don't know where I got the idea, but I thought I had a defective copy of the game.
I absolutely *scoured* the internet looking for a download of UW1, and even made some bids on ebay
for floppy versions[^4], but eventually, I found it, on page 20+ or something. Google, at that time,
was very different from now. And I was a persistent kid. The only thing I remember was that the 
site was perhaps blue-ish? In any case, I downloaded it, probably with my trust NetAnts download 
manager. I still have the file, dated to 2003/10/16 (YMD). Hopeful, I tried to run the game.

Nope, still couldn't do it. All this was in vain.

I kinda gave up at this point. Nothing I did got me closer to playing these games. It was at 
this time that, for reasons I can't remember, my father brought back our old computer (or was it 
a work computer?) and it ran Windows NT 4.0, which had the option to reboot to DOS. I 
immediately jumped at that opportunity. I installed UW1 successfully and got it to run. It was 
perfect to me. The only downside was that we had no speakers or sound cards, so the game was 
100% on the internal PC speakers, with its limited beeps and boops when I swung my weapons. But I
was *elated*. I even went all the way down the map let you, lvl 99, and told my father the game was
really long. Actually, the game only had 8 levels[^5], but the automap wasn't limited by that. 
Perhaps the extra space was there for player annotations? It was at this time I reached lvl3, 
which is when my father told me he never got further than this.

I *think* I managed to finish the game in this old computer, but I'm not sure. I do remember 
that it rained one day, and the monitor was *right below* a leak in the ceiling, and the monitor 
fried. Oh well.

In any case, I had fulfilled my first wish. Now I had to see what the other games in the 
series had in store. Without a computer that could run them, I was out of options. Besides, I 
didn't have the other games in the series. Back into the bowels of internet I went.

It just so happens that around this time, ~2004, there was a website that had *all* the Ultima
games, from I to VIII, to download, for free, in self-extracting zip executables. Downloading 
and running this kind of stuff is unthinkable nowadays, but I was a desperate kid. I even have 
managed to keep a hold of these files to this day. They're all dated 2004/07/12 and 2004/07/15, 
during my Winter holidays. 

The website was called `the penalty`, and it added a `.nfo` file to the game files. This is what it 
contained:

    This archive passed through:

    http://free.techno-link.com/apenalty/

    COME AND DOWNLOAD YOUR ULTIMA !!!

The wayback machine even has it preserved! The link is a bit different, but you can access
it [here](https://web.archive.org/web/20080410135029/http://apenalty.hit.bg/). I clearly remember
this red-on-black tone and weird font; gave me some bad vibes at the time. `The Penalty`, if you're
out there, I thank you immensely. I downloaded a backup of this
website, just in case ([see here how I did it](https://linuxnightly.com/how-to-download-a-website-from-the-wayback-machine/)).

I think it was around this time I discovered DOSBox, so I could really play a lot of these games.
Or perhaps I played some "alternative" versions first? My memory is fuzzy. If I ever have the 
patience, I could try to look around the [VOGONS forums](https://www.vogons.org/index.php), 
which I briefly was a member of, to pinpoint better a date. It was also around this time I found 
the [Bootstrike](http://bootstrike.com/Ultima/) website, which is still up. Some time later, in 
2006, I even sent him a very poor walkthrough I made of Ultima VII, and it's still up. It's bad, 
but hey, I was 16.[^6]

And it was here that my journey through the Ultima games truly began. I had the tools to run the 
games, and the games themselves. Unfortunately I don't really remember which games I played, and 
in which order. In any case, I certainly played VI, VII, UW1, UW2, SE, MD, VII-P2, VIII. The 
earlier ones, I think I played only I and II. To this day, I haven't played III and V (in the 
original, I played Lazarus), and only recently I played IV. So you see, even at that time, I 
couldn't stomach a lot of old game design philosophy.

Some specific memories I have from this period:

1. I remember being surprised by the weird language the games had. The "thee"s and "thou"s. I had
   never read anything like that before, but I got used to it. Some time later, I think during what
   would have been called my 8th grade at the time, an English teacher asked the class what they did
   during their vacation, related to English learning. I mentioned Ultima VII (not by name) and its
   weird language. Almost certainly received eyerolls from the class, and a slightly disappointed
   look by the teacher.
2. I remember liking Savage Empire, but not finding it that special. What stuck with me, however,
   was googling the game and finding a blog by a girl that *loved* the game. She would post stuff
   and sign off with "*see you later for another post in the valley of Eodon*". I kinda liked the
   game more just because of her enormous enthusiasm for the game. I've tried to find this blog in
   recent times, but was unable to, unfortunately.
3. I remember being enchanted by Martian Dreams. Of the 3 games in the UVII engine, it was the one I
   liked the most.
4. I played through Ultima Underworld again, starting from scratch. I even emailed Sir Cabirus,
   [of the fantastic web-walkthrough fame](https://www.sircabirus.com/) to talk about builds. He
   told me he played a Druid, for the high strength and intelligence, so he could cast spells
   *and* carry stuff around. In all my subsequent playthroughs of UW1 since, I played as a druid. I
   also remember being a supreme hoarder, and having occupied one of the storage rooms in the
   Mountainfolk part of Lvl2. I would go back and forth, carrying food, weapons, armor, light
   sources and neatly placing them around the room. I also commented about this to Sir Cabirus, and
   he said he mostly carried what he needed with himself. And yeah, money is nigh-useless in UW1, as
   is hoarding food or light sources. It's all very abundant, unless you play very slowly. I think
   it was at this time I got good at reading maps, because of the back-and-forth. I would open it,
   memorize the path, and follow it. I even showed a friend/neighbor how I navigated around, cutting
   corners in the dark corridors of the Abyss. He was a bit surprised, because of how good I was,
   and how useless this skill was.
5. I played Ultima Underworld 2 and was very struck by the second to last level. I even adopted 
   the name as my moniker since. If I had to choose between UW1 and UW2, I think I would go with 
   UW1. A lot because of nostalgia, but also because it's much more straightforward.
6. I remember going around in Ultima VII and talking to people, until I noticed Iolo wasn't 
   there with me, possibly because he was supposed to join a conversation, but didn't. I never 
   dismissed him, so I got very worried. For *some* reason, my kid-brain remembered I had a 
   tough fight during the Test of Strength of the Forge of Virtue addon. However, I had finished 
   it, and couldn't get back. But I *also* remembered having Marked one of my stones to just 
   that place, perhaps to get back to the dragon more quickly, so I cast Recall and snuck back 
   to the forbidden area. And voilá, there was Iolo's body, among the many bodies of our enemies,
   in the first room. I revived him and pretended nothing happened. My companions spared 
   telling Iolo of what happened to him.
7. I remember *really* liking the plot of Serpent Isle, and connecting it to previous Ultimas, 
   like I (Shamino's past) and V (Blackthorn's fate). The unfinished aspects of some areas just 
   made them even more mysterious. I also remember breaking the progression by talking to one of 
   the ghosts in the ruins of a lighthouse, and doing exactly the same in all my subsequent 
   playthroughs, because I didn't know how to do the quest without breaking it that way. Also, I 
   always got frostbite when entering the cave-prison as punishment from the mages, and no 
   amount of warm clothing protecting me. Getting killed and being resurrected fixed it. Weird 
   reproducible bug. I also remember having the toughest time killing Aram-Dol, so I resorted to 
   abusing the game's pause feature and killing him with the ritual bloodletting device. Death 
   by 1000 cuts (or 30-ish to be exact).
8. I remember disliking the gargoyles in VI and often killing them for distaste (and experience),
   even though I played VI after VII, and I knew the gargoyles were the good guys. Years and 
   years later, I saw Richard Garriott, the creator, talking about how he wanted to make the 
   player *racist* and boy, did I fall for that.
9. I remember playing Ultima VIII and being enchanted by the atmosphere. The magic system was 
   one of the best of the series, in my opinion. I actually liked taking notes and performing 
   the rituals, especially fire sorcery. I had the pleasure of having aimed jumping and other 
   fixes, so my experience wasn't as bad as other people's.
   
And that was it. I finished Ultima VIII, returned to Britannia and saw the Guardian head. Since 
none of the websites I visited talked much about Ultima IX, I simply thought it didn't exist. Or 
would be released in the future. This was perhaps around end of 2004, early 2005.

In actuality, it had already been released, but it came in 1 or 
2 CDs. Downloading that much data was absolutely impossible in the era of unreliable Dial-Up 
connections. Still, I tried to find it, but no luck. I had to buy the CDs, but as a broke kid, I 
couldn't afford an original, imported from the US. The game wasn't for sale anywhere.

Well, it's already established I pirated most of the Ultima games. In my defence, there wasn't 
really a viable way for me to obtain them legally. And the same was about to happen again. By 
sheer luck, there was a dude that had a website that sold pirated CDs of games and software -- 
and he had Ultima IX! I bought it from him and received it in the mail some time later. As usual,
I still have some of the files from this era stored. If you're reading this, owner of `www.procd.hpg.com.br`, I am deeply thankful to you.

So I installed and ran it without major hiccups that I can remember. Yes, there were crashes, 
but I managed to play it from start to finish. Ultima IX is kind of a black sheep of the series. 
Many people hate it and what it did with the story, especially Spoony. I never felt this 
strongly about it. The line `What's a paladin?` never struck me as weird. It was clearly a way 
of letting new players learn more about the world. But I do understand the *implications* of it, 
after all, Dupre is a major companion since IV, and his sacrifice in VII-P2 was all the more 
significant for it.

I liked the puzzles and the dungeons, but didn't like the new world layout or the closed, hand-holdy
nature of the story progression.[^7] I didn't feel particularly satisfied with the
"I'm your opposite, Avatar" plot-twist the Guardian throws at you, or the sacrificial ending, or the
character assassination that was Blackthorn. But I still enjoyed the game, despite Hythloth, and
felt some closure after defeating the Guardian once and for all.

So yeah, this is what I wanted to tell you at this time. Hope you enjoyed going through my 
memories with me.



[^1]: I still have it, albeit with a replacement cover, and the pages are dirty and crinkled from use.

[^2]: My first time installing an Operating System is a tale for another time. 

[^3]: DOSBox came out in 2002, yet I only found out about it many years after the fact.

[^4]: thankfully never outbid anyone - and never insisted a lot on this. It freaked me out.

[^5]: or 9, if you count the ethereal void

[^6]: I think I still have my handwritten notes stored somewhere. I should scan them, for posterity.

[^7]: I do recognize, now, that it's much less linear after some time, or not linear at all if you creatively jump, or use bread. The game still expects you to follow the intended path anyway.