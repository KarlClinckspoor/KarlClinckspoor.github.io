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

TODO: Check all notes in the text.

TODO: Make a flowchart

TODO: Review the entire text and rewrite entire section so they follow the code less, but more of
        the logic

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
* I've decided to forego a bit the "blow-by-blow" description of what each function does to a more
  general approach, to improve the readability of this text.
* I'm using the master branch on github, commit 4df38f34, if you want to accompany some parts.

## TLDR

* Combat is a type of Schedule, which controls every action an npc performs. Other schedules include
  baking, serving food, etc. Aggressive actions do not happen outside of combat (save for usecode).
* To enter combat, the npc needs
  * Spawn in the combat schedule
  * Be in a patrol schedule and find an NPC with a clashing alignment
  * Be in the Avatar's party and have the UI switch to combat mode
  * Be attacked
* There are 4 npc alignments: neutral (0), good (1), evil (2), chaotic (3). Evil is aggressive to
  neutral and good, chaotic is aggressive to all, good is aggressive to evil. Good is mostly the Avatar's
  party. Evil are mostly enemies, but some "peaceful" npc also spawn as evil.
* Every schedule alternates between animating an `action` and determining what action to do in
  a `now_what`.
* The combat schedule is a finite state machine with the states `initial`, `approach`, `strike`,
  `fire`, `wait_return`.
* Every cycle in the `approach` state (which I'll call a "turn"), an internal counter
  called `dex_points` increases based on the npc's dex attribute. When this reaches `dex_to_attack`,
  which is 30, it can change its state from `approach` to `strike` or `fire`. The difference between
  the latter 2 is small, mostly having to deal with creating a projectile or striking directly.
* To determine if an attack hits, the attacker's combat attribute `attval` is compared to the
  defender's `defval`. 
  * `attval` is altered by the game difficulty (up to +/- 6 combat), the weapon type (good thrown,
      poor thrown, ranged) and the range in the case of thrown weapons.
  * `defval` is only affected by the Protection spell status, which
    increases `defval` by 3.
  * the probability of hitting is `(15 + defval - attval + 1)/30`. However, there's always a 1/30
    chance of hitting and a 1/30 chance of missing.
* When an attack hits, the hit points lost and other effects are calculated.
  * Attacks can have damage types (normal, fire, magic, lightning, ethereal, sonic) and powers (sleep,
      charm, curse, poison, paralyze, magebane and two extra).
  * If the weapon's damage is 127, it's an instakill and no further checks are performed.
  * The weapon/monster's attack value is the base damage.
  * Game difficulty adds/removes damage depending on the bias.
  * A random value from 1 up to a third of the attacker's strength is added to the base damage.
  * A random value from 1 up to the weapon's attack rating is added to the base damage, except
    lightning weapons.
  * An npc's base armor value is summed with every armor item in its inventory to give a final armor value.
  * A number between 1 and the total armor value is removed from the base damage.
  * Lightning, ethereal and sonic damage ignore armor.
  * If the npc is immune to that specific damage type, damage is set to zero. If the monster can't die,
    the same is made.
  * If the npc is vulnerable to that specific damage type, damage is multiplied by 2.
* Health is considered in the following manner:
  * Max health is equal to strength.
  * If health falls to or below 0, the npc is knocked unconscious.
  * If the health falls below 1/3 of its max health, it dies.
  * Some npc's have a `tournament flag` on, which hands off health reduction calculations to the usecode
    machine, meaning they have special treatment.
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
    strength (Poison, paralyze) or intelligence (curse, charm, sleep)
  * Magebane always works. It sets the target's mana to zero and removes all spell "weapons" from the npc.
* Combat ends when:
  * The UI is changed.
  * All enemies are dead and the npc failed to find an enemy several times.
  * The npc despawns/becomes dormant.

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

{%include exult_study/plotly_figs.html %}

We can see when difficult is 0, the 50% hit probability is slightly offset downwards, indicating the
green-yellowish area (>50% hit prob) is bigger. Then, as you go from -3 to +3 (easy to hard), the
yellow region shrinks considerably. Since the bias is symmetrical, if you want to know how probably
it is for a monster to attack your party, you just need to consider the opposite sign graphs (i.e. a
monster to-hit map at the hardest difficulty is the -3 map, and so on.)

Going back to `attack_target`, we finally reach the part where damage is calculated. The target actor
has a function called `attacked` which receives who attacked, the weapon used, the ammo and if it's
an explosion attack. Let's see how this works.

#### Dealing damage

The function `actors.cc:4005 attacked` is quite simple. It does some housekeeping with checks and
setting oppressors and if the `show_hits` option or `combat_trace` cheat are enabled, prints out the
appropriate messages. Of interest, it calls `actors.cc:3805 figure_hit_points`, which calculates
how many hit points to remove *and* how many xp points are received.

First, it checks if the god mode cheat is activated. If so, damage received by party is always 0, and
damage dealt is always max (127). The combat difficulty comes back again and the bias is now: if 
the attacked is in the party, it's the bias, if the attacker is in the party, is the negative of the bias,
and enemies fighting each other isn't affected by bias. The weapon base damage, damage type (normal,
fire, magic, lightning, ethereal, sonic), weapon powers (sleep, charm, curse, poison, paralyze,
magebane, unknown and no_damage) and if it explodes if stored, and the same with ammo. The ammo damage
is added to the ranged weapon damage, the powers and explosion effects are also added. The damage
type replaces the weapon's only if the ammo has normal damage. Otherwise, the weapon's damage type
is transferred to the ammo. Then the function calculates how many hp should be lost, then applies
the checks for weapon powers.

