## Debris Sector Generation

The last few posts have been some dull framework stuff; infrastructure with a promise of being useful later and being vital, necessary work. What about the game itself? What have I been up to?

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/stairs.png "no comment")

I've been working on figuring out exactly how I want the "debris" sector to play, since it's the initial sector and I want it to be interesting enough to keep going. But as it's stood for the past few months, and indeed for most of the project, the generation has been pretty basic and dull - the same type of debris, over and over again, in a randomized order within a sphere. Sure, it blew up when you shot it, and you could tape someone to it or bonk 'em into it, but all that isn't very interesting. There needs to be more!

Around this time I briefly considered random generation of debris / asteroid meshes - procedural generation for obstacles - but rejected it here because no matter how much I change the shape of a given chunk of debris, it's going to be the same type of debris. Randomness does not necessarily translate well to "fun". For reference, witness every single god damn landable world in Elite: Dangerous. There's nothing there, despite each landscape being completely unique. I might go back and actually implement this later, but it's no substitute for gameplay.

So is that what we're working with here? Procedural generation? Hell no. What do I look like? A goddamned data scientist? The first thing I threw together was a high-level design document for things I wanted to see happen with the debris sector...

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/highleveldiagram.png "Not pictured: the reverse side of the paper, picturing me riding a missile straight into a sun with the caption 'HELL YEAH'. Maybe in a future update.")

...and got to work. With that in mind, let's take a look at...

### The Debris Sector

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/newdebris.png "This was really satisfying.")

Bam! Different pieces of debris!

Obviously the first thing I needed to do was come up with some more interesting "trash" to throw around - obviously it'll still work pretty much the same as prior, but different models are needed. I already had my model for the human carrier, so I chopped it up a bit and made some extra spare components out of what I had there. There's now quite a bit more variety.

Once I had that, I needed to think a little harder about my generation. This field should be a whole bunch of demolished ships, so you wouldn't necessarily expect to see it all scattered completely randomly around the place. Under the hood, my prior method came up with a bunch of points in a sphere for obstacle positions and then put stuff there. It's still similar, but now it selects an initial point in the sphere, and then some other points all around that roughly correlating to the shape of the ship. The result is that you have what really looks like a demolished "ship", broken open in the middle and splayed open. Right now I only have the one "ship" for it, so it's a bit placeholdery, but this is what we're gonna be going with in the future once I actually knuckle down and make some better damned models.

In here I also included another piece of functionality. The "engine" piece of debris will turn back on the second it gets shot, and it will quickly accelerate forwards and smash just about anything in its way. I don't think it's particularly useful, but I *do* think it's extremely entertaining and prone to cause all kinds of chaos in the environment, which is what we like to see. During testing, I did see one time at which an engine slammend straight into another chunk of debris which in turn caused a Peragus-style cascade of kaboom, which was neat.

Great! So that's my random trash out of the way, and ships should generate a little better - what about actually dangerous stuff?

### Fuel Tanks

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/fuel.png "It's orange, so you know it's dangerous.")

A classic. The destroyed ships will sometimes have fuel tanks spilling out of them. These work exactly like you would expect - you shoot them and they detonate with a massive bang. I had a lot of fun baiting the AI into a position where I could set off a fuel tank and cause some havoc. That's kinda rare, though; these are generally better for causing seriously massive explosions involving lots of debris that will pretty much obliterate anything caught near it. Good for carriers.

### Supply Crates

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/supplies.png "Free stuff!")

Supply crates, when destroyed, will add supplies or ammo to your current carrier. A nice little bonus for poking around the map a bit more. I think in the future I'm probably going to make it so that these also repair / reload your *current* ship, so you can use them as health packs if you're down a bit, but I haven't done that yet. Short and to the point.

### Mines

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/mine.png "These really need a flashing light. Soon.")

Hey, of course some of the larger ships would be carrying these! Another one that works exactly like you'd expect it to. If a ship flies too close to the mine, it detonates. Way, way easier to bait the AI into getting near, and also a bit better for weaponizing the gravity bolas, although it's a bitch to hit. It won't kill you outright, but you won't be happy.

