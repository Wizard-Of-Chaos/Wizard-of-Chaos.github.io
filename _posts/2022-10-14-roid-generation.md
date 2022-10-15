## Asteroid Generation and Entertaining Bugs

To start off: We've got a mess hall! We've got a kill-board! We've got new and spicy upgrades! We've got new and exciting wingmen, and dialogue to go with them! We've got wingman retrieval and salvage missions! We've got a new ship for the humans, and a bunch of new weapons! We've got a boss-fight! We've got a neurotic comms officer to talk to! We've got improved aim for the AIs, and a whole mess of bugfixes! The first sector is done! We've got an actual settings menu! We've got new explosion graphics! None of these are particularly interesting to talk about, though, development-wise aside from "we have them now", so I'm going to post a few screenshots and get on with things.

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/kate.png "Kate's not happy to be here.")
![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/messhall.png "Cat Cheadle and Sean Cooper are on the board.")
![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/newguns.png "The new guns offer a variety of playstyles.")
![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/anubis.png "Turns pretty good. Guns are on the bottom.")
![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/roids.png "'Roid rage!")

Whoa! That last one was actually the thing I wanted to talk about. The others had some funny bugs, but mostly they're just fleshing the game out - no new algorithmic design there. It's been really great to get the game into a playable and fun state where you can fiddle with loadouts to your heart's content, and I'm very happy with where it's going, to summarize the rest of it. Now, time to talk about rocks.

### How To Make Asteroids Interesting?

[To paraphrase myself from a few weeks ago,](https://wizard-of-chaos.github.io/2022/08/26/debris-sector.html), random generation is not, in itself, fundamentally interesting, to the continued outrage of Elite: Dangerous players. Sure, I can generate a big-assed field of rocks, but what the hell's the point if they're just there to be background noise? Or don't do anything or impair the player in anyway? Might as well set up a big damn skybox at that point.

From the get-go, I was pretty sure of the direction I wanted to take as far as *design* for the 'roid sector goes. The debris sector has an actively hostile environment. Things blow up when you get close, shoot at you, go flying off into the void, or even chase you down, so you have to watch your step (or, y'know, trip the enemy fighters up and send them flying into a mine). I wanted the asteroid sector to be *passively* hostile by comparison - it doesn't care that you're there and will make no effort to avoid you. The debris sector's environment is fairly static, so it's like stepping around a minefield (although hopefully not that nerve-wracking - it is the starter sector, after all). The asteroid sector will be more like trying to play dodgeball. The asteroids are more like baseballs, but you get the idea. If you don't, here's a helpful graphic.

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/randy.gif "Randy Johnson is one of very few people to have killed something on live television. Fear him.")

Point is, I want the asteroid sector to be that much more nerve-wracking. I want the player to be looking around to see if there's a rock coming. I want them to be able to barely scrape by a rock and force the AI to collide with it. I want them to weave in and out of asteroid belts and tape enemy fighters to particularly fast-moving ones. And then I want some other crap on top of that fundamental concept.

But how do you make a decent looking belt? How do you get it moving in a reasonable fashion?

### Asteroid Belt 101

