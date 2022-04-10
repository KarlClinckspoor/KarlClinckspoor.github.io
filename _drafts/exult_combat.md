---
layout: post
title: Exult combat
description: A brief description of how Combat in Exult works
author: Karl Jan Clinckspoor

tags:
    - games 
    - ultima 
    - c++
---

I decided to peek into the source code of [Exult](https://github.com/exult/exult) and figure out how
combat works. Here's the result of quite a few hours of code reading, note writing, source code
manipulation and running some simulations. I hope this brings you a bit of satisfaction, and I'm
certain you'll read some parts and think "Oh, so that's why this happens!"

A few disclaimers before we start:

* The combat in exult does not work exactly like in the originals. It's simplified. The combat goes
  by so quickly that many of the nuances can't be properly felt.
  (Thanks for DominusExult for pointing this out to me!)
* I'm not well versed in C++, so it's possible I grossly misread/misinterpreted something. Please
  point it out! However, I'm simplifying some of the code for conciseness, e.g., many if statements
  are quite complex, and I'll only mention the most relevant aspects at each part.
* I've decided to forego a bit the "blow-by-blow" description of what each function does to a more
  general approach, to improve the readability of this text.
* I'm using the master branch on github, commit 4df38f34, if you want to follow the code some parts.
* Initially, I made this 3x longer, but I opted to cut down a lot of the code logic/descriptions and keep
  only the main logic. I've made a flowchart if you want to follow the logic a bit closer.

With that out the way, let's start.

## How combat works

![Flowchart](/assets/img/exult_study/Flowchart.drawio.png)

* Combat is a type of `Schedule`, which controls every action an npc (`Actor`) performs. Other
  schedules include baking, serving food, etc. Aggressive actions do not happen outside of combat (
  save for usecode).
* To enter combat, the npc needs to:
  * Spawn in the combat schedule
  * Be in a patrol schedule and find an NPC with a clashing alignment
  * Be in the Avatar's party and have the UI switch to combat mode
  * Be attacked and able to fight back.
* There are 4 npc alignments: neutral (0), good (1), evil (2), chaotic (3). Evil is aggressive to
  chaotic and good, chaotic is aggressive to all, good is aggressive to evil and chaotic. Good is
  mostly the Avatar's party. Evil are mostly enemies, but some "peaceful" npcs also spawn as evil.
* The game goes through all events it has to handle. An npc can be either performing an action or figuring out
  what action it'll do with the function `now_what` from its schedule.
* The combat schedule is a finite state machine with the states `initial`, `approach`, `strike`,
  `fire`, `wait_return`.
* Every cycle in the `approach` state (which I'll call a "turn"), an internal counter
  called `dex_points` increases based on the npc's dex attribute. When this reaches `dex_to_attack`,
  which is 30, it can change its state from `approach` to `strike` or `fire`. Checks are performed 
  to see if there's a line of sight or if the npc should move. The excess value is carried over between
  iterations, so an npc with 29 DEX, after 2 turns, has 29+29+29(-30) = 28 points.
* To determine if an attack hits, the attacker's combat attribute `attval` is compared to the
  defender's `defval`. 
  * `attval` is altered by the game difficulty (up to +/- 6, or 2 times the difficulty), the
    weapon type (good thrown, poor thrown, ranged) and the range in the case of thrown weapons (
    poor thrown removes 1 from attval per distance unit, good thrown removes 1/2).
  * `defval` is only affected by the Protection spell status, which increases `defval` by 3.
  * the probability of hitting is `(15 + defval - attval + 1)/30`. However, there's always a 1/30
    chance of hitting and a 1/30 chance of missing.
* When an attack hits, the hit points lost and other effects are calculated.
  * Attacks can have damage types (normal, fire, magic, lightning, ethereal, sonic) and powers (sleep,
      charm, curse, poison, paralyze, magebane and two extra).
  * If the weapon's damage is 127, it's an instakill. Armor and any other resistances are ignored.
  * Base damage starts at 0.
  * Game difficulty adds/removes damage depending on the bias, up to +/- 3. This only affects damage
    being dealt by/received the party. Combat between 2 random monsters isn't affected.
  * A random value from 1 up to a third of the attacker's strength is added to the base damage, if the
    weapon doesn't deal lightning damage. This is compensated by Lightning ignoring armor.
  * A random value from 1 up to the weapon's attack rating is added to the base damage. 
  * An npc's armor received the opposite bias (Game difficulty) as the weapon damage, meaning it increases
    party armor at lower difficulties and decreases at higher difficulties, and the opposite for enemies.
  * An npc's base armor value is summed with every armor item in its inventory to give a final armor
    value.
  * A number between 1 and the total armor value is removed from the base damage.
  * Lightning, ethereal and sonic damage ignore armor (armor is set to 0).
  * If the npc is immune to that specific damage type, damage is set to zero. If the monster can't die,
    the same is made.
  * The damage dealt is the attack value minus a random number from 1 to the npc's armor stat. 
  * If the npc is vulnerable to that specific damage type, damage (after removing armor) is multiplied by 2.
* Health is considered in the following manner:
  * Max health is equal to strength.
  * If health falls to or below 0, the npc is knocked unconscious.
  * If the health falls below 1/3 of its max health, it dies.
  * Some npc's have a `tournament flag` on, which hands off the handling of death/defeat to the usecode
    machine, meaning they receive special treatment.
  * An npc is considered defeated if it's knocked unconscious, and exp is calculated.
* Exp is calculated in the following manner:
  * If the npc was unconscious when defeated, exp is set to zero.
  * The combat, strength, (dex+1)/3, int/5, monster base xp, total armor protection value, worn
    weapon's base xp with some range bonus and monster immunity/vulnerability count are all summed.
  * This value divided by 2 is the total exp gained.
  * The current level can be calculated with `1 + log2(total exp / 50)`. Every level up, 3 training
    points are added.
* Weapon powers are processed by:
  * If the npc was under the protection spell, no powers can affect it, even magebane.
  * To apply an effect, a check is made comparing the attacker's intelligence and the defenders
    strength (Poison, paralyze) or intelligence (curse, charm, sleep). It's the same calculation
    used when processing if an attack hits.
  * Magebane always works. It sets the target's mana to zero and removes all spell "weapons" from the npc.
* Combat ends when:
  * The UI is changed.
  * All enemies are dead and the npc failed to find an enemy several times.
  * The npc despawns/becomes dormant.

## Other fun stuff

### Combat_trace and show_hits

You can enable a debug feature in Exult called `combat_trace` to print out some additional
information about what's going on in combat to what's called `standard out`, or `stdout`. If you run
exult from a console, it'll print out to that, or it'll be in the file called `stdout.txt`
in the Exult folder. To enable this, modify or include these lines in `exult.cfg`:

```
<debug>
    <trace>
        <combat>
            yes
        </combat>
    </trace>
</debug>
```

If you want, you can also enable the option called `show_hits` in the game's options. This shows
how many hit points an npc lost, and how many are remaining.


### How long it takes to "charge up" an attack

Dexterity controls how quickly an actor can attack. A dex value of 30 means they can
attack every other turn. If the npc's dex is 29, the actor only attacks after 3 initial turns, and
then the accumulated `dex_points` lets it attack every other turn. Only after **59** turns its
excess points are spent and the npc will have to build up points for 3 turns again, and attack on
turn 62. If the npc's dex is 15, this means it will always attack once every 3 turns. A dex
attribute of 1 is comical. The npc will stare at the target for quite a while until they attempt to
raise their arms to attack.

I've... spent way more time trying to illustrate this relatively inconsequential aspect of combat
than I should have. You can skip to the next section of you want. Here's a few graphs:

How many attacks can be performed given a fixed number of turns. Notice how, for short battles, the
number of attacks is the same for relatively wide ranges of dex values. Only for really long battles
do we see a clear trend favoring higher dex values.

![Number of attacks for n turns](/assets/img/exult_study/Number_of_attacks_for_n_turns.png)

Here we can see how many turns are required to perform a specific number of attacks. Notice the
logarithmic scale in the y axis. For absurdly low dex values, it takes *ages* to perform an attack.
Note also how 30 dex is favored, especially for shorter battles.

![Number of turns for n attacks](/assets/img/exult_study/Number_of_turns_for_n_attacks.png)

Last we can see a visualization of attack frequency and the role of the leftover `dex_points` on
each turn. Here, an attack being performed in a turn is indicated by a peak. The closer the peaks,
the faster are the attacks coming. Notice how sometimes there's 3 turn gaps, then 2 turn gaps, then
1 turn gaps and finally at 30 dex, there are no gaps — an attack goes through every other turn.

![Hit frequency visualization for 30 turns](/assets/img/exult_study/hit_frequency_visualization_after_31_turns.webp)

You might be wondering how long a turn lasts. To test this, I went to the Trinsic stables and
modified the Avatar's stats to 1 STR and varying DEX values. Then, I modified the game engine to
print out the current state and the game's tick number at that instant (1 tick is 1 millisecond
since the game started running). I then attacked the horse and noted down how long between I
started "charging up" and I finally performed the attack animation. The speed the game runs depends
on the set fps, so I did vary the fps values also. Here's the results.

First, we see how long it takes to start a strike. This is calculated from the moment I double click
the horse to induce the attack (and initiative starts building up) to the moment the Avatar decides
to hit the enemy. As expected, this time depends on the fps and roughly half the fps leads to a
delay twice as long. I've measured these a few times each and noticed there's a variation in how
long it takes to hit the target, and this increases at lower dex and lower fps. Nevertheless, the
highest standard deviation I found was 80 ticks, or 0.08 seconds. Imperceptible in normal
circumstances.

![Hit timings](/assets/img/exult_study/times_lim.png)

If we take the timing at 30 dex and consider this 2 turns, we can rescale this plot and compare to
our previous result. This is what we get. Note how the actual number of turns to attack varies
between fps. I don't know the exact cause of this, but I guess my assumptions might be too
simplistic.

![Measured and theoretical number of turns](/assets/img/exult_study/turns.png)

### A visualization of hit frequency for two melee enemies and different game difficulties.

For combatants with equal combat stats, the attacker is always *slightly* biased towards hitting. To
better visualize how this works, and to show how bias affects the probabilities, I've prepared some
hit probability maps containing theoretical values of hit probabilities, from game
difficulty -3 to +3. These consider you're attacking (i.e. party member)
a random monster.

{%include exult_study/plotly_figs.html %}

We can see when difficult is 0, the 50% hit probability is slightly offset downwards, indicating the
green-yellowish area (>50% hit prob) is bigger. Then, as you go from -3 to +3 (easy to hard), the
yellow region shrinks considerably. Since the bias is symmetrical, if you want to know how probably
it is for a monster to attack your party, you just need to consider the opposite sign graphs (i.e. a
monster to-hit map at the hardest difficulty is the -3 map, and so on.)

### Examples of foes you can safely interact until you enter combat

You can calmly walk around and talk to npcs with hostile alignments
provided none of you enter the combat state. The instant you press 'C' however, you'll attack. For
example, Iriale Silvermist, located in the Fellowship retreat, is Evil but peaceful (for some time),
as well as some npcs in New Magincia (Robin, Battles and Leavell).

You can also do some shenanigans and try to change the Avatar's alignment to `Evil`, either with 
Exult Studio (I failed at that) or modifying the variable using a debugger, and see if you can
complete the game.

### Combat schedule states and attack modes that aren't used

The `parry` and `stunned` states aren't used anywhere in the code. Neither is the attack mode `flank`
or `defend`. `berserk` is the same as `random`, but prevents npc from fleeing.

### Health-based actions

* If the npc's health is less than 3, it tries to flee.
* If its health goes from above 50% to below 50% from an attack, it can yell "Ouch!" or equivalent.
* If the Avatar received more than 1/3 of its health in damage or its health is below 1/4 of max or
  received lightning damage, the screen flashes red. Other attacks flash only the outline red.

### How enemies use spells

Enemies don't require a spellbook to cast spells. Rather, they have spells are weapons they can fire
a limited number of at enemies. If you manage to get one, you can use it too. This makes
spellcasters especially dangerous, since they have no mana requirements. When killed, these items
are removed from them so you can't access them natually. However, I distinctly remember a few
jesters around the castle of the white dragon that spawned with death bolts or something like that,
and which I looted and used for a bit. For example, Aram-dol has two spells equipped, one with 99
uses and another with 2. Good luck surviving 99 casts of some spell. If you have Exult Studio
equipped, you can open an npc's inventory gump and see the spells. Here's an example of a mage
located at an island to the south of Trinsic.

![Mage inventory](/assets/img/exult_study/exult_mage_items.PNG)

### Experience and levels

An npc's level is calculated by
`1 + log2(total exp / 50)` (see `actors.h:490`). If you want to calculate the exp for a specific
level, you can use `25 * 2 ^ level`. Every level gives the npc 3 training points. I prepared two
graphs with this info:

![Experience and level](/assets/img/exult_study/Experience.png)

![Experience and level, log scale](/assets/img/exult_study/Experience_log.png)

### How to compile exult myself and make modifications?

I'll preface and say compiling exult was a bit of a pain, because I can't use Visual Studio (disk
space concerns). If you can, then it's by far the easiest method to compile Exult. I managed to build
it in Windows (msys2), in WSL (so technically Linux, but under Windows), in Windows (Visual Studio) and
Linux.

First, follow the instructions in README.win32, README.MacOSX or INSTALL (Linux). If you can't get
it to work, send me a message and I'll try to help you. I have all my steps recorded, but there's a
few aspects I don't understand, so I won't post here, lest it confuses people.
