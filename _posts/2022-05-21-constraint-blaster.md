## Gravity Bolas Implementation

With a lot of the really dull work out of the way, it's now possible to get back to the actually fun thing about game design - new weapons.

Before today, there was only one kind of physics-based stupid weapon, and that was the impulse cannon. That weapon is the utter essence of simplicity as far as physics assault goes. You fire the cannon and on impact, it triggers an explosion causing a massive blast of force to anything unlucky enough to be caught in the area. It's simple, clean, and incredibly entertaining to bounce the enemy fighters around like a ping-pong ball in a washing machine.

I have plenty of ideas for other weapons, but the first one I wanted to tackle was a constraint weapon, which I titled a "gravity bolas" in traditional sci-fi nonsense nomenclature. My idea was pretty simple: you fired a shot at two objects and they were yanked together, eventually slamming them together to cause impact damage or just setting them up for an AoE attack like a missile.

If you're not big into game physics, the term "constraint" might be nonsense. It's pretty simple; a constraint is a chunk of code you send to the engine that informs it about the way two objects behave. For instance, I might set up a constraint that says "I want object A and object B to not get more than 15 meters apart", and the engine will take steps to make sure that doesn't happen - usually by applying a huge force to make sure that they stay under 15 meters apart. In my case, the gravity bolas would ask for a constraint where the two objects I shot are 0 meters away from each other - colliding. In my head I pictured the classic cartoon moment where someone knocks two peoples heads together.

My infrastructure, unfortunately, was not set up for such a weapon yet, so I had to do some cleaning to get it to where it is. The previous implementation of a weapon collision effectively just applied damage, and had a rather stupid check to see if it needed to do anything special:

```cpp
	if (proj->type == WEP_MISSILE) {
		explode(irr->node->getAbsolutePosition(), 1.f, 1.f, 20.f, proj->damage, 100.f);
	}
	if (proj->type == WEP_PHYS_IMPULSE) {
		gameController->registerSoundInstance(impacted, stateController->assets.getSoundAsset("physicsBlastSound"), 1.f, 200.f);
		explode(irr->node->getAbsolutePosition(), 1.f, 1.f, 80.f, proj->damage, 500.f);
	}
```

You might notice that this is a dumb way to handle things; not very linear and certainly not very extensible. What I needed was a couple of more specialized functions to handle any sort of weirdness that might be necessary for a projectile impact, so in the case of a missile a highly damaging explosion and in the case of my impulse cannon a low-damage, high-force explosion. In the case of my gravity bolas, it needed to do a bit more.

---

### Inadvertent logic change

Along the way here, I noticed a problem that I had yet to get around to fixing; the projectile code was being handled in a rather silly way. The projectile would constantly check how far away it was from its spawn location, and if it were further than a set "range", it would de-spawn. I needed to change this but forgot initially; an example of how this is stupid would be a missile looping around the spawn location and never despawning. The change made is now simply a timer; a projectile will de-spawn after a set amount of time, solving the issue. Score one for new features causing unintended design fixes.

---

### Back to the goofy weapon

The gravity bolas needed two things: it needed a way to track whatever it hit, and it needed the track the constraint so it could remove it after a set duration. Easy enough. 

```cpp
struct BolasInfoComponent
{
	EntityId target1;
	EntityId target2;
	f32 duration;
	f32 currentDuration;
	f32 timeToHit; //the time required between two hits
	f32 currentTimeToHit;
	f32 force;
	btTypedConstraint* constraint;
};
```
My new features call a function to track the entities involved on impact. I also realized I needed to keep track of a "timeout" feature if you took too long between shots, so that also got tacked in. I paused briefly here to make some placeholder audio effects to use for when the bolas latched.

This is about when I decided to look into the Bullet3 docs to see what kind of constraints were available, and made the unsurprising discovery that the docs were almost completely unhelpful. Bullet3's docs are very extensive, but include very few comments on just what the hell a function actually does.

Squinting my eyes and cursing the developers who wrote them (and making a mental note to go contribute to the bullet repo to add some goddamned documentation myself), I decided on a simple Point-to-Point constraint, since it _sounded_ like what I wanted It was... _sort_ of.

https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/p2pconstraint.mp4

It works! I mean, it was completely weak and ineffectual, but it _did_ work. Turns out point to point constraints aren't all that great with variable masses - they're much better suited for something like, say, the links of a bridge. Yanking a large asteroid towards a tiny ship doesn't work all that well. In this case, the AI was actually _stronger_ than the constraint; it barely moved at all because it was issuing a thrust to keep itself still due to its idle state. That said, I did know I was on the right track, and was shocked that it worked without anything breaking.

After some cursory googling and looking around, I wasn't able to figure out any good way to increase the strength of a p2p constraint without getting hacky about it (although I'm sure _someone_ reading this knows a better way). I figured that a 6 Degrees of Freedom constraint was more what I was looking for, which was a more complicated constraint that also worked for restricting rotational values. I don't care about that, but I _do_ care about the increased flexibility and possibility for adjustment, so I implemented one of those as my constraint instead.

https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/6dof1.mp4

Hahahahahahahahahahaha!

As incredibly entertaining as that was (I spent a solid ten minutes just duct-taping the AI to various obstacles), this is clearly not the most balanced it could be. This strength immediately yanked the two objects together as quickly as possible, instantly demolishing whatever the hell got in the way and the objects themselves due to the impact damage. Funny as it is, this needs a bit more nuance to it.

https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/balance.mp4

This is much closer to what I'm looking for. I eventually damped it even more, since this was still a little quick for my tastes, but the basic concept works and it's very fun to send AI ships smacking into each other or whatever the hell happens to be nearby. I think my favorite was causing two asteroids to smack into each other behind me, crushing the ship tailing me like a beer can. I also tested this out with a carrier and had a blast setting up the forces between a carrier and the nearby debris. That was much more fun than just shooting it a bunch till it dies. There will probably be a few more balance iterations, but I think the gravity bolas is here to stay.