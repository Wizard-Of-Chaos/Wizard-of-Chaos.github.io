## Objectives and AI and UI, Oh My

Debris sector is pretty much done! The only thing left to do is the mess hall for wingmen, but otherwise I'm pretty happy with how the rest of it plays. I'm a bit behind where I'd like to be for the month, but honestly crackheading out an entire UI in three weeks alongside myriad other changes probably isn't too bad of a pace. This time I'm going to talk a bit about some of the objectives I just added as well as what I've done to the AI to make it fly better.

### New Objectives and new HUD

I've had the code for some of the additional objectives like "retrieve object" and "destroy this station" sitting around for awhile now, but I've been busy with other stuff, so it hasn't been used much. Now that the UI is (almost) entirely completed, this is no longer the case, so one of the first things I added was a new space station objective to replace the carrier as the "destroy object" objective.

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/station.png "Whoa!")

Oh yeah - while I'm at it, the HUD has been updated to look less like garbage. I'm probably gonna stick with this general look for the future, although individual elements are likely to get higher-res updates as you might expect. Also in this image are some objective markers to show what the current objective is and a big ol' station. The station is effectively the same principle as a carrier where it has tightly-bound turrets tacked onto it. It does not, however, have any way to spawn more ships, so it's effectively just the four turrets you see. Pretty simple.

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/stasis.png "Taken from a significant distance away.")

I've also added a "salvage" type objective, with the first of these being a stasis pod to recover (the picture is taken from a decent distance away because you collect it when you get close so it's hard to get a close-up shot). Once you've retrieved a stasis pod, you'll add a new wingman to your crew - and after you retrieve it, you'll need to mop up the nearby enemy ships.