Damage is applied by `actors.cc:2628 apply_damage`. Base damage depends on damage type, bias and the
attacker's strength. Base damage is a value between 1 and a third of the attacker's strength plus a
value between 1 and the attack's base damage (weapon + ammo). If it's lightning damage, there's no
strength bonus, but that's compensated by it ignoring armor.

Bias either removes or adds base damage at 1 damage per difficulty level. It's a bit harder to
explain by text, so here's a table.

| Difficulty/Damage dealt... | To party | By party |
|----------------------------|----------|----------|
| Easier                     | Removes  | Adds     |
| Harder                     | Adds     | Removes  |

Then, we consider armor. All the npc's worn armor base defense is summed, and added to the npc's
base armor value (for monsters). If any armor has immunity to the attack's damage type, damage is
set to zero, (I guess this means even a godmode avatar can't deal damage to enemies immune to it's
weapon damage) a metal clang is played and the npc `fight_back` (as I mentioned, a way of entering
the combat state). However, if the attack type of lightning, ethereal, sonic or it's supposed to be
a 1hit kill attack (glass sword), armor is set to zero. Bias also affects this just like before, 1
extra/less armor per difficulty. 

| Difficulty/Armor of entity ...   | In party | Outside party |
|----------------------------------|----------|---------------|
| Easier                           | Adds     | Removes       |
| Harder                           | Removes  | Adds          |

After this calculation, the final damage value is calculated. A random number between 1 and the
total armor is removed from the damage calculated so far. This means there's no difference between
armor piece location, meaning you can have your head exposed all the time if you want. If the npc is
paralyzed or unconscious and the damage was less than or equal to zero, damage received receives a
bonus between 1 and the attacker's strength.

After all these considerations, if damage was less or equal to zero, plays that a metal clang sound,
but flashes the attacked's outline in red, and makes it fight back. And finally, we reach the point
where health is reduced, in `actors.cc:2721 reduce_health`. Don't forget about exp, since that's
being carried over for quite a while now.

Here in `reduce_health`, we check if the npc is already dead or if there's god mode on, and prevents
any health reduction. Then, if the monster can't die or is immune to the attack type, makes it
attack back and deals 0 damage. If the monster is vulnerable to the attack type, damage is
multiplied by 2.

The game then calculates how much the npc's health would drop by subtracting the current hp by
the damage (limited to -50 final health). There's some shenanigans with enemies with the tournament flag.
From what I've searched, in SI there's a lot of npcs with this flag, meaning it's up to the usecode
machine to allow or not the npc to be receive damage/die. The comments say 'no more pushover banes',
which I have no idea what it means. Maybe some day I'll take a dive into the usecode files, but not now.