[A bunch of fucking losers](https://en.wikipedia.org/wiki/Asteroid_belt) will tell you that asteroid belts aren't actually all that dangerous and you're unlikely to collide with any of them due to how thinly scattered they are. The hell with them. This is fiction. We're playing by Star Wars rules. Asteroids are all over the damn place and *you will hit them.* Or, I guess, more accurately, they'll hit *you.*

My first thing to do was come up with a couple other asteroid models and textures just to make it look a little more interesting, as well as add in basic functions that I didn't have before to initialize a given object with some velocity and angular velocity, since I hadn't needed to do that before now.

### Stupid Bug No. 1

When writing out the initial angular velocity feature for setting up a rock with some movement, I figured that I could just toss in a random rotation vector (like I have for the other asteroids) and then set the angular velocity to that rotation vector to make it move. Trouble is, I forgot about unit conversion.

A rotation vector, i.e., the vector that stores how much an object is rotated along each axis, might look like this in Irrlicht:
```
vector3df rotation(245.4f, 10.f, 345.f);
```
Which is to say - a value from -360 to 360, in Euler angles. Simple, right? A circle has 360 degrees, so this is easy enough.
A velocity vector in Bullet3 uses radians, so that might look like this:
```
btVector3 angularVelocity(-1.1f, 0.05f, 1.56f);
```
A radian is a value from -PI to PI. If an object has an angular velocity of PI, it's completing one rotation every second. I was initializing things with angular velocities not ranging from -3.14 to 3.14, but ranging from -360 to 360. This is bad for a variety of reasons, first and foremost being that asteroids shouldn't complete several hundred fucking rotations every second. This was quickly fixed.

### Back To Math

Cool. So we've got these. Second thing to do is come up with a basic field of asteroids that are moving in random directions with random angular velocities. I already have the features for getting a random spot in a sphere used for the debris sector, so let's do that and put some spin on it.

This was easy enough to do, but it looked really, really disturbing and I don't have a good clip to properly show why. It was like watching a bunch of bacteria. I showed it to a few people and most of them agreed that yes, it looked incredibly unnatural. Here's where we can go back to those [huge nerds from earlier](https://en.wikipedia.org/wiki/Asteroid_belt) for some advice. Most things tend to orbit in the same direction, and asteroids are contained in a *belt*. That means my asteroids should all be moving on roughly the same plane. Sure. Let's set that to the left and right of the player (moving across each other, sometimes, like two-way traffic, just to dunk on the astro-nerds) and then muss those directions up a little so it's not perfectly uniform.

Cool. So now we've got the basic idea of a player crossing the street and playing frogger, but in space, and they're also being shot at. How can we make this a little more interesting?

### Big, Huge Rocks

The asteroids so far are moving in swarms going left and right, and they're all moving in similar directions to their friends, and they're all sized abouuuut the same as their friends with some variation. There's nothing *huge* to get in the way. Let's consult those [unwashed geeks](https://en.wikipedia.org/wiki/Ceres_(dwarf_planet)) a third time for some inspiration - a vaguely spherical, king-sized rock. Just for funsies, let's give our huge rocks little belts of their own!

That last one proved to be a bit of a stumbling block, because I didn't have any functions for generating a point in a torus, and had to write some. Thankfully, this was a solved problem, but it was math I hadn't poked too much at. To explain, let's start with a quick diagram.

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/torus.png "MSpaint really set explanatory graphics back several decades.")

Let's say we want our asteroids to generate in this donut. I know it doesn't look like a donut because it's a flat MSpaint drawing, but shut up and trust me - it's a donut, dammit. Generating it in a sphere was fairly easy: effectively I pick a point from -1.0 to 1.0 on each axis, then I multiply it by the radius of the sphere I'm trying to get. Then I check to see the distance from that point to the center, and if it's more than the radius I throw the point out and try again. This algorithm requires about two runs to get a point in the sphere, since what it's effectively doing is imagining a box, picking a point in the box, then throwing it out if it's not in the sphere.

### Why Not Just Pick A Point In The Sphere?

Shut up and [read about it yourself](https://karthikkaranth.me/blog/generating-random-points-in-a-sphere/) if you're so damn interested. The case I'm describing is the first one in this article, and while he comes up with a more interesting solution that only requires one run, the first method is simpler and I *think* actually takes less time computationally (square roots are expensive as hell). I might be wrong on that, but I *am* very lazy, so we're sticking with the first one.

### Fine

As I was saying, that's how you get a point in the sphere. In the case of the torus, we have a few more numbers that we need to consider.

1. The inner ring's radius
2. The outer ring's radius
3. The axis the torus is on

First thing we need to do is find a point in the dead center of the torus's ring - the point on the line in the little diagram. We can do that by picking a random point on the circumference of a circle with that ring as its edge.

```cpp
	f32 rad = (outerRadius - innerRadius)/2;
	f32 radiusToTorusCenter = innerRadius + rad;
	f32 angle = static_cast<f32>(rand()) / (static_cast<f32>(RAND_MAX) / 6.28f);
	f32 x = sin(angle);
	f32 z = cos(angle);
	vector3df centerPos(x, 0, z);
```

Remember how I whined about radians and degrees earlier? Welcome back to math class, asshole. Programming is just applied math. Get used to it. 

You'll notice how this is taking a point within PI x 2, which gets me a random point on the circumference defined by the center of the torus. I multiply this by the radius of the torus, and we're in business. Now that I have my center point *there,* I can re-use my "get point inside a sphere" code and get a point within the radius of the actual "donut" bit, which is guaranteed to be inside the torus. Rotate this around by the vector that defines "up" for the torus ("up" points directly out of the hole in the torus), and we're in business!

### Wait, Why A Sphere? Wouldn't A Flat Circle Work?

Yeah, it would, but I already had the sphere code and I didn't feel like writing out a "get point in circular plane" function, so we're going with sphere. It's effectively the same thing. It *might* have some kind of mathematical bias included by using the sphere instead of the torus (causing it to clump around the central line, maybe) but none of the tests I ran on it caused any weird-looking formations or anything, so I think it's pretty much equivalent. Are you done nitpicking?

### Never!

So now I can generate things in a belt around other things. Cool. If we scatter some loose rocks around on top of the swarming asteroids going in random directions, create some Ceres-sized big rocks with their own little belts, we've got ourselves a fairly interesting looking belt that's fun to fly around and gets dangerous sometimes. The trouble here is the one that I initially stated, though - random generation is not inherently interesting. Sure, we've got neat formations, and the asteroids themselves pose a decent threat if you're not paying attention, but it's all the same *type* of danger. Let's go back to the initial gif to explain the danger.

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/randy.gif "It's sort of mesmerizing. The throw, the puff of feathers, the confused batter...")

For every obstacle here right now, the danger is the same - you are a bird, and Randy Johnson has just thrown a fastball at your head. The strategy is the same - you get out of the way or you shove someone else in front of the fastball. To mix this up, we can scatter some of the debris elements around as well (mines and ship missiles, around wrecks maybe) as well as the old classic of having explosive rocks. On top of that, I'm going to try and get some gravitational anomalies down that form constraints on anything that gets too close, and maybe some clouds that screw with sensors in preparation for the next sector, which will be a gas field. Those will be fun to do!

### I'm Bored Of Math, Tell Me About Some Bugs

Sure.

So when adding in new wingmen, that exposed a whole field of bugs. The weirdest one was the fact that all wingmen appeared to be using the player's ship, which I'd never noticed due to the fact that I only had *one type of ship*. That was hell to track down.

Another one is that, since I added the killboard, there's a new "feature". The check for damage looks like this:
```cpp
				if (last.from.has<StatTrackerComponent>() && last.to.has<ShipComponent>()) {
					auto stats = last.from.get_mut<StatTrackerComponent>();
					++stats->kills;
				}
```
Basically, to add a kill, it just checks to see whether or not the thing that got killed has a ship component, and if so, it qualifies as a kill. You'll notice this doesn't make any mention of factions. This means that friendly fire counts as a kill.		

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/kboard.png "Definitely a threat eliminated, alright.")

Yeah, I'm not fixing this one. It's too funny.

A really irritating one that's going to send any back-end developer reading this into a frothing rage was some code misplacement. I hadn't messed with the loot system for months, and I found the code for actually generating loot rolls hiding in the code for the *loot menu itself.* Code does *not go in the UI elements.* Structural changes don't go in the GUI. You wouldn't jam a bunch of rebar straight into your window, and you shouldn't do that in your code either. That got fixed in a frothing rage.

On top of that, re-doing the settings menu, some of the wingmen mechanics, and the loot rolls meant I was looking at code that got written several months ago that was still using massive enums for menu elements, which is how Irrlicht initially *wanted* me to do it, but I threw all that out when I implemented a callback system. Or, well, y'know, *thought* I threw it all out. Some of the cockroaches got cleaned out from under the couch. Thankfully, this wasn't affecting anything function wise, it was just irritating to look at.

### What's Next?

Finish off the 'roid generation and the settings rework, implement more wingmen, guns, enemy ships and objectives available, write more dialogue, and then get started on the next sector and figure out just what the hell works for a gas field as far as environment goes. I'm not sure how much of it will get posted about necessarily, given that it's only really fun to write posts about design processes or interesting quirks of development, but it's pretty cool anyway. The game's pretty fun to play right now, even with limited loadouts.

Things are going pretty well! There's always some new feature to get to. Never the slightest peace, but I'm sort of comfortable with that by now.