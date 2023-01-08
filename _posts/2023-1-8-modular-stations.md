### Modular Stations

I've been struggling conceptually with the supply depot sector for awhile now. I mentioned [on the previous post](https://wizard-of-chaos.github.io/2022/12/22/stutter-optimization.html) that it necessitated a bunch of new station models, ship models, and general mechanics, all of which are pretty slow going for me as a single dude. Thankfully, I've got modelling help from my team. I guess I have a 'team' now for this thing, come to think of it. Weird feeling.

Anyway, in my head what I considered the basic mechanic for the supply depot sector would be capturable stations and defense missions. Effectively, this needs a few things. I need the ship model for a shuttle (done), I need a point-defense style objective set up (done, this is what the previous post was anticipating with making it easier to spawn waves of enemies), I need another objective as a timer for how long it takes to capture a station (partially done) and I need docking to identify when something has been "captured" by whoever happens to be aboard the shuttle (not done). In terms of game design and general narrative, this also necessitates a few other things -- I needed to come up with a reason for space marines (in my head I can't *not* prounounce that ['spess mehrens'](https://www.youtube.com/watch?v=MbRGg3e1b68)) to be aboard the ship with the new shuttle, and I need a "face" for them too for character interactions. I swear to god this is the last main-plot character I'm coming up with. *Last one.* (I'm going to eat my words in less than a month, and I know it.)

So those are all the systems I needed for capturable stations. Now how do we make this fun gameplay wise? And how do we make it *unique* gameplay wise? The previous sectors are littered with random generation, which I've already claimed is not inherently interesting by itself. In their case different asteroid configurations, gas cloud configurations, debris types, asteroid types, gas types, etc., all of these things behave differently and make any configuration of the environment different enough that missions will emphatically not play the same. In the future I'd like to tweak this a bit more so they're even less similar.

This sorta breaks down for the supply depot sector. The flying might be different, but at the end of the day, you'd still be docking on... let's say one of five stations, assuming I make five station models, which would capture turrets and require wave defense. There is no functional difference between these five, really, aside from where the ship might dock. It'd play pretty identically from mission to mission and you'd effectively be doing the same missions eight times in a row no matter *how* many optional capture stations there were around. This did not sit well with me, and I was struggling with it until talking about models with one of my team members.

"Okay, just a thought," he remarked, "How about making these stations out of modules? Mix and match - either manual, or spawning them together. Easier debris."

"I--"

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/CANTWAKEUP.png "BRAAAAAAAAIN BLAAAAAAAAAAAAAAAAST")

## Development

Obviously, this struck fire pretty much instantly. A bunch of ideas immediately ricocheted through my head, and I had to sit down to figure out just how the hell I wanted this to get done. The first order of business was figuring out how I wanted these to connect, since that's sort of the fundamental point for modular stations. I sat down in Blender and quickly sketched out (is 'sketch' the right term here? Extruded? Transformed? Morphed?) a sort of "plug" for these stations as a connecting piece. Station modules would connect at these points.

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/plug.png "I originally intended this to be the other way around, as in the wide part was the connector, but then it occurred that I was being stupid.")

Cool. That's standardized. That's how I want a connection to look between two modules.

Next up I sat down and mentally sketched out what a connection *entailed* -- code wise. I wanted a concept of "power", so if you broke two pieces apart, and one was connected to a reactor and the other wasn't, the former would remain operational and the latter wouldn't. Additionally, I wanted to make it so that there were a few other things applicable to the whole station, such as a sensor array, turret platforms, and shield generators, all of which were modular. Most of these are yes or no boolean values. "Is this thing powered? Is it shielded? Is it *generating* power or shields?"

While I'm at it I needed to figure out what information my system is eventually going to need to know in order to be able to slap two modules together. It needs to know the *position* of a connection point, it needs to know the *rotation* associated with the connection point, and it needs to know how many connections there are. Additionally, it needs to be able to "reserve" a connection point to indicate what position it's connected to the rest of the structure at. These underwent a few revisions to be directional vectors instead of rotation values, but the idea's basically the same - where do I slap this sucker, and how do I orient it?

