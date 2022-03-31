---
layout: post
title: Exult combat
description: A primer on how combat works in Exult
author: Karl Jan Clinckspoor

tags:
    - games 
    - ultima 
    - c++

---

TASKS:

TODO: Add some traces to double check if it's only party members that give up combat and return to
the previous schedule

TODO: Triple-Check if one npc protecting another increases defense.

TODO: Check if heatmaps instead of contour is better to represent the probabilities. Also, check if
putting numbers in the heatmaps is more didactic.

TODO: Check all notes in the text.


I decided to peek into the source code of [Exult](https://github.com/exult/exult) and figure out how
combat works. I really enjoy watching/reading deep dives into game mechanics and thought,
"well, why couldn't I do one, and of one of my favorite series?". Here's the result of a few hours
of code reading, note writing, a bit of source code manipulation and running some simulations. I
hope this brings you a bit of satisfaction, and I'm certain you'll read some parts and think "Oh, so
that's why this happens!"

A few disclaimers before we start:

* The combat in exult does not work exactly like in the originals. It's simplified. The combat goes
  by so quickly that many of the nuances can't be properly felt.
  (Thanks for DominusExult for pointing this out to me!)
* I'm not well versed in C++, so it's possible I grossly misread/misinterpreted something. Please
  point it out! However, I'm simplifying some of the code for conciseness, e.g., many if statements
  are quite complex, and I'll only mention the most relevant aspects at each part.
* Anything pertaining usecode is, at this moment, a complete black box to me. I didn't get into it
  in the least bit.
* I'm using the master branch on github, commit 4df38f34, if you want to accompany some parts.

## Structure

This document will be divided into a few sections

1. Preliminary knowledge
2. An overview of how the combat schedule works
    1. How to enter combat
    2. What happens every "turn"
    3. Differences between melee and ranged weapons
3. Hit probabilities
4. Damage calculations
5. Experience calculations
6. Final notes
7. Appendices
    1. What stat to invest in?
    2. How to compile exult and modify the code?
    3. Other schedules
    4. Alignment

## Preliminary knowledge and brief overview

In U7, the people and monsters out and about are `Actor`s, and every one has a schedule, even the
Avatar! It dictates what they'll do and where they'll go and schedules are, in great part, what
makes U7 special to so many. In essence, it's a form of artificial intelligence. From now on, I'll
use the terms Actors and npcs interchangeably.

The simplest schedule is `wait`, which does **nothing**, followed by `loiter` which means wander
around aimlessly. Even the simplest schedule has a function called `now_what`, which tells the npc
what they'll do once their current task is finished.

The schedule of interest of this document is appropriately called `Combat_schedule`. One of the
first things that happens in `Combat_schedule::now_what` is to look for a foe. An npc is considered
a foe if there's an *alignment clash*.

`Alignment` refers to an npc's stance towards the Avatar. There are 4 alignments, starting from 0.
First, `neutral`, comprising most of the named npcs around (Even LB!), then there's `good`, which
is *mostly* the Avatar and its current party, with a few exceptions. Then there's `evil`, which is
hostile to `neutral` and `good`, and `chaotic`, which is hostile to all. These can be temporarily
modified by being charmed (and the `charmed_more_difficult` game option). Even if you attack a
peaceful npc, yours or their alignment won't change.

During combat, you attack with melee weapons, thrown weapons (good or bad), or with magic. These all
have a base damage, powers and damage types. Hit probability and damage calculations are modified by
the attacker's and defender's stats (STR, DEX, INT, COMBAT). Unless usecode shenanigans are at play,
everyone is considered the same way!

After defeating an enemy, you gain experience points and, if you kill the enemy, access to (part of)
its inventory. Combat continues until all enemies around are dead or unconscious.

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

## Overview of how the combat schedule works

### How to enter combat

The simplest way of entering combat is by toggling the combat state either in the ragdoll screen or
pressing 'C'. This iterates through all party members and sets their schedule from
`Follow_avatar` to `Combat`. In this state, everyone in the party will start to look for enemies and
approach them. But not before! You can calmly walk around and talk to npcs with hostile alignments
provided none of you enter the combat state. The instant you press 'C' however, you'll attack. For
example, Iriale Silvermist, located in the Fellowship retreat, is Evil but peaceful (for some time),
as well as some npcs in New Magincia (Robin, Battles and Leavell).

An npc being attacked is another way of entering combat. An `Actor::fight_back` function is called.
First, it checks if someone in the party is being attacked and the Avatar can't act
(paralyzed, asleep, knocked out or dead), and if so, sets everyone to fight. If the Avatar was being
a bully and attacking a peaceful npc that can react back or someone saw or heard the attack, between
1 and 3 guards will be summoned from offscreen to fight/arrest the Avatar and up to 3 sentient
npcs (INT >= 6) in the vicinity will be set to attack the Avatar. Funnily, their alignment is *
chaotic* and their schedule is set to talk to the Avatar.

From what I've found, enemies created from eggs, summon, cloned (slimes) or manually placed are
either in the `Combat` or `Patrol` schedules. In the `Patrol` schedule, and its derived schedules,
if the npc is set to look for combat and it sees another npc with an alignment clash, it will enter
the combat state. Npcs have a counter to determine how many times they've failed to find an
opponent. After having failed, if they had a previous schedule and their alignment is good, they'll
return to their previous schedule (TODO: re-verify when to give up and explore other spawn
situations). Others just keep trying and trying to find an enemy until you walk far away enough from
them.

### Combat state was created

When a combat schedule is created, a few standard values are added (`combat.cc:1262`). The npc
checks if it has any weapon (or spellbook) readied and, if not, tries to ready the best weapon and
shield it can find in its possession. If it couldn't find any, the npc tries to protect itself with
the best shield. If anything couldn't be found, the npc resigns itself to fight with their hands.

The best weapon is found by iterating through all the items in its inventory that aren't inside
locked containers, that can be equipped in the hands, and the finding the one with
the `base_strength`. If it's a ranged weapon, it considers the ones with the longest range and the
best ammo found in the inventory. I'll describe weapons more elsewhere, but suffice to say that the
strength of a weapon is its base damage and weapon powers and damage types provide bonuses. The best
ammo depends on its base strength (same as weapons) and how much ammo you have left.

Right after creating any schedule, its `now_what` function is called. In the case of
`Combat_schedule::now_what`, this function looks through the current state of combat of this
specific npc and decides what action to perform. After this event is handled, this function is
called again. If this game was a more traditional turn-based game, every npc would call
its `Combat_schedule::now_what` function in sequence. However, actions, like moving, striking, take
varying amounts of time, so a set order isn't always followed. Nevertheless, I will call this a
"turn", to simplify the nomenclature.

### The combat loop

What actions are performed in each turn depend on the state of the combat. These states are defined
in `combat.h:36`, and make the combat schedule a finite state machine. These are:

* `initial`: When the schedule was just constructed
* `approach`: When the npc is approaching an enemy to attack
* `retreat`: When the npc is moving away from an enemy (e.g. get to a better range)
* `flee`: When the npc is fleeing from combat
* `strike`: When the npc is striking the enemy with their melee weapon (TODO: thrown uses this?)
* `fire`: When the npc is firing their ranged weapon
* `parry`: Should be used when parrying, but this isn't used anywhere in the code base.
* `stunned`: When the npc was just hit
* `wait_return`: When the npc is waiting for their ranged weapon to return (boomerangs, magic axes)

At the start if `what_now`, the function does some minor tests. It sets the state to `approach`
if the schedule was just created, makes the npc wait for 1 second before calling again if it's
sleeping, makes the npc run away if it's attack mode is `flee`, or resumes following the Avatar if
you exited from combat after some time, and checks if the npc needs a new target. For example, it
could have died or turned invisible.

The running away function just finds some random position and with a 3/4 chance, screams loudly, and
1/4 chance just yells some standard fleeing scream. No mention here of dropping equipment (a
aggravating feature of the original).

And finally we reach a branching point, where the actor will decide on its course of action
depending on its state. The sequence that follows if roughly this:

Tries to find and approach an enemy. If the npc is close enough, its state is changed to `strike`
or `fire` and next time `now_what` is called, it will process the hit and damage calculations. Right
as `strike` or `fire` are processed, the state goes back to `approach`. The exception is for thrown
weapons such as a boomerang, where the state is changed to `wait_return`, then reverts to `approach`
. However, before any attack can happen, actors need to build up "initiative" (`dex_points`), as
I'll discuss in the next section.

#### Building up initiative and the `approach` state

If the schedule was just created, an enemy died/turned invisible, we need to approach
it (`combat.cc:616 approach_foe`). If there's no opponents, it tries to find a foe
`combat.cc:445 find_foe`. And `find_foe` calls `find_opponents`. It's quite a rabbit hole for a
seeminly simple function.

In summary, npcs keep and regularly update a list of possible opponents. As the enemies are
defeated, they are removed from the list. After all are defeated, npcs consider unconscious enemies
to finish them off. One thing that greatly increases the complexity of these functions is alignment
changes caused by being charmed. In the original, if the Avatar was charmed, you could still control
him/her despite the status, you could access any charmed party member's inventory, and they would
leave combat if you did. Exult introduced an option called `charmed_more_difficult` where this is
fixed. Charmed party members won't obey you, will continue to attack and you won't be able to access
their inventory, including the Avatar. Upon alignment changes, the list of opponents is reset. Also,
invisible enemies are ignored if the npc can't see invisible. The range is 9 chunks around the npc.
If you have `combat_trace` on, you'll see a bunch of `X pushed back (number) y`. Here, `pushed back`
means "added to the end of the list".

After having found a list of possible opponents, which one that's selected depends on the npc's
attack mode. I personally always left all party members in `nearest` and just mowed through all
resistance. Here's what the attack modes do in Exult.

* *weakest* targets the opponent with the smallest *strength* status.
* *strongest* targets the opponent with the largest *strength* status.
* *nearest* finds the closest opponent and avoids fleeing enemies.
* *protect* finds the attackers of a protected party member. Apparently, if there's more than one
  protected party member, the more recently added ones receive priority (so only leave one
  protected). From these attackers, gets the closest one to the protected party member. If no one's
  protected, a random one is chosen. Also, the npc will yell they'll protect their friend (50%
  chance).
