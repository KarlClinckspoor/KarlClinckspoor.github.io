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

TODO: Add some traces to double check if it's only party members that give up combat and return
to the previous schedule

TODO: Check if npc spells have a set number of uses. ExultStudio can be useful for this.

TODO: Check all notes in the text.

DONE: Check if random mages spawn with spells equipped.

DONE: Do more testing here and apply some prints to random enemies spawned, e.g, from eggs. Do
they spawn in patrol?
- No! Many normal enemies spawn in combat already. They never left the combat state and kept
  failing repeatedly. Still need to dive a bit deeper to see why they aren't giving up.
- Some humanoids did spawn in patrol though (pirate and mages)


I decided to peek into the source code of [Exult](https://github.com/exult/exult) and figure out 
how combat works. I really enjoy watching/reading deep dives into game mechanics and thought, 
"well, why couldn't I do one, and of one of my favorite series?". Here's the result of a few 
hours of code reading, note writing, a bit of source code manipulation and running some 
simulations. I hope this brings you a bit of satisfaction, and I'm certain you'll read some 
parts and think "Oh, so that's why this happens!"

A few disclaimers before we start: 

* The combat in exult does not work exactly like in the originals. It's 
simplified. The combat goes by so quickly that many of the nuances can't be properly felt. 
(Thanks for DominusExult for pointing this out to me!)
* I'm not well versed in C++, so it's possible I grossly misread/misinterpreted something. 
Please point it out! However, I'm simplifying some of the code for conciseness, e.g., many 
if statements are quite complex, and I'll only mention the most relevant aspects at each part.
* Anything pertaining usecode is, at this moment, a complete black box to me. I didn't get into 
it in the least bit.

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
makes U7 special to so many. In essence, it's a form of artificial intelligence. From now on, 
I'll use the terms Actors and npcs interchangeably.

The simplest schedule is `wait`, which does **nothing**, followed by `loiter` which means wander
around aimlessly. Even the simplest schedule has a function called `now_what`, which tells the npc
what they'll do once their current task is finished.

The schedule of interest of this document is appropriately called `Combat_schedule`. One of the 
first things that happens in `Combat_schedule::now_what` is to look for a foe. An npc is 
considered a foe if there's an *alignment clash*.

`Alignment` refers to an npc's stance towards the Avatar. There are 4 alignments, starting from 0.
First, `neutral`, comprising most of the named npcs around (Even LB!), then there's `good`, which
is *mostly* the Avatar and its current party, with a few exceptions. Then there's `evil`, which is
hostile to `neutral` and `good`, and `chaotic`, which is hostile to all. These can be temporarily
modified by being charmed (and the `charmed_more_difficult` game option). Even if you attack a
peaceful npc, yours or their alignment won't change.

During combat, you attack with melee weapons, thrown weapons (good or bad), or with magic. These all
have a base damage, powers and damage types. Hit probability and damage calculations are modified by
the attacker's and defender's stats (STR, DEX, INT, COMBAT). Unless usecode shenanigans are at 
play, everyone is considered the same way!

After defeating an enemy, you gain experience points and, if you kill the enemy, access to (part of) 
its inventory. Combat continues until all enemies around are dead or unconscious.

You can enable a debug feature in Exult called `combat_trace` to print out some additional 
information about what's going on in combat to what's called `standard out`, or `stdout`. If you 
run exult from a console, it'll print out to that, or it'll be in the file called `stdout.txt` 
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

The simplest way of entering combat is by toggling the combat state either in the ragdoll screen 
or pressing 'C'. This iterates through all party members and sets their schedule from 
`Follow_avatar` to `Combat`. In this state, everyone in the party will start to look for enemies 
and approach them. But not before! You can calmly walk around and talk to npcs with hostile 
alignments provided none of you enter the combat state. The instant you press 'C' however, you'll attack. 
For example, Iriale Silvermist, located in the Fellowship retreat, is Evil but peaceful (for 
some time), as well as some npcs in New Magincia (Robin, Battles and Leavell).

An npc being attacked is another way of entering combat. An `Actor::fight_back` function is 
called. First, it checks if someone in the party is being attacked and the Avatar can't act 
(paralyzed, asleep, knocked out or dead), and if so, sets everyone to fight. If the Avatar was 
being a bully and attacking a peaceful npc that can react back or someone saw or heard the 
attack, between 1 and 3 guards will be summoned from offscreen to fight/arrest the Avatar and 
up to 3 sentient npcs (INT >= 6) in the vicinity will be set to attack the Avatar. Funnily, 
their alignment is *chaotic* and their schedule is set to talk to the Avatar.

From what I've found, enemies created from eggs, summon, cloned (slimes) or manually placed are
either in the `Combat` or `Patrol` schedules. In the `Patrol` schedule, and its derived schedules,
if the npc is set to look for combat and it sees another npc with an alignment clash, it will enter
the combat state. Npcs have a counter to determine how many times they've failed to find an
opponent. After having failed, if they had a previous schedule and their alignment is good, they'll
return to their previous schedule (TODO: re-verify when to give up and explore other spawn
situations). Others just keep trying and trying to find an enemy until you walk far away enough from
them.

Now that the required parties have entered combat, everyone will go through its 
`Combat_schedule::now_what` function. 

------------
Funny thing is, even if they aren't hostile when encountering them (because their schedules don't
seek out foes), if you enter attack mode near them, you'll attack them "unprovoked".
### What happens every turn

