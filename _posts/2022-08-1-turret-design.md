## Turret Implementations and Future UI Work

What I've been working on for the past week or two is a solid implementation of how turrets would work on a larger space station or a carrier. This is one of those things that sounds so simple when I initially come up with the idea or the design, but as soon as I waded into an actual implementation there were just hurdles one after the other that I'd never really considered. Most of which were self inflicted, but that's how it is most of the time.

Way back when I first thought about how this would work, I considered my previous implementations and decided on what a turret would need. It'd need to know its cone of fire, its weapons, and how quickly it could pitch and yaw to be able to get to those. It'd also need some AI of its own, obviously, but I'd already written that. So I slapped this into the repository:

```cpp
const u32 MAX_TURRET_HARDPOINTS = 2;

struct TurretComponent
{
	vector3df startingRotation; //this constrains how much the turret will be able to pitch/yaw
	vector3df hardpoints[MAX_TURRET_HARDPOINTS];
	flecs::entity weapons[MAX_TURRET_HARDPOINTS];
	f32 pitchThrust;
	f32 yawThrust;
};
```

...and called it a day, going back almost immediately afterward to work on whatever the hell else I was doing that day. I didn't come back to this code for at least two months (probably more like four) and didn't actually use it anywhere until I started work on this. The first thing I did was created my usual attribute loaders so I could load turrets from file and then add them onto ships and what-not: boilerplate work, basically.

The first problem I ran into was this: my movement controls were fundamentally not designed for turrets. The only things that moved around in the scene were ships, so the appropriate thrusts and such were loaded onto the ship component. Since anything moving needs to know how hard it can thrust, all my functions looked kinda like this:

```cpp
//Gets a force in the up direction, modified by the ship's strafing thrust.
btVector3 getForceUp(btRigidBody* body, ShipComponent* ship);

//Gets a force in the down direction, modifed by the ship's strafing thrust.
btVector3 getForceDown(btRigidBody* body, ShipComponent* ship);
```

You see the problem. These are all taking in ship components. So if I wanted to be able to move my turrets around properly, I'd need to rewrite these to be able to accomodate for turret components and end up repeating myself a whole hell of a lot. This solution is actually pretty simple, and came mostly out of my burning desire to repeat myself as little as possible. The movement functions here and the movement systems have *absolutely no business* using a ship component. What needs to happen is that it should use a *thrust* component, that I can attach to *anything*, that should tell it how quickly it can move in a given direction. Fine! Let's set that up.

```cpp
struct ThrustComponent
{
	f32 pitch=0.f;
	f32 yaw=0.f;
	f32 roll=0.f;
	f32 forward=0.f;
	f32 strafe=0.f;
	f32 brake=0.f;
	f32 boost=0.f; //if the thing has a boost capability (afterburners).
	bool moves[MAX_MOVEMENTS];
	f32 velocityTolerance=0.0001f; //How tolerant the entity is of going over the max speed it has
	f32 linearMaxVelocity=1000.f; //Max linear velocity
	f32 angularMaxVelocity=1000.f; //Max angular velocity

	ThrustComponent() {
		for (u32 i = 0; i < MAX_MOVEMENTS; ++i) {
			moves[i] = false;
		}
	}
};

```

I won't bore you with the details here, but suffice to say this involved trudging around the repository and doing a lot of mind-numbing work to rewire everything so that it used a thrust component instead of a ship component. The upshot here is that now turrets can just use one of these - their linear thrusts and their roll thrusts can just be set to 0, and I can set their pitch and yaw thrusts to get their rotations down. Great! Now our turrets should work as intended. I can slap 'em in and they'll twist around with a basic AI component to follow my ship.

Except not quite. Y'see, my AI setup also had problems that didn't account for this. Let's take a look at some behaviors from my AI code...

```cpp
	virtual void idle(ThrustComponent* thrust, ShipComponent* ship, BulletRigidBodyComponent* rbc);
	virtual void flee(ThrustComponent* thrust, ShipComponent* ship, BulletRigidBodyComponent* rbc, IrrlichtComponent* irr, flecs::entity fleeTarget);
```

And there's another problem. I've reworked it to use a thrust component, but I'm still using the ship component - this time to make sure the AI can actually shoot. It took me an embarrassing amount of time to figure out why my AI wasn't working here, actually, but as soon as I spotted it I cursed myself and my lack of foresight. Once again, the solution here seems obvious enough. This needs to be genericized so that once again *anything* with these components can move and shoot. We need still *another* component, this time for hardpoints.


```cpp
const u32 MAX_HARDPOINTS = 6;
#define PHYS_HARDPOINT MAX_HARDPOINTS +1

struct HardpointComponent
{
	u32 hardpointCount;
	//This and the weapons array are initialized to the maximum number.
	vector3df hardpoints[MAX_HARDPOINTS];
	vector3df hardpointRotations[MAX_HARDPOINTS];
	flecs::entity weapons[MAX_HARDPOINTS];

	vector3df physWeaponHardpoint;
	vector3df physHardpointRot;
	flecs::entity physWeapon;
};
```