* *random* and every other attack mode: Chooses the first one in the list of opponents, which I
  guess is random enough.

If no enemy is found, a `failures` counter is increased and a 24 second pause is added before
searching for new enemies. Otherwise, the npc starts trying to path to its opponent, and if it can
teleport close to the target, there's a 1/4 chance of doing so. Before that though, a monstruous
chain of boolean operations is made to check if the npc should run away. Let's break it down:

* If it isn't a monster and, if it is, it can die (??) AND
    * The attack mode is flee OR
        * the attack mode isn't berserk AND
            * the enemy can move (?) AND it isn't the main actor (Avatar) and the npc's health is
              less than 3

Only if all these conditions are met, the npc runs away. Phew!

Now we try to path to the opponent, combat music starts playing, `combat_trace` prints out "X is
pursuing Y", yells something with 50% chance and sets the npc to approach the opponent, stopping
when its weapon is close enough to hit. If the npc couldn't path to the opponent, it tries to path
to the closest opponent. If it still couldn't, makes the npc walk randomly vaguely towards the
opponent and then tries again in the next cycle. A failure count is added in this case.

Right now, the actor is either approaching or is close enough to its opponent that it can attack.
However, before performing an aggressive action (strike, fire, summon, turn invisible), the actor
needs to "build up initiative", which is tracked by the variable `dex_points`. An
actor's `dex_points` starts at 0. The npc's dexterity stat is added to `dex_points` until a
hardcoded value of 30 (`dex_to_attack`) is reached. Before that, no attacks can occur. If an attack
goes through, 30 is removed from the accumulated dex, so any excess value carries over to the next "
turn".