### Differences between weapons types and magic

Magic works different between enemies and the Avatar, who requires a spellbook. Spells are
actually items they can equip and "throw" at you. This makes spellcasters especially dangerous.
When killed, these items are removed from them so you can't access them natually. However, I
distinctly remember a few jesters around the castle of the white dragon that spawned with death
bolts or something like that, and which I could loot (in the original)

-----------


## Hit probabilities


## Damage calculations

--------------------

That's why you don't
find liches or mages, for instance, with reagents and spellbooks. When they die, the "magic"
weapons are removed.

When receiving damage, an npc's health is reduced. If it gets to zero or below, it's knocked 
unconscious. In this state, its dex is reduced to 0 (meaning it can't attack) and it is given 
less priority than conscious opponents. 

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

* Weapon have a few types of powers:
    * sleep, charm, curse, poison, paralyze, magebane, unknown, no damage (puts Dragan to sleep, see [here](https://ultima.fandom.com/wiki/Draygan))
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

* There's checks for paused, checks for difficulty (0 = normal, >0 = harder, <0 = easier). These can be accessed from the menu!
* There's two combat modes, original and keypause, with the latter using space as suspend/resume combat. This really lets you direct attacks from specific party members to specific targets.
* There's an option to show hit numbers? -- yes! That's actually an option in the menus.
* What's charmed more difficult? - According to the manual, does this:
    * In normal, Avatar can't be charmed. In the original, he could, but nothing would be changed. In hard, charmed party members can't have their inventory accessed, and will continue to attack even if combat is stopped.



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
    6. Strength is balanced if there's little of the ammo (if there's less than 5 times what's needed). If so, strength is divided by 3. If it's less than 10 times what's needed, strength divided by 2.
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
2. If it's less than 0 (i.e., -1), it uses the weapon's reach. If there's no weapon info, uses monster's specific reach or the default reach.
3. If (not uses), which means, if hand-hand or ranged, returns the weapon's reach.
* Uses is, from `weaponinfo.h`:
    * 0 if hand-hand, 1 of poor throwable, 2 if good throwable, 3 if missile firing
4. If either a poor or good throwable, gets the strength and combat stats of the actor. Whichever is higher is the current range. If it's a good thrown weapon, the range is multiplied by 2. If it's lower than the reach, then it's replaced by reach, and if it's greater than 31 (units?), it's cut to 31.

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
