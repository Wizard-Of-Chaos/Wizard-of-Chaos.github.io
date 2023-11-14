### The Terrible Danger of "Works on My Machine"

The last few sectors are being built as we speak, and I anticipate the game to be entirely done (but unpolished) by the end of the year. After that it's gonna need another pass or two to really nail down shaders, particle effects, more weapon and ship variety, some extra tiny little details, and the multiplayer, but the actual underlying structure will be done. The sector I find myself in is the ring sector, which takes place in the dark half of a ringed gas giant.

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/ringsector.png "Spooky!")

It's dark here. It's very dark. The only light comes from your ship's spotlights and weapons, and the occasional glowing asteroid, and navigation is rather difficult against that background... and worse, there are *things* out there waiting for you. They're not the aliens you've been shooting... but they're not happy to see you all the same.

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/UNKNOWN.png "I seriously doubt anyone is going to get a good look at these, so I could have skimped on making the model, but I didn't, which is my fault for wasting time, I guess.")

There's just two more sectors after this, most of which the modelling and design for is completed, but which haven't had a pass to make them into a coherent form -- specifically, I need a final boss fight. The next sector will be a fleet-grouping sector, which will involve managing and supporting a larger-scale fleet (and some big, huge battles!) and then the last will take place in the comet shield of the system with uncapped speed and a hell of a lot more running than fighting, I think. 

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/causality.png "One of the major reasons I wanted to re-work the guns was so I could get the big, huge railguns on this ship down properly.")

A lot of this will trickle back to the player as I steadily add more crap for this final fight -- specifically new fighter designs and weaponry. Two birds with one stone.

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/hummingbird.png "The player needed an option for a strafing ship, so here's one.")

But the ring sector will be a nice detour into a slower sector before the finale. I'm very happy with it so far.

On top of that, the art for the game is almost finished as well, thank god. The wingmen look excellent, and I'm very happy with the look. Hell, some of them aren't even the wingmen. One of them is the space-marine captain you keep sending to capture stations. She'll have some things to say, too!

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/ivanova.png "'Is the name Sarah Ivanova a Babylon 5 reference?' Would you be asking that question if it wasn't?")

But none of that is what I want to talk about -- this one will be short, and will be a rather quick story, but it needs to be said.

## The Problem

About six months ago, I noticed a really weird bug with my turret code, where whenever a turret got shot down it had a chance to crash the game and cause a segfault somewhere. I rolled my eyes and went to go fix the thing, which was pretty easy, since I work in Visual Studio and it points me nicely to the offending line of code. Shortly thereafter, I reworked turret architecture entirely in terms of how they get initialized and deleted, and now for all intents and purposes they're ships with a very limited range of motion. Problem solved, right?

Not so much. Occasionally I'll send out beta builds to people for testing and general opinions on how the game works and things I should add -- and *especially* if it crashes somewhere, because I'd like to know about that specifically. By this time the turret bug is, to me, long forgotten. It's been fixed. But I was getting extremely strange reports that sometimes when a turret got shot, the game would just outright lock up. It wouldn't *crash*, it would just permanently freeze. This baffled me, and I went back through my code to make *damn* sure that the turrets were getting deleted properly. So I send out more builds after this, sure that I've licked the thing this time, because I've added a new layer of safety onto an original fix that I couldn't even replicate. I thought it was solved.

## The Frustration

But it wasn't solved. This time I send out builds again, and once again, people come back to me with reports that yeah, sometimes when a turret gets destroyed the game will just lock up. By this point, I'm weirded out, but it's to be admitted that I don't spend a lot of time shooting turrets in the game, so maybe my replication efforts have just been garbage. The bug's clearly in the code, even if some people are getting the freeze and others aren't (myself, most irritatingly, in the latter category). So I decide to rework one of the sectors temporarily so it *only* spawned turrets, so I could just gun them down one after another and check it that way. By the way, I'd like to brag at this point that even with literal hundreds of AI and sensors running at once, I was still getting 100+ FPS in the game. Hell yeah.

But it's for naught. Y'all, I spent *two god damn hours* sitting in my own game, shooting at turrets. I had to keep reloading the game because I kept running out of ammo. And not *once* did this bug replicate.

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/smashpc.gif "I can't believe I make a living doing this shit.")

By this point I'm pissed off and frustrated that this damn thing is just refusing to manifest itself. So I set up two approaches to this problem, because clearly this isn't something I can do on my own hardware. I grab my music guy, who I *know* has had the bug, and get them to set up the dev environment on their machine (most of which they had already, due to needing access to the various repositories). At the same time, I grab my backup PC and start working on my dev environment there to get that back up to code, since I switched machines awhile back and don't use that one anymore for anything except playing movies.

## The Unrelated Problem (?)

For some reason it's impossible to get my dev environment to compile the program, even after repeated fresh installs on the other hardware. It keeps throwing me an error from my Ogg Vorbis library saying that it's missing a critical function and can't link properly. The function in question is, weirdly, a part of the god damn MSVC environment, which should come *with* Visual Studio. I check in with the other guy and they confirm that yep, they're getting the same compiler error. So now I'm confused -- the two environments, the one on my backup PC and on my main PC, are absolutely identical, and I keep running up and down the stairs in my house to check this. The other off-site environment should be identical as well. So what the hell is wrong?

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/smashpc.gif "I can't BELIEVE I make a living doing this shit!")

Since it's a linker error, that means the library in question is shitting the bed somewhere. I had it compiled as a static library and roped in with the rest of the executable. I can't think of anything else to do, so I recompile it as a dynamic library, bring in the .dll to the project, and reconfigure the project slightly to use the .dll instead of the static library.

It compiled.

It not only compiled, it ran. I set out to replicate the turret problem... and couldn't. Once again I spent an hour attempting it before I directed my musician to use the new build environment with the .dll to compile it and attempt it themselves.

They confirm that it works... and also that they can't replicate the problem anymore. It's just not there, hundreds of turrets being shot and the game just runs perfectly. They definitely had the freeze before, but it's gone now. It's gone for *good* this time.

## The Moral

The moral here is blatantly obvious to anyone who's had to deploy software. The worst thing you can say about your program is "It works on my machine". I'm going to go even further than that and say that if you're deploying your software to more than like, four people, and you haven't tested it on at *least* two separate pieces of hardware with separate compiles and installs, *your software has not been tested.* I have no idea why the game was even compiling on my current hardware, but it shouldn't have been, and it took a complete recreation of the dev environment and two other entirely separate computers to be able to track it down -- and the solution wasn't even related to the turret code except by sheer coincidence. Always, *always* test on more than one piece of equipment, or bugs like this are going to kill your software before it's even had a chance in the wild.

## What's Next?

Finishing off the last two sectors, with all the models and resources... and plenty of testing on multiple sets of hardware. We'll see you soon.