In other words, dexterity controls how quickly an actor can attack. A dex value of 30 means they can
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

#### Starting a strike

Now that the npc has finally built its dexterity, it can perform an offensive action. If the npc can
turn invisible and it isn't invisible, there's a INT/300 chance of it turning invisible (so 30 int
means 10% of turning; 15 INT means 5% of turning). Then, the npc checks if it can summon, and the
chance is INT/600, so half of invis. In each case, 30 is removed from `dex_points`. If none of these
conditions is met (most common situation), the function `start_strike` is called (`combat.cc:820`).

This function gets the weapon/monster reach and determined if the npc is too far away. If it is,
tries to approach it and exits. Then it checks if it's using a ranged weapon or similar and if it's
out of charges/ammo/reagents. If so, tries to swap for its next best weapon and the npc goes back to
approaching. Otherwise, its state changes to `fire`. Otherwise, it's a melee weapon (that doesn't
have those problems), so the state changed to `strike`.

Then the npc tries to check if there's a straight line of fire between the npc to its opponent. If
not, reverts to the approach state.

If none of those pesky conditions changed the npc's state, there's a 1/20 chance of yelling a taunt
and finally `dex_points` is decremented.

We're now back to `now_what` with the state either `strike` or `fire`. Both cases work similarly.
First, they revert the state back to `approach` (so the dex buildup restarts), then set up some
frame information (funny note, slimes and sea serpents are considered to have "strange movement", so
they receive special treatment here) and finally the target is attacked by calling the function
`attack_target` (`combat.cc:929`).

