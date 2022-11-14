### Stupidest Bugs and Implementations

Since most of my time has been spent smacking down whatever leftover bugs happen to be in the code before I submit the game to competitions, I thought it'd be fun to compile a list of the stupidest bugs I've seen in the code. Some of them were my fault, some of them were baffling, and most of them were pretty funny.

## V-Sync Screws With Music

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/vsyncbug.png "This one still gives me fits.")

Back when I was initially implementing audio into the game I set it up so that music would fade in and out as you navigate through menus and the game. Thing is, that's a thing that inherently requires a timer - "fade out for .5 seconds, then fade in for .5 seconds" - and somehow that did *not* play nice with visual synchronization, presumably because v-sync screws with the FPS and therefore with the timer. I don't know. This one got fixed, but it still blows my mind.

## Turrets Are Afraid Of You

I recently implemented a bunch of AI behaviors, which I think I mentioned. One of the flags available is the flag for "flees when targeted" - that is to say, if you have the AI in your firing cone, it's going to try and run away from you.

However, because my turrets use the same AI that the rest of the game does (they just don't have any lateral thrusts - exclusively rotations), this meant that when their AI was smart enough to have the "run away" behavior enabled, the turret AI would literally try and run away - fruitlessly, due to being locked in place. I only discovered this after I noticed that turrets kept turning away from me when I got close. I eventually realized that they were *afraid.*

As they damn well *should* be, but I have sinced patched it so that turrets know no fear. They will simply shoot at you until you leave.

## MISSILES Are Afraid Of You

In a hilarious turn of events, I realized my missiles - which also use AI behaviors - *also* had the same problem, where they would spaz the hell out when getting too close to an enemy target. I had no idea why this could be until I remembered my turret bug, and sure enough, the missiles got really, really scared when they were too close to the enemy target and tried to run away. Poorly. Missiles are not optimized real well for turning, so they mostly just spazzed in place.

The game has been patched to since make sure missiles *also* know no fear, but shit, I'm kind of tempted to create some sort of "fear" augmentation after those.

## The Wingmen Exclusively Spawn Clones

This one was undetected for a long, long time, since for awhile there I only had the one ship type - the Tuxedo - and one type of human weapon, which was the Tsunami LMG. I'd noticed that my game wasn't loading wingman names properly, like it does now, but I shrugged my shoulders and figured that was just some sort of initialization error. It was, but not in the way I think.

I only realized this after adding the Anubis and several new types of weapon - it wasn't spawning the wingman at all. It was instead, for some fucking reason, spawning an exact duplicate of the player ship. I don't remember *why* exactly this was aside from extremely poorly constructed code, but it got ironed out quick after I knew what to look for. That one was fun to track down.

## You Can Outrun The Audio

OpenAL has tools to simulate the doppler effect. I know there's no *sound* in space, but we're already breaking that rule, so why the hell not go with having some cool doppler sound effects as you're moving around?

The problem here was pretty simple - OpenAL's arbitrary internal units treated the speed of sound as the speed of sound expressed in meters per second, which is 331.5 m/s. My game's units allowed for speeds well beyond the "sound barrier" here, and if your ship boosted in any capacity or turned off safeties, you would literally outrun sounds and be unable to hear shit until you turned around, at which point the soundscape would snap back into focus.

While that sounded cool as *hell*, it ultimately got on my nerves and made sound management hell, so the speed of sound got bumped up and the doppler effect got turned down. It didn't help that during this time, all sounds were playing as stereo sounds. That is to say, all sounds were playing at max volume like they were *right next to me.* I ripped off my headset a few times.

## Carriers Have No Idea How Big They Are

The initial implementation for some behaviors like "form up" where your wingmen will attempt to get close to you and stay there, then get to your orientation was pretty good, but it had one small problem - it didn't account for scale. It's all fine and dandy when your wingman wants to be 50 meters off your left side, but when your big-assed carrier wants to do the same thing, it's either going to crush you like a beer can or spend all its time contorting instead of, yknow, flying.

This also went for "pursuit" behavior on larger vessels, actually, which I believe [I mentioned many months ago](https://wizard-of-chaos.github.io/2022/05/17/scenarios.html), where since the carrier had no concept of how big it was it just piledrove itself straight into whatever it happened to be targeting. I fixed the former, but kept some of the latter behavior because I think it's incredibly goddamned funny to watch huge ships plow straight through smaller ones. Am I right? [See for yourself.](https://youtu.be/KBYkjy9nta4) This is from back when I re-implemented allied carriers.

## Angular Velocity Measured Wrong

This one got [detailed here](https://wizard-of-chaos.github.io/2022/10/14/roid-generation.html), but it's a great example of why you should really pay attention to unit conversions.

> A radian is a value from -PI to PI. If an object has an angular velocity of PI, it’s completing one rotation every second. I was initializing things with angular velocities not ranging from -3.14 to 3.14, but ranging from -360 to 360. This is bad for a variety of reasons, first and foremost being that asteroids shouldn’t complete several hundred fucking rotations every second. This was quickly fixed.

## Stray Turrets Are Empaths

Another one that I found earlier and detailed [here](https://wizard-of-chaos.github.io/2022/08/26/debris-sector.html).

> These actually led me to find a bug in my sensor code. Effectively, these turrets have a different type of hostility - since they're shooting at both friendlies and hostiles - which required a slightly different "faction" setup for them. I couldn't figure out why the hell they weren't shooting at me for awhile, despite spawning in properly, and eventually tacked it down to a chunk of my faction code that checks whether or not a given ship is a hostile. It was actually checking whether the *other* ship would be hostile instead of whether *you're* hostile to them. Of course this doesn't matter when you only have two types, since they'll be identical, but it doesn't work *here*. That was neat to find.

Indeed, it was neat. All "factions" were basically mirrors of the other guy. I know what you are feeling because I have told you what you are feeling.

## Character Attributes Lacking

This isn't a bug so much as it is "useless feedback":

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/feedback.png "Poor Cat.")

This is, without question, the single funniest piece of feedback I have received on this game.

## Explosions just KEEP GOING

A particularly stubborn bug I had with some of the particle effects is that they weren't recognizing when they should be "finished", or delete themselves as entities, as well as they should, due to several problems with initialization and a *really* harebrained explosion setup that I'd written out six months prior to doing anything interesting with, back when I still had no idea what I was doing. Effectively, it wasn't recognizing some entities as the type it needed to catch and delete. What this *meant* was the battlefield would occasionally turn into a minefield of explosion effects that just *kept going*, like some kind of god-awful nuclear disaster.

This was solved when I both reworked the effects system to be less silly and discovered the "delete" animator for scene nodes in Irrlicht, which just automatically deleted nodes after a set duration. Easy enough to fix, but it was there for *months*, and sometimes made the game look really, really weird.

## The Developer is an Idiot

This was an implementation-specific problem, because I was simply just doing things wrong.

When initially coming up with the methods for how you would get random weaponry, I had the developer weapon (the one with max damage that I use to blitz through the game for bug-testing) set to a negative value, and all the weapon data was stored in a map to the weapon's ID. My previous method of simply doing `std::rand() % weaponData.size()` wasn't going to work. So I decided, okay, let's be really clever here - let's set it to `% weaponData.size() - 1`. That'll prevent it from going out of bounds on an access, since on size 8 with the dev weapon you technically only have 7 values applicable.

I ran into a roadblock here eventually, though, when I was implementing weapons that I didn't want the player to be able to just loot *besides* the dev weapon - turret specific weapons, or carrier specific weapons, or any number of things really - just stuff that the player wouldn't be able to loot. So I'm thinking to myself okay, do I re-define all the IDs for weapons and make it so that the rollable weapons are in a *range* of possible values? And all the un-rollable weapons are outside that range? This approach has problems too - how do I fit in the dev weapon? Do I re-define this every time I add a new weapon? I was attacking this question for days, off and on, trying to think of a good way to fit it into my code.

Looking through my weapon data storage function one last time, late at night, my eyes happened upon the field I had for whether or not you could build a weapon: `canBuild=yes`. Thinking briefly to myself, I typed into the weapon data `canLoot=yes`. I made the roll function roll for the entire table and if the weapon's `canLoot` value was false, it rolled again. It worked on the first try. I closed Visual Studio and I went to bed.