Then the function checks if the npc can split (slimes). There's a 50% chance of splitting provided the npc
wasn't vulnerable to the attack type.

Finally the npc's health is reduced to the calculated value, and we check if the npc should die or
if it was defeated (health goes below or equal to 0, and gets knocked out). An npc dies if its
current health drops below a third of max health (which is equal to strength). After this, if the
enemy was defeated, the exp value is calculated.

#### Calculating EXP

First, it checks if the npc wasn't unconscious when attacked. If it was, exp is **zero**, so don't
go around murdering people in their sleep.

Exp is calculated by adding several small contributions to a pool. First, exp is equal to the
opponent's combat value, its strength, its (dex+1) / 3 and its intelligence / 5. Then, if it's a
monster, it has a base xp value, which gets added to

1. Combat stat
2. Strength
3. (Dex+1) / 3
4. Intelligence / 5
5. If monster, its base exp.
6. Goes through all the npc's possessions and sums the armor protection values, the weapon's base xp
   to the pool and a small bonus depending on the weapon type.
   1. Melee weapons with range greater than 5, bonus exp is 2, if it's 4 or 5, bonus exp is 1, else it's 0. 
   2. For poor thrown weapons, it's a fifth of the npc's combat value.
   3. For good thrown weapons, it's a third of the npc's combat value.
   4. If it's ranged, it's half the weapon's range.
7. Exult does something slightly different here, and adds/removes 1 point for each vulnerability (-)
   or immunity (+).

Last, this exp pool is divided by 2, and that's the final exp value received by the defeat.

#### Applying weapon powers

After the damage was applied the weapon powers get applied. If the npc is protected 
with the Protection spell, it's absolutely immune to any effects like poison, curse, charm, etc. I
found that super interesting as I always thought this spell was pretty useless. If the game had a more
tactical combat, I'm certain this spell would be used much more.

These powers get applied by rolling the same 30 sided die for combat, but uses different stats. Only
the attacker's intelligence is considered (if it doesn't have one, like a trap, it's set to 16).

| Effect   | Attacker's attribute | Defender's attribute |
|----------|----------------------|----------------------|
| Poison   | Int                  | Str                  |
| Curse    | Int                  | Int                  |
| Charm    | Int                  | Int                  |
| Sleep    | Int                  | Int                  |
| Paralyze | Int                  | Str                  |
| Magebane | -                    | -                    |

Magebane always affects the victim. It acts by setting its mana to zero and collecting all the
spells in the npc's inventory and removes them. In case you're confused by this, enemies don't
require a spellbook to cast spells. Rather, they have spells are weapons they can fire a limited
number of at enemies. If you manage to get one, you can use it too. This makes spellcasters
especially dangerous. When killed, these items are removed from them so you can't access them
natually. However, I distinctly remember a few jesters around the castle of the white dragon that
spawned with death bolts or something like that, and which I looted and used for a bit. For example,
Aram-dol has two spells equipped, one with 99 uses and another with 2. Good luck surviving 99 casts
of some spell. If you have Exult Studio equipped, you can open an npc's inventory gump and see the
spells. Here's an example of a mage located at an island to the south of Trinsic.

![Mage inventory](/assets/img/exult_study/exult_mage_items.PNG)

Last, this function calls the weapon's usecode functions, such as the weird `no_damage` power and any
usecode functions specific for that weapon. 

The attacker also receives the exp value at this point. If you're curious, your level is calculated
as
`1 + log2(total exp / 50)` (see `actors.h:490`). If you want to go the opposite way, you can
use `25 * 2 ^ level`. Every level gives the npc 3 training points. I prepared two graphs with this info:

![Experience and level](/assets/img/exult_study/Experience.png)

![Experience and level, log scale](/assets/img/exult_study/Experience_log.png)

#### Wrapping up and exiting combat

After dealing damage and getting the exp, the stack of function start returning one after the other
and we're back in `Combat_schedule::now_what`. The npc struck the target, its `dex_points` got reset by 30,
its state reverted back to `approach` so it can build up `dex_points` again.

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
