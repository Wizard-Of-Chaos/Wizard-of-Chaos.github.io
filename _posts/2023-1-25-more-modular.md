### Modular Station Math Class

If you're just now reading these, what I've been working on for a few weeks is building up some modular stations as a concept, like the ISS, but cooler and with guns. My most recent achievement was finally coming up with a system that could randomly generate these things from their constituent parts so I could scatter them around the map and what-not.

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/fullyrng.png "It's like the ISS, but in spa-- wait.")

This, inevitably, required quite a lot of math to be able to do properly. [Last time I managed to get the constraints working properly](https://wizard-of-chaos.github.io/2023/01/08/modular-stations.html), although I later found out some of my actual code there was flawed, but I hadn't gotten down spawn points properly for each of the individual sections, so spawning the things was a hot mess. They looked like piles of station spaghetti.

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/spaghetti.png "Yeah, I think something's wrong.")

The way bullet works to save time on calculations is that some rigid bodies are "inactive" when they aren't moving; this means that it just straight up ignores calculations with that rigid body since it's not important. I tended to spawn in station modules as "inactive" for awhile just to see how their initial positioning worked. Now, what *happened* was that as soon as you activated one of these modules here, the constraints would "snap" the whole thing into place and it looked like a normal station. It was *super* weird to look at.

> "If you're not big into game physics, the term "constraint" might be nonsense. It's pretty simple; a constraint is a chunk of code you send to the engine that informs it about the way two objects behave. For instance, I might set up a constraint that says "I want object A and object B to not get more than 15 meters apart", and the engine will take steps to make sure that doesn't happen - usually by applying a huge force to make sure that they stay under 15 meters apart.

Now, a lazier developer might simply have spawned the things as active and then kept the loading screen up for an extra two seconds while the constraints did their thing. But, hell, I'm better than that. It's just a number problem; it can't be *that* hard to figure out where they're supposed to be...

## Yes, But Also No

This took an embarrassing amount of time. Let me preface this by saying that I took exactly *one* computer graphics course as part of my CS major, so vector-based math and rotational geometry are a bit foreign to me and I've mostly had to pick it up from scratch (although hilariously my initial method of moving ships around *directly* stole code from my Boids project in senior year, and it also proved to be the basic idea for my AI).

"Are foreign". That is to say, *were* foreign.

Thankfully, my esteemed partner on this project (who is currently hard at work coming up with some networking protocols to allow for co-op) has a master's degree in this, and is *far* better with the concepts involved. He was kind enough to humiliate me for several hours while explaining exactly what the hell was going on with rotations.

To start off, let's pick a point in space. Let's give it a rotation too. Irrlicht wants this thing to be in degrees as Euler angles, so we'll just call it at 180 around the X axis.

```cpp
vector3df pos(54,10,900);
vector3df rot(180,0,0);
```

So this is where we want to throw the basic module. Cool. My basic module here is an "X" shaped reactor module.

[alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/reactorblender.png "Stolen from last post")

As I explained [last time](https://wizard-of-chaos.github.io/2023/01/08/modular-stations.html), in order to be able to attach things to this sucker correctly, we need to know what direction any module attached to it is facing, and we need to know where the attachment point is.

Orientations in 3D space can be defined by two vectors: an "up" vector defining what the local 'up' is for that object, and a "forward" vector defining what the local 'forward' is for that object. We're storing both of those in local coordinates.

# What The Hell Is A Vector? {#wtfvector}

If you already know this stuff, feel free to skip ahead.

Basically, everything in 3D space can get defined by a set of three values, and the term "vector" gets thrown around interchangeably a lot and it can get a little confusing to follow if you don't know what the hell the context is. Engines like Irrlicht use [the term 'vector'](https://irrlicht.sourceforge.io/docu/classirr_1_1core_1_1vector3d.html) to represent a whole lot of things, and Bullet3 [does the same thing](https://pybullet.org/Bullet/BulletFull/classbtVector3.html).

I don't propose to explain vector math in a single damn blog post, so this is just a cheat sheet for whatever I'm talking about.

-**Vectors** are a set of values used to define... something. Whatever you want, really. In most game engines the vector has 3 values.

-**Direction vectors** define a given *direction* in 3D space. This is a *unit vector* (I never use this term and just call it 'normalized' most of the time), which means that it has a length of 1 ('length' just defines how big the thing is compared to other vectors). If it helps, you can think of it similar to rotations: if you turn 3 degrees left, 10 degrees up, and 25 degrees forward, you will face this direction. That's not how it works [under the hood](https://en.wikipedia.org/wiki/Unit_vector), just a simplified version.

-**Position vectors** are a simple position in 3D space. This is generally how you keep track of where something is on the map. It says where the object is along the X, Y, and Z axes. Position vectors are equivalent to a direction vector multiplied by some value (or magnitude, or length, or whatever) pointing away from the origin (0,0,0 in space).

-**Velocity vectors** are used to define how an object is *moving* through said 3D space, and are a directional vector with a specific magnitude -- I am moving in *this* direction at 40 miles an hour. If you *normalize* a velocity vector, you reduce the length to 1, and you end up with a direction vector to point which way the object is going. This is true of angular velocity, too, which is where you determine how quickly an object is rotating and in what direction.

-**Rotation vectors** are a Euler angle thing that store pretty much what I said earlier -- what the rotation of the object is along each of the global axes X, Y, and Z. These are easy to think about, but come with some problems, namely 'gimbal lock' (where you bring two axes into alignment with each other and lose the rotation). These can be represented in *degrees* (0 to 360) or *radians* (0 to 2pi).

-**Local space and global space** are two different frames of reference for an object. The easiest way to explain it is that if you're on the couch facing the TV, and I'm standing in front of you facing *you*, my local 'forward' is markedly different from your local 'forward'. In *local coordinates,* my 'forward' is going to be the direction 0,0,1, and so is yours, but we'll have different ones when translated to global coordinates (where forward is, I don't know, magnetic north or something).