Finally, it should keep track of what entities are connected to *it* as well as the constraints for those. When I say "constraints", I'm using that term in the physics engine sense. I've previously tackled constraints [for the bolas weapon](https://wizard-of-chaos.github.io/2022/05/21/constraint-blaster.html) and also for [turret design](https://wizard-of-chaos.github.io/2022/08/01/turret-design.html), both of which are just bent around keeping two objects in specific positions to one another.

> "If you're not big into game physics, the term "constraint" might be nonsense. It's pretty simple; a constraint is a chunk of code you send to the engine that informs it about the way two objects behave. For instance, I might set up a constraint that says "I want object A and object B to not get more than 15 meters apart", and the engine will take steps to make sure that doesn't happen - usually by applying a huge force to make sure that they stay under 15 meters apart. In my case, the gravity bolas would ask for a constraint where the two objects I shot are 0 meters away from each other - colliding. In my head I pictured the classic cartoon moment where someone knocks two peoples heads together."

All of the data structures were now in place -- or at least, naive implementations -- so now I could think about how I wanted these to look.

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/stationsketches.png "Lovingly hand-drawn in a fit of madness and general excitement.")

Different modules obviously want to be doing different things here, but that's a later problem -- the first goal in design is coming up with a concept before writing out the code to execute that concept. Here I was trying to think of useful things that the player would want to engage with (or just blow directly to hell to see a big explosion). Maybe you want to knock out sensors so the turrets are blind. Maybe the whole structure is shielded, and you need to blow out the shield generator before you can do any damage to the rest of it or land. Maybe you want to strategically shoot the whole damn thing apart so that the command tower is separated for take-over. And then plenty of things for flavor.

This neatly sidesteps my original problem where all stations function the same. With the notion of *modular* space stations, I can generate these suckers on the fly and cause different mechanics for each individual structure just based on how they're constructed, and I can come up with cool design patterns and add a lot more flavor to the surrounding space as well. Objectives can change and distort (or be failed!) based on what kind of station you're assaulting or taking over or disabling. Plus, frankly, it's a fun coding challenge. I've been getting damn bored just playing Lego with my earlier functions and snapping pieces together that I already wrote. Gimme a tough one!

## Implementations

So now I can actually knuckle down and implement this stuff. First and foremost I want a reactor module.

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/reactorblender.png "I love Blender.")

Cool. Stations can generate off of this. Next I went with a simple "dock" module that only had one connector.

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/dockmodule.png "I love Blender even more.")

I slapped together these two things in Blender to get a rough idea of how the connection might look, and I was satisfied, so now we're off to actual implementations of the constraint that will snap these two connections together. Thankfully, I have some experience with these by now (much to my annoyance). Let's look at the gravity bolas set-up.

```cpp
btTransform tr;
btVector3 ori(0, 0, 0);
tr.setIdentity();
tr.setOrigin(ori);
auto p2p = new btGeneric6DofConstraint(*rbcA->rigidBody, *rbcB->rigidBody, tr, tr, false);
p2p->setLinearLowerLimit(btVector3(0, 0, 0));
p2p->setLinearUpperLimit(btVector3(0, 0, 0));
p2p->setAngularLowerLimit(btVector3(-PI, -PI, -PI));
p2p->setAngularUpperLimit(btVector3(PI, PI, PI));
```

Pretty easy. The origin here is dead-center of each rigid body and it wants to put them directly on top of each other. It doesn't care about rotation, relative positioning, or anything else, so I can just use the same "transform" there twice and save myself a line. I don't care about rotations, so those get free reign, but it wants the limits for linear movement to be *right on top of each other* so that the two ships, objects, whatever, smack straight into each other. It took some time to adjust how strong this thing was, but the result was neat.

Turrets were a bit harder.

```cpp
btTransform trA, trB;
trA.setIdentity();
trB.setIdentity();
trA.setOrigin(irrVecToBt(turrHdp->turretPositions[hardpoint] * ownerIrr->node->getScale()));
trB.setOrigin(btVector3(0, 0, 0));
auto constraint = new btGeneric6DofConstraint(*rbc->rigidBody, *turrRBC->rigidBody, trA, trB, false);
constraint->setLinearLowerLimit(btVector3(0, 0, 0));
constraint->setLinearUpperLimit(btVector3(0, 0, 0));
constraint->setAngularLowerLimit(btVector3(-PI, -PI, -PI));
constraint->setAngularUpperLimit(btVector3(PI, PI, PI));
```