Again, this function does some checking for dead combatants, lack of ammo, insufficient range, etc.
Then, if the weapon uses charges and there's charges to be used, uses up the charges and, if it's
depleted, deals with it, and requests a new weapon. If it's a ranged weapon, decrements the ammo
available and if it was a thrown weapon, tries to ready a new one or waits for it to return (
boomerang).

Now here's the interesting part, where the function calculates if the attack goes through or not. A
comparison is made between the (adjusted) attacker's combat stat and the base defender's combat
stat. The attacker's combat is increased by 3 if the weapon or ammo has the lucky property
(no such benefit for the defender). Then, bias is applied to the attack stat.

Bias is something introduced by Exult, and it's known as Combat difficulty. This ranged from -3 (
easiest)
to +3 (hardest) in steps of 1, 0 being default. Twice the difficulty is either added or subtracted
from the attacker's combat stat (for a max of +/- 6). In the easier settings, party members receive
a boost in attack and hostiles receive a penalty in attack. In the harder settings, the opposite
happens. I'll show later how this affects the hit probabilities.

Ranged and melee weapons differ slightly here. Ranged/thrown weapons provide a bonus of 6 to the
attack value. Then, if it's a poor thrown weapon (think improvised weapons), attack value is reduced
by the distance. A good thrown weapon (think more specialized weapons), the penalty is half the
distance. I guess these exist to reflect that it's harder for the defender to protect against a fast
moving arrow, but it's easier to deflect slower moving weapon.

Then, a new projectile effect is created, which carries the attacker, target, weapon, projectile and
attack value. The comments say this is to prevent the attacker from having its combat lowered during
the projectile flight (infinitesimal chance I think, but they probably have a good reason). The
attack itself is processed somewhere else entirely,
in `effects.cc:697 Projectile_effect::handle_event`. This function deals with the details of missile
attacks, like rotation, speed, if the ammo drops, if an explosion should occur, if it's homing, if
the projectile returns (boomerang), if the target moved/teleported away, and, of special interest it
calls `try_to_hit`. We'll deal with it together with melee weapons.

Going back to `Combat_schedule::attack_target`. The case for ranged weapons always returns `true`,
meaning the game thinks it *hit*, but the npc just fired the weapon. Melee weapons are a bit
simpler. The function checks if the weapon has the `autohit` property and if the game is in god
mode. In god mode, you always hit the targets, but they never hit you, and this overrides `autohit`.
Then, `try_to_hit`
is called. The current function deals with some minor stuff and returns `true` if `try_to_hit` was
true, otherwise returns `false`.

#### Trying to hit

We're now in another file entirely, since `try_to_hit` depends on what type of entity the npc is
attacking. In this case, it's actors, so we're specifically in `actors.cc:3976 Actor::try_to_hit`.
This function handles the defense value of the defender, `defval` and compares it to the attacker's
attack value (after all those adjustments). `defval` is simply the defender's combat stat. If it's
under the *protection* spell, its defense is increased by 3 which is, all things considered, quite
good. Then, this function calls `actors.cc:3773 Actor::roll_to_win`.