-Lastly, when I talk about **orientation**, I am generally referring to the combination of position and rotation to define where the object is and how it's facing. This can be represented either with the above vectors I mentioned, or with a **quaternion**, which I [poorly explained in the prior post](https://wizard-of-chaos.github.io/2023/01/08/modular-stations.html), and which is a set of *four* values used to define an object's orientation. People don't like when you refer to quaternions as 4D vectors, so don't do that.

-If you want to know more, you can read up on [Euclidean geometry here](https://en.wikipedia.org/wiki/Euclidean_vector). Again, that's just a basic overview of how crap gets represented.

# Back To It

Right, so like I was saying, I can get the orientation for an object with the direction vectors for 'up' and 'forward', which should be perpendicular to each other. Each connection point has an up and an an 'out' to say what orientation matches up with this particular slot. We need to actually match *two* orientations here - we need to make sure that the two modules have *opposite* 'out' directions, and that their 'up' vectors are aligned. This should bring them into the correct orientation with each other.

So how's that get done? Well, first we need to take those directions *out* of local space. All these directions are stored on file as local coordinates because I don't know *what* the hell the game is going to be doing with them when they spawn. We need to translate the old module's directions by its current orientation. Thankfully, Bullet3 has made this easy to do.

```cpp
oldSlotUp = oldSlotUp.rotate(oldOrientation.getAxis(), oldOrientation.getAngle());
oldSlotOut = oldSlotOut.rotate(oldOrientation.getAxis(), oldOrientation.getAngle());
```

The rotation I'm doing there is taking the *axis* of the old orientation and the *angle* around that axis, which is an entirely different way of defining rotation using quaternions that I don't feel like re-hashing. Basically, you can *also* define rotation by defining an *up* vector and then rotating around that up vector on a perpendicular plane. Bullet3 and Irrlicht do the math for me on the 'how', so I won't explain in too much detail.

Great. That's our old slot's orientation fixed up. Now we need to bring it into alignment with the new slot's orientation. Keep in mind that the new module *doesn't exist yet* because we're trying to determine the *position* and the *rotation* that this module needs to be spawned into. So our new module's orientation is in local coordinates. First, we need to align the up vectors.

```cpp
btQuaternion ups = shortestArcQuat(newSlotUp, oldSlotUp);

newSlotOut = newSlotOut.rotate(ups.getAxis(), ups.getAngle());
newSlotUp = newSlotUp.rotate(ups.getAxis(), ups.getAngle());

```

'Ups' here represents the rotation that is the **shortest arc quaternion** between two vectors, where the quaternion represents the rotation needed to get from the first vector to the second. Once we have that rotation, we rotate around the new module's out and up vectors accordingly. All our vectors are now in the appropriate reference frame.

Next, we need to bring the 'out' vectors into opposite alignment.

```cpp
btQuaternion outs = shortestArcQuat(newSlotOut, -oldSlotOut);
```

Again, this is fairly simple - to get the two to be in opposite alignment we simply get the rotation from the one to the negative of the other. To get the final orientation for a new module, we just need to multiple the `outs` and the `ups` quaternions together, and again, I'm not gonna go into the nitty gritty of how to actually do that.

```cpp
btQuaternion finalRot = outs * ups;
```

Done. Here's our orientation that we can spit out to whatever format we like (Euler angles or something).

# Position Of The Module

We're doing it in this order because the position *requires* the rotation in order to be calculated at all. The positions for each slot are, again, in local coordinates (how far away they are from the center of the object) and do not take into account the object's orientation. So, here, we need to adjust those values yet again. Thankfully, this is a lot easier to do. We do the same rotation by the axis and the angle.

```cpp
vec.rotate(orient.getAxis(), orient.getAngle())
```

We grab the old module's orientation and rotate it, and then we calculate the new module's orientation and do the same thing. At this point we're just using algebra, since we have most of the values we need. Solve for X.

`(oldPosition + oldOffset) = (X + newOffset)`

```cpp
return ((oldPos + oldSlotPos) - newSlotPos);
```

Donezo. We use these two values to snap the two modules into place with respect to each other. Great! The problem is that the constraints operating to *keep* them there are operating in local coordinates, which was a whole separate thing that I actually got wrong (my quaternion was a bit off and using the wrong values), but which has since been fixed. I think. Mostly.

## That Looks Easy!

Yeah, but it took me a couple weeks. Thinking in this manner can get a bit counterintuitive and figuring out how to define exactly what I wanted out of my rotations took some brainpower. It's done now though, thank god, and I will never have to look at it again until I find out something *else* that I did wrong. This code is going to get maliciously abused for the entirety of the supply depot sector, so I'm sure there's *something* slightly off with a piece of it. We'll see how it goes. Next time I'll probably talk more about how to get these suckers to actually dock to each other. 