While designing the mines, I figured that the best way to implement the "check for anything nearby" stuff was to use code I'd already written - namely, my AI rework. I quickly split off a new type of mine AI that just checked its surroundings to see if it should blow up or not, and we were off to the races. Prior to my AI rework from earlier, this would have been an absolute bitch to pull off, but instead it was something like twenty lines of code. *God,* I love it when I leave presents for my future self. The mine AI also didn't impact FPS at all, despite the fact that during testing I had hundreds of the suckers all running their checks. Score one for ECS.

### Anti-Ship Warheads

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/warhead.png "AAAAAA")

I got a nice action shot of one of these things on its way to get my ass. Some leftover ship-to-ship warheads have made their way into the debris field, and their targeting software has gone horribly wrong. If a ship - *any* ship, player or enemy - gets near the thing, it will activate and then start chasing them down, detonating either when it's close enough or after a certain duration has passed.

This was another one where my AI rework came in very handy, and I followed a similiar procedure to how mines work, although with a bit extra because these actually *chase* you. I had a lot of fun flying through and around these, and watching them deal with the AI. My favorite was flying past one then turning so the AI triggered the ship warhead, then turning around and continuing to flee. A second or so later, the AI ship sails past me and smacks into a pile of garbage, dying instantly. That was satisfying. I also had a few instances where I wasn't paying enough attention and one of these suckers tried to park in my backseat. Again, they won't kill you outright, but they'll knock you way the hell out of position and deal plenty of damage. Best to outrun or outmaneuver 'em.

### Faulty Turrets

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/sentinel.png "Yeah, it's the same model from earlier, pipe down.")

The last thing I wanted to implement was some malfunctioning turrets that had broken off from their respective ships and, much like the warheads, had their targeting go absolutely haywire. These turrets will track and shoot at any ship that gets near them, ally or enemy.

These actually led me to find a bug in my sensor code. Effectively, these turrets have a different type of hostility - since they're shooting at both friendlies and hostiles - which required a slightly different "faction" setup for them. I couldn't figure out why the hell they weren't shooting at me for awhile, despite spawning in properly, and eventually tacked it down to a chunk of my faction code that checks whether or not a given ship is a hostile. It was actually checking whether the *other* ship would be hostile instead of whether *you're* hostile to them. Of course this doesn't matter when you only have two types, since they'll be identical, but it doesn't work *here*. That was neat to find.

### Final Touches

All of these generate in clusters around where a destroyed ship might be, and then on top of all that it scatters a whole bunch of random crap around the place, just to make it interesting. This can be any kind of debris. After it gets done adding another fine layer of problems for the player, it then creates some seriously gigantic chunks of debris that you're effectively forced to fly around or demolish, which makes the actual flying far more interesting.

There's still a few spit-and-polish things I need to get done - for example, mines should have a flashing red light, as should warheads and probably the turrets as well to make sure you spot them - but those are mostly visual effects and fairly easy to get done. Obviously I need to figure out some exact numbers on balancing as well, like detection ranges, damage numbers, number of any given obstacle, and other things like that.

### End Result

The starter sector is now a hell of a lot more dangerous, with ample opportunity for the environment to kill you, or, if you're a smart pilot, to use the environment as a weapon against the enemy ships. It's way more fun to fly through, and I personally was having a pretty good time just flying through the thing and trying to deal with all the crap that's been scattered around now *while still being shot at* was challenging and rewarding when I managed to use it to pick an AI flyer off. 

Still a few tweaks to put in place, but I think I'm headed in a good direction here. Also necessary here is to adjust AI positioning so that they're not, like, *right next to you*, and figuring out a boss fight for the end of the sector. The former will be easier as soon as I tweak the HUD for objectives to allow you to see general areas that an enemy ship might be in so you can go and splatter them with your wing. This'll also allow you some more time to explore the environment. Looking forward to it.

### What's Next?

Well, given my decision to really try and nail down how the sector plays, that means I also need to get all my ducks in a row system-wise, so I need to actually head back to GUI hell and create some GUI to display dialogue. I'd also like to adjust the AI to *patrol* the sector, as well as enhance their AI a bit with some of the goose behaviors I defined earlier regarding flocking and teamwork. I also need a method of spawning in different wingmen and adding them to the ship, and I need to finally get around to the construction bay. I'm thinkin' dialogue first; that'll have plenty of impact right there. After I get done with all of those, I need some better menus in place to more accurately reflect how it's going to play. The campaign menu has had it too good for too long.

And then, once all that is in place, I get to do it all over again with the next sector to see how *that* plays! We're making progress here!