This brings me to the other thing I've done - some more dynamic HUD and objective updates. Effectively objectives now follow in a chain, one after the other, so I can theoretically link up as many objectives as I want. In this case it switches off the "collect" objective and then creates a new mop-up objective for the other alien craft still in the vicinity, changing their HUD icons appropriately. Fairly simple, but I'm sort of kicking myself for not implementing it sooner. What the hell was all that [nonsense about objective components?](https://wizard-of-chaos.github.io/2022/05/19/objectives.html) Damned if I know. This method is easier and actually more flexible.

On top of the HUD updates and the objectives, there's now also a quick loading screen with a tip and some art on it. Before adding that, I don't think I'd have ever said that loading screens *add* to the experience of a game. I woulda been incorrect; the prior system effectively just made the game look like it froze for a few seconds. Having an actual loading screen indicates "oh, yeah, it's loading" to the player. You'd *think* you would get that just from the name, right? I never claimed to be all that smart.

The carrier objective has been moved to the end of the scenario as the boss-fight for the sector, so we're all wrapped up there with objectives for the debris sector! The in-game combat is pretty fun and works about how I expect it to. Especially since I've added...

### Heavy Weapons

Heavy weapons are a thing now! They're an additional hardpoint that fires really really high-damage guns with something that makes them utterly crappy for ship-to-ship combat. I anticipate them being used as anti-large weapons (carriers, ships, hell, even random debris) almost exclusively. The first thing I did was move the missile to a heavy weapon slot - better for everyone that way, since the tracking is still a bit off - and implemented a plasma flamethrower that melts anything about two feet in front of it and hits nothing outside that range. If you wanna use that versus enemy fighters, be my guest! They're likely to fry you before you get in range, but it's great versus large things and if you *do* manage to hit them, they're not getting back up.

It's even harder to hit enemy ships, though, since the AI flight patterns have been upgraded.

### AI Upgrades

Much to my shame, I have yet to implement [goose flight mechanics](https://wizard-of-chaos.github.io/2022/07/18/ai-design.html), specifically the honk behavior and the "run away when chased" behavior, but I'll get there - those are actually pretty easy to implement. My bigger problem was that the AI flew like a robot, which is no fun for anyone and is wicked easy to outsmart.

Here's my AI flight code as it was, in pseudocode:
```cpp
turnToDirection(direction);
if(angle < 20) {
	//figure out brake time and arrival time
	if(timeToStop >= timeToArrive) {
		brake();
	}
	else {
		thrust();
	}
}
else {
	brake();
}
```

Effectively it just looked at the angle to the target, and if it was above 20 degrees, it would hit the brakes so it could turn to the direction, and if it was under twenty degrees, it would thrust if it needed to and brake otherwise. This is, technically, perfectly fine flight behavior to get to a point, but it's crap for moving targets. If I were to shoot by the AI, for example, it would stop flying completely and turn towards me before hitting the gas again - and it *always* slows down on approach so as to not overshoot. This led to a lot of sudden stops and starts, which looked mechanical and was easy to shoot.

So I looked at that, then looked at my own actual fingies on the keyboard while flying, and realized pretty damn quickly that I don't fly like a robot - I'm much less strict about angles, for one, and I don't care if I overshoot so long as I can get off a clean shot - I'm just gonna hit the brakes on the rebound, strafe, and turn back towards it as fast as I can. My next take on flight behaviors looked like this:

```cpp
turnToDirection(direction);
if(distanceToTarget < 45) brake();
if(angle < 50) thrust();
```

Way simpler, right? It actually looked *far* more natural in some test flights. It turns toward you, sure, but is far more relaxed on the restriction and starts thrusting way sooner - and it doesn't hit the brakes at *all* until it's sure it's close to the target. This led to a lot of hilarious overshoots and swoops that looked like someone was flying while drunk. It was way more difficult to hit, but it also had absolute jack for precision and couldn't hit *me*. This is clearly not ideal.

You'll also notice this makes no mention of strafe behavior, which I personally as a pilot tend to use frequently to correct errors in my own flight path. Plainly this needs some sort of middle ground and some additional behavior. My third and current "go to this point" function for piloted craft looks like this:

```cpp
turnToDirection(direction);
if(distance < 45) brake();
if(angle < 30) thrust();

if(direction.left) {
	if(velocity.right > 5) strafeLeft();
}
else {
	if(velocity.left > 5) strafeRight();
}
if(direction.up) {
	if(velocity.down > 5) strafeUp();
}
else {
	if(velocity.up > 5) strafeDown();
}
```

This is broadly the same as the prior behavior (although it has some significant upgrades in the actual code to make it a bit more responsive) in terms of actual logic, although it has a few tighter constraints, and it incorporates strafe behavior. It checks the direction it needs to go and its own velocity, and if the two don't agree it strafes in the direction to compensate.

These combined led to smooth turns, smooth swoops, tight turns and me being shot in the back of the head way more frequently than I was used to. It's rather irritating to have your own AI give you a run for your money in flight patterns, and it's only going to get worse when I implement some of the goose behavior to make the AI even smarter. The better flight patterns do quite a lot for the overall difficulty, and the only thing to add on top of this would be more specific behaviors (if I'm taking too much damage, boost away, if I'm being attacked honk, etc). Suffice to say the AI work is coming along well.

Armed with the new and dangerous flight patterns, I also implemented a PATROL state where the AI flies around a specific path instead of standing still all the time. This is much more dynamic than just having them sort of *sit* there, and looks pretty cool seeing them group up  for defending a structure of some kind.

### What's Next?

Well, I discovered that my audio driver needs some different management due to an oversight on my part (briefly - your sound driver only has so many audio channels, which means only so many sounds can be playing at a time, which means that when I go over that number my sound starts acting really funny). That will need some additional work in the future (probably just a list of sounds and exclusively playing the ones that are loudest through however-many audio channels the user has). I bumped into this while trying to implement idle sounds for some things, so that's delayed a bit.

I also need to finally implement retrieval of wingmen (currently the scenario just completes) as well as the mess hall UI so you can actually talk to your wingmen in the downtime, as well as loading up conversations for the wingmen you have and implementing effects from said conversations. That is honestly not too hard to do, but I'd hoped to get it done by the end of September and I'm clearly missing that mental deadline. Oh well; it should only take me a few days (art assets notwithstanding).

Once I have that, though, the first sector is pretty much done, like I said, and it'll be time to move onto the asteroid sector and see about implementing some new and shiny terrors for the player! As well as some new weapons, physics weapons, heavy weapons, ships, you know - game design stuff. Should be soon. I'm planning on seven sectors total, as I believe I said [in a previous post](https://wizard-of-chaos.github.io/2022/08/01/turret-design.html), but I wanted to get all the work done for UI and such done immediately so I'd have less work in the future. 

And now I have less work in the future! Feels good. The work continues, but at least the game is pretty damn fun to play now if I do say so myself!