Same deal, for hte most part, except for the fact that now I wanted the turret to be locked to a specific position *with respect to the original ship*, at the turret's hardpoint. I didn't care about rotational values (although I will in the future for the turret's cone-of-fire), just that it stayed in the same damn spot. So that means the constraint, with respect to the original rigid body, is at the hardpoint location, modified by how big the damn thing is. With respect to the turret itself, it wants its center of mass to be there, so we just set that to 0,0,0. Local coordinate systems are a little annoying.

You see where this is going. For stations I need to make sure that it's locked in place, like turrets, but I also need to lock it to a *specific rotation* with respect to the other thing. More to the point, I can't just set it to dead-center anymore for either - they have to stay at that point and rotation so the little clamps look like they're attached. And it has to be a wicked strong constraint so it doesn't wobble or stretch when getting shot.

The result here is a lot of inelegant quaternion and vector math and several frustrated days from me, but I can summarize it like this:

```cpp
btTransform trB;
trB.setOrigin(connectionPoint * scale));
btQuaternion quatB;
quatB.setRotation(connectionUpVector, PI);
trB.setRotation(quatB);
```
Hey, that wasn't inelegant at all, my source code just looks like crap for no reason! I'm gonna go and re-factor it after finishing this post so I'm less ashamed of myself.

The connection point for the transform is like it is with turrets -- the connection point modified by the scale. That's the same for both sides. The rotational part is a little trickier, but if you know what the hell a quaternion is it's pretty simple. Quaternions are effectively a set of values that determine an orientation for a given object. If you're not particularly mathy yourself, you can think of it like this -- a 2D position is an X and a Y value. An orientation for a 2D object can then be stored as 3 values -- X and Y for position, and Z for rotation angle to say what it's pointing at. Extrapolate that to 3D space, where you have X, Y, and Z for position, and then W to hold the rotation for a 4D value. If you're a math person and you're spitting blood at this oversimplification, I apologize. Here's a [wikipedia link](https://en.wikipedia.org/wiki/Quaternion) instead.

How you can set up a quaternion in Bullet3 is you take an *axis* value, which is a directional vector determinining local "up", and then a *rotation* value determining how many degrees to rotate the object around this arbitrary axis. In this case, I want the "up" vector for the given connection and I just want to rotate it by PI radians to get it to turn around properly and connect, since there are 2PI radians in a full circle. Relatively simple. This is not to say I didn't airball it a few times while figuring out what values I needed, exactly.

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/failconnect1.png "Not like that.")

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/failconnect2.png "Not like that, either.")

Whoops. I got it eventually.

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/successfulconnect.png "Like that!")

My favorite bug during this process was a slight misplacement of the actual rigid bodies while setting up the constraints, leading to the pieces abruptly snapping into place and as a result causing the whole damn thing to spin like a demon. This was quickly fixed, and then I implemented a patch-job so that it'll self correct in case I screw it up in the future (the central component is always trying to thrust itself to stop rotating). Once I had at least *one* module connected, it was time to go for *more.* After all, my code worked the once, why shouldn't it work for the others?

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/failconnect3.png "Unfortunate.")

Damn it! 

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/finalsnop.png "Success! Only took a few days.")

It was at this point I figured out how to use my axes and quats better and got it set up appropriately. Orientations now make sense as well as positioning. All of it's in local reference frames, so the code I wrote to snap an arbitrary module onto another arbitrary module should just work forever now... and what you can't see through text is me compulsively knocking on the wood of my desk that it remains so. 

Anyway, modules work now! Look forward to modular stations and capture mechanics and blowin' up shield generators in the near future. It should be fun to crank out the rest of the models for that, and even more fun to finally get to blow them up myself as part of a mission. This is something I never planned at the start of this project. Hell, if you'd asked me eight months ago, I'd have said I was incapable of doing it. Now that I've got it pretty much under wraps, I'm glad I did -- it'll add a lot of unique gameplay options for a small time cost on the part of myself. The moral is that sometimes -- *sometimes* -- feature creep can be a good thing. And also that you should always take a swing at a challenge.