And now with this my AI was finally working again, this time for turrets. I ran into a few bugs here and there (Carriers were no longer spawning ships with guns, for example, because I hadn't adjusted them to account for the new component), but the transition was pretty smooth, considering the amount of code that needed rewriting.

Hilariously, my ship component is now mostly barren. It has a few toggles for things like the safety override and some vectors to show where engine and maneuver jets are (used by the ship loader), but that's pretty much it - all of it has simply been abstracted out to other pieces. The other neat thing about having to yank out so much code and put it elsewhere is that now I can do this to *anything.* As soon as I got done, I had visions of some sort of idiotic weapon where you fire a little package at an asteroid or some debris and suddenly it's acting like a ship. I should be able to rework my missile code to use this new thrust / hardpoint thing as well. I could also have a weapon that fires off little micro-drones! The future is now.

### So was that it? Was it really that simple?

Hah! No.

No, you see, now that I'd done all of this work to define how turrets actually work and behave, I still had some glaring problems with clean-up and actual functionality. I needed to figure out a way to make sure of a few things:

1. That turrets do not collide with whatever owns them,
2. That turrets can't shoot whatever owns them (their projectiles shouldn't collide),
3. That turrets *stay attached* to whatever owns them, specifically at their hardpoint,
4. That turrets get destroyed properly and that the game doesn't crash when their owner dies.

All of this was simple except for the third point. I already had some code in place to make sure you can't literally shoot yourself in the foot, and extending that code was pretty simple, although it required a redefinition of "ownership" given the new and exciting use case for it. The collision code was likewise simple; just another relationship added. The finicky one was the attachment.

I knew how this needed to go down from the start - I'd be using a Bullet3 constraint, similar to what [I did for the gravity bolas weapon](https://wizard-of-chaos.github.io/2022/05/21/constraint-blaster.html). That constraint was simple, though - I just needed to make sure two entities were powerfully attracted to each other. This one was more complicated. I needed to first make sure that they stayed at the hardpoint location *specifically*. That means I needed an extremely strong linear  movement constraint with a very specific pivot point. I don't mind saying this took some time to figure out because getting any usable information out of the Bullet3 docs for constraints is [like trying to translate an alien language](https://pybullet.org/Bullet/BulletFull/classbtGeneric6DofConstraint.html). I'm sure this would be useful if I had any goddamned idea what half of it referred to, but I'm not a physics engine designer - I'm an idiot with a keyboard and a working knowledge of C++. This shit is like code archaeology.

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/daniel.jpg "Jack, it sounds like they're speaking an ancient dialect of C... they're, they keep muttering something about macros. I need my reference material.")

But, eventually, I figured this out too and got it all set up. The next thing to do will be (and I haven't written this yet, but I know exactly how it needs to be done) setting up the constraint to be equally rigid on rotations - that is, making sure the turret can't shoot straight through the ship. Should be easy enough. As it stands, I was really happy when I got the turret to stick with the ship and shoot at me the whole while. That was a fun moment.

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/turrets.gif "I'm using the gravity bolas here to test out yanking them around. It worked beautifully.")

### What's next?

Well, this all happened like four days ago. I took a few days off to go to an anime convention dressed as a character from western sci-fi. It was vitally necessary, you understand.

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/sheppard.jpg "FUCK the Genii!")
![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/wocsheppard.png "FUCK the... convention background!")

But right after this, the problem was actually still more planning. I got together with a friend and hammered out how the campaign should end up looking. Up to this point, I've been more concerned with implementations of features than with larger-scale stuff. Now that the vast majority of that is out of the way, I can consider what I want the actual... well, game to look like. I think I've got a good idea of where gameplay is. The question is how do we get around that gameplay?

Between us we decided on a format: there will be several scenarios in a row, like there presently are, but each will be part of an individualized "sector". Wingmen will be static and picked up along the way, with a bit of randomization so you might not get the same wingmen every single run. The random scenario generation will be kept, but more specific - if your ship is in an asteroid belt, it will primarily generate asteroid scenarios. If you're in a supply depot type area, it will be debris fields and space stations. This will culminate in a large-scale final battle.

I mentioned for AI design that I was effectively building personalities. We're rolling with that. A dialogue system is going to be implemented so you can talk to your wingmen and explore them and their personalities a bit more. This will also allow for more effective communication as far as the plot goes - your ship is the only survivor of a horribly lost battle, and now needs to escape the system, which is now utterly overrun with enemies. You'll be able to talk to your wingmen and some ship personnel to get more info on this, or if you don't care much you'll be able to skip that. The structure of all this has now been plotted out, and the planning as far as progression goes looked like this:

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/roughprogsketch.png "Expertly drawn with a tablet and impatience.")

This was scribbled out to get a good idea of how the game is going to progress from sector to sector. We have that now! Yay!

The problem as it stands right now is not only do I need to actually *implement* sectors, I've just guaranteed myself a ton of extra UI work - there's now at least three additional menus I'll need, with *massive* functionality as far as game-state goes, and then on top of that I'm going to also need to write an entire dialogue system and figure out how that works. Simply put, my current UI code is ill-equipped for this, and in order to implement a lot of the complex stuff I'm going to need, it requires some significant utilities tacked on. I need to make it more robust, extensible, and easier to write more menus with greater complexity.

So I'll probably end up doing that for awhile, but knowing myself in-between I will probably be throwing some time at the various other implementation bits I still need (sectors and construction bay), so the next update will likely be about those. Don't be fooled, though - those will be procrastination on UI reworks. Shit's ugly. On top of that, given that I'm actually *employed* as a developer now, flight sim development spare time might drop sharply. But that's where we are - while planning all this out and looking at what I currently had, I was really happy with where the game is going. The gameplay is fun and extending it (with more weapons, ships, etc) will be a joy, and if I get the context for it down properly with a simple plot and sector design, we'll have something in a releaseable state.

To think this all started as the result of nerd rage about space sims and the need to have a side project for sanity's sake while job-hunting. I never thought I'd get this far.