`roll_to_win` rolls a 30 sided die (technically a die from 0 to 29, but I'll treat it as 1-30). If
it's a "critical miss" (1), then the attack missed. If it's a "critical hit" (30), the attack hit.
For intermediary values, the roll + attack - defense has to greater or equal to half the number of
sides minus 1 (14). In terms of probability, there's always a 3.3% (1/30) chance of hitting, a
3.3% (1/30) chance of missing and a `(15 + defense - attack + 1)/30` chance of hitting.

This means for combatants with equal combat stats, the attacker is always *slightly* biased towards
hitting. To better visualize how this works, and to show how bias affects the probabilities, I've
prepared some hit probability maps containing simulations and theoretical values of hit
probabilities, from game difficulty -3 to +3. These consider you're attacking (i.e. party member) 
a random monster. 

![Difficulty 0](/assets/img/exult_study/hit_probability_map_dif0.png)

| ![Difficulty -3](/assets/img/exult_study/hit_probability_map_dif-3.png) | ![Difficulty -2](/assets/img/exult_study/hit_probability_map_dif-2.png) | ![Difficulty -1](/assets/img/exult_study/hit_probability_map_dif-1.png) |
|-------------------------------------------------------------------------|-------------------------------------------------------------------------|-------------------------------------------------------------------------|
| ![Difficulty +3](/assets/img/exult_study/hit_probability_map_dif3.png) | ![Difficulty +2](/assets/img/exult_study/hit_probability_map_dif2.png)  | ![Difficulty 1](/assets/img/exult_study/hit_probability_map_dif1.png)   |

We can see when difficult is 0, the 50% hit probability is slightly offset downwards, indicating the
green-yellowish area (>50% hit prob) is bigger. Then, as you go from -3 to +3 (easy to hard), the
yellow region shrinks considerably. Since the bias is symmetrical, if you want to know how probably
it is for a monster to attack your party, you just need to consider the opposite sign graphs (i.e. a
monster to-hit map at the hardest difficulty is the -3 map, and so on.)

Note the combat stats are integers (there's no 14.5 attack), so these are not continuous probabilities.
Yet, I found these maps pretty.




### Differences between weapons types and magic

Magic works different between enemies and the Avatar, who requires a spellbook. Spells are actually
items they can equip and "throw" at you. This makes spellcasters especially dangerous. When killed,
these items are removed from them so you can't access them natually. However, I distinctly remember
a few jesters around the castle of the white dragon that spawned with death bolts or something like
that, and which I could loot and use for a bit. However, these items have a quality level, which
specifies their number of uses, such as wands. For example, Aram-dol has two spells equipped, one
with 99 uses and another with 2. Good luck surviving 99 casts of some spell.

![Mage inventory](/assets/img/exult_study/exult_mage_items.PNG)

-----------

## Hit probabilities

## Damage calculations

--------------------

That's why you don't find liches or mages, for instance, with reagents and spellbooks. When they
die, the "magic"
weapons are removed.

When receiving damage, an npc's health is reduced. If it gets to zero or below, it's knocked
unconscious. In this state, its dex is reduced to 0 (meaning it can't attack) and it is given less
priority than conscious opponents.

## Experience calculations

## Final notes

## Appendices

### What stats to invest in?

### How to compile exult myself and modify the code?

### Other schedules

In U7, Actors have schedules. At least, it's set to `loiter`. Available schedules are defined
in `schedule.h`, in an enum called `Schedule_types`. They are: `combat`, `horiz_pace`, `vert_pace`
, `talk`, `dance`, `eat`, `farm`, `tend_shop`, `miner`, `hound`, `stand`, `loiter`, `wander`
, `blacksmith`, `sleep`, `wait`, `sit`, `graze`, `bake`, `sew`, `shy`, `lab`, `thief`, `waiter`
, `special`, `kid_games`, `eat_at_inn`, `duel`, `preach`, `patrol`, `desk_work`, `follow_avatar`.
There a few more defined by exult: `walk_to_schedule`, `street_maintenance`, `arrest_avatar`
, `first_scripted_schedule`.

### Actor alignment musings

If you manage to set your own schedule to Evil, many shenanigans can happen.

### A brief description of weapons

Weapons

---------------------

## Overview of schedules

## NPC alignments

In BG, as soon as you start the game, only you and Iolo of the companions are good. Spark, Shamino,
Dupre, even Lord British, are neutral.

Sullivan, a prisoner of the fellowship in Buc's den, and Leonardo, a dog i Serpent's hold, is good.

Evil is "Juggernaut", a fighter in IOTA, Kali, a dragon is a cave in IOTA, Stone Harpy in Spektran,
some enemies at that base in the Serpent's spine, and of course Elizabeth, Abraham and crew. Batlin
has many "faces", and only "Batlin_monster" is evil. The others are neutral.

Chaotic is Goth, a guard in Yew,

Evil is Iriale Silvermist, in the caves in the Fellowship retreat, Robin, Battles, Leavell in New
Magincia.

In SI, a cursory look using the cheat menu reveals that the *vast* majority of NPCs are neutral.

The good ones are the ones in the party, Stephano, Dupre, Iolo, Shamino.

Evil ones are enemies like some Gwani (?), Goblins, prisoners, Perry Stokes (It's the software
pirate in the hut in the woods), billy_cain (goblin), steve_powers (goblin), chuck (goblin),
key_guy (troll), Anti Dupre (but not Anti-Iolo??? or Anti-Shamino???), Goblin king and a few
antagonists, like the

Chaotics are Rabindrinath, Crusty (a jester in a hut in the goblin woods) and perhaps some other
unnamed enemies

Shoutout to the neutral nightmare in the dream world, SmithzHorse.

## Damage calculations

Quality dictates weapon charges, like wands.

* Weapon have a few types of powers:
    * sleep, charm, curse, poison, paralyze, magebane, unknown, no damage (puts Dragan to sleep,
      see [here](https://ultima.fandom.com/wiki/Draygan))
    * These appear to be a byte. Maybe `weapons.dat` is read like this?
  > 00000000
  > ||||||||
  > |||||||sleep
  > ||||||charm
  > |||||curse
  > ||||poison
  > |||paralyze
  > ||magebane
  > |unknown
  > draygan sleep, King's Savior flower
* There's 6 types of damage.
    * normal (0), fire(1), magic (2), lightning (3), ethereal (4), sonic (5)

### From combat_ops.h - DONE

* There's checks for paused, checks for difficulty (0 = normal, >0 = harder, <0 = easier). These can
  be accessed from the menu!
* There's two combat modes, original and keypause, with the latter using space as suspend/resume
  combat. This really lets you direct attacks from specific party members to specific targets.
* There's an option to show hit numbers? -- yes! That's actually an option in the menus.
* What's charmed more difficult? - According to the manual, does this:
    * In normal, Avatar can't be charmed. In the original, he could, but nothing would be changed.
      In hard, charmed party members can't have their inventory accessed, and will continue to
      attack even if combat is stopped.

## Conclusions with code snippets

### Get knocked out if health <= 0

`actors.h (:487)`

```c++
bool is_knocked_out() const {
	return get_property(static_cast<int>(health)) <= 0;
}
```

### Dies if health < -1/3 str

`actors.h (:483)`

```c++
	bool is_dying() const {     // Dead when health below -1/3 str.
		return properties[static_cast<int>(health)] <
		       -(properties[static_cast<int>(strength)] / 3);
	}
```

### Finds best ammo based on its strength *and* remaining amount.

* Find best ammo:
    1. Starts with a value of -20 for strength.
    2. Considers all objects (maximum of 50).
    3. For every object:
    1. Is it inside a locked chest? If yes, next.
    2. Is it of the wrong ammo family? If yes, next
    3. Finds ammo info, and if it's not available. Didn't understand.
    4. Is it enough? (Triple crossbow?)
    5. Calculates the strength with `ammo_info->get_base_strength`
    6. Strength is balanced if there's little of the ammo (if there's less than 5 times what's
       needed). If so, strength is divided by 3. If it's less than 10 times what's needed, strength
       divided by 2.
    7. Goes through everything until the best ammo is found.

```c++
Game_object *Actor::find_best_ammo(
    int family,
    int needed
) {
	Game_object *best = nullptr;
	int best_strength = -20;
	Game_object_vector vec;     // Get list of all possessions.
	vec.reserve(50);
	get_objects(vec, c_any_shapenum, c_any_qual, c_any_framenum);
	for (auto *obj : vec) {
		if (obj->inside_locked() || !In_ammo_family(obj->get_shapenum(), family))
			continue;
		const Ammo_info *ainf = obj->get_info().get_ammo_info();
		if (!ainf)  // E.g., musket ammunition doesn't have it.
			continue;
		// Can't use it.
		if (obj->get_quantity() < needed)
			continue;
		// Calc ammo strength.
		int strength = ainf->get_base_strength();
		// Favor those with more shots remaining.
		if (obj->get_quantity() < 5 * needed)
			strength /= 3;
		else if (obj->get_quantity() < 10 * needed)
			strength /= 2;
		if (strength > best_strength) {
			best = obj;
			best_strength = strength;
		}
	}
	return best;
}


```

### The effective range is either melee or the missile weapon's own range. If it's a poor or good throwable, the distance is affected by the quality and attacker's stats, limited to 31.

1. Starts with weapon info and reach (int).
2. If it's less than 0 (i.e., -1), it uses the weapon's reach. If there's no weapon info, uses
   monster's specific reach or the default reach.
3. If (not uses), which means, if hand-hand or ranged, returns the weapon's reach.

* Uses is, from `weaponinfo.h`:
    * 0 if hand-hand, 1 of poor throwable, 2 if good throwable, 3 if missile firing

4. If either a poor or good throwable, gets the strength and combat stats of the actor. Whichever is
   higher is the current range. If it's a good thrown weapon, the range is multiplied by 2. If it's
   lower than the reach, then it's replaced by reach, and if it's greater than 31 (units?), it's cut
   to 31.

```c++
/**
 *  Get effective maximum range for weapon taking in consideration
 *  the actor's strength and combat.
 *  @param winf Pointer to weapon information of the current weapon,
 *  or null for no weapon.
 *  @param reach Weapon reach, or -1 to use weapon's.
 *  @return Weapon's effective range.
 */
int Actor::get_effective_range(
    const Weapon_info *winf,
    int reach
) const {
	if (reach < 0) {
		if (!winf) {
			const Monster_info *minf = get_info().get_monster_info();
			return minf ? minf->get_reach()
			       : Monster_info::get_default()->get_reach();
		}
		reach = winf->get_range();
	}
	int uses = winf ? winf->get_uses() : static_cast<int>(Weapon_info::melee);
	if (!uses || uses == Weapon_info::ranged)
		return reach;
	else {
		int eff_range;
		int str = get_effective_prop(static_cast<int>(Actor::strength));
		int combat = get_effective_prop(static_cast<int>(Actor::combat));
		if (str < combat)
			eff_range = str;
		else
			eff_range = combat;
		if (uses == Weapon_info::good_thrown)
			eff_range *= 2;
		if (eff_range < reach)
			eff_range = reach;
		if (eff_range > 31)
			eff_range = 31;
		return eff_range;
	}
}

```

## Some time calculations with dex to attack.

Put Avatar in the Trinsic stables and ordered him to attack the horse. FPS=4. Lightning whip.

See excel spreadsheet

* Dex = 0, fps=4: NEver attacked
* Dex = 1, fps=4: 8 seconds

* Dex = 1, fps=2: 16 seconds (15183 ticks)
* Dex = 1, fps=2: 903840 - 888492 = 15384 ticks
* Dex = 2, fps=2: 1025047 - 1017279 = 7768 ticks
* Dex = 3, fps=2: 1119036 - 1113778 = 5258 ticks
* Dex = 3, fps=2: 1196321 - 1191057 = 5264 ticks
* Dex = 5, fps=2: 1268251 - 1265015 = 3236 ticks
* Dex = 10, fps=2: 101680 - 99964 = 1716 ticks
* Dex = 15, fps=2: 161137 - 162341 = 1204 ticks
* Dex = 15, fps=2: 345291 - 344077 = 1214 ticks
* Dex = 15, fps=2: 536924 - 535713 =
* Dex = 20, fps=2: 234732 - 233505 = 1227 ticks
* Dex = 20, fps=2: 267006 - 265783 = 1223 ticks
