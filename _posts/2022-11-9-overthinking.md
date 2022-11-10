## Overthinking Your Code

I've been stomping on a lot of bugs recently, trying to get my game as ready as possible for the [competition I'm submitting it to](https://igf.com/), which means fixing all the bugs that I've habitually been ignoring, [setting up methods of actually getting to the game](https://baedsdevelopment.itch.io/extermination-shock), creating some custom saves, stuffing in as many extra pieces as I possibly can and [setting up a neat little trailer](https://www.youtube.com/watch?v=UlmLvJMVxY4) for the game. Overall, been fun, if a bit hectic.

Bugs are weird, as a developer. You, as a user, will tend to ask questions such as "How the hell could this bug possibly slip past the developers?" And the answer is, honestly, it probably didn't. I've been having a consistent bug with the game crashing every time you try and launch a game while currently in a different save - i.e., if you try to start a new game while currently in one, or try to load an earlier save. So I marked it down, couldn't immediately see a problem with it, and simply just stopped trying to do that ever since I implemented the feature initially. It's like trying to keep your weight off an injured leg. You *know* it's injured, but you've grown accustomed to limping a bit. Then, to stretch the metaphor, someone else takes over your body for a bit, puts some weight on the leg and snaps the damn thing in half.

One of the areas where I've been limping for awhile and ignoring is actually somewhat visible in that trailer (since I only fixed this damn problem today, *after* I threw the video together). Bullet trails just simply did not follow the direction they'd been assigned - they just stayed on the ship's "forward" vector and didn't rotate at all. It looked fine most of the time and only really looked weird from a few angles (and it looked *really* weird with shotgun-type weapons, but I didn't use those much - keeping the weight off the leg), so I mostly ignored it.

To fully explain what the hell I was doing wrong, let's set the wayback machine for...

### January 2022

Early in January, I got really pissed off at the state of space-shooter style games and started developing my own. Incidentally, during this time, I also shot out an application to Cloud Imperium (the guys who develop Star Citizen) as a fresh college grad and got turned down, but that's neither here nor there. At the time, the "game" looked like this:

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/jan2022.png "Hey! That's a copyright infringement!")

...and I was just implementing the ability to shoot guns, having finally figured out how to make the little ship move around. I was entirely new to Blender, texturing, game development, all of that. The first spaceship model I ever made was a shitty rip-off of a [Hellcat fighter](https://www.wcnews.com/wcpedia/Hellcat_V) and I made the worst gun mount known to man to go alongside it.

So how do projectiles get rendered here?

### Billboards

The same way they did back in the 90s, and actually in a few games up to now. You see the big, white light in that image? That's a "billboard" style effect. Billboards are effectively an image that, when rendered, is always facing toward the player. A great example of how these get used (and one of the first implementations) is the original Doom.
 
![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/doom.png "Hey! That's a different copyright infringement!")
 
The enemies are sprites that are always facing toward the player, no matter what the player does. That was all they could do back at the time. In some ways, we're still not beyond this - most games will use a billboard somewhere, usually as a particle effect of some description. If you're curious, this is the [Irrlicht docs on billboards](https://irrlicht.sourceforge.io/docu/classirr_1_1scene_1_1_i_billboard_scene_node.html). For a spherical particle effect, this works great. It blasts out a red ball of light and that was my first weapon - the plasma blaster. The rest - health pools and whatnot - was details. The effect was there.
 
Eventually (April) I got to making more types of weapon with different damage types and effects, and that's about when I made the first "kinetic" weapon, which was an LMG-style gun that made a really irritating noise and spat out a ton of smaller metal slugs. 
 
![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/crappybullets.png "Yeesh. This actually hurts to look at.")
  
The billboard *worked* here, but it wasn't really satisfying. The individual bullets were barely noticeable, since they were smaller than the plasma balls, and more importantly when you picture a bullet you don't picture a single little ball. You picture something closer to a tracer round, with streaks following behind it.
 
![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/dragonsbreath.jpg "This is a dude using Dragon's Breath rounds out of a shotgun. They are ridiculously cool looking, and I've always wanted to own a few rounds.")
   
Okay, maybe not quite that dramatic, but you get the idea. So how the hell do you implement a particle streak? I can't just *extend* the billboard - that just stretches it left or right, or up or down.
   
### Axial Billboards

This is where another modern 3D feature comes into play - the axial billboard. An axial billboard is similar to a regular billboard, except that instead of always facing the player, it's a flat plane that is always facing the player but at a specific *angle.* If you use an axial billboard to create a laser beam, for instance, if you view it from the right it'll look like a beam, and if you view it from the left, it'll still look like a beam because the thing rotated to face the player. If you face it from either the front or the back, it's going to flip around wildly as it tries to determine the angle the player is viewing it from, but generally looking straight into a laser is bad for you, so we don't do that. 

Axial billboards are used *everywhere*. A great example in most games is leaves - they're 2D planes with a leaf texture on them that are always facing the player to make the tree look fluffy (although sometimes they won't even bother with this and just do flat planes always facing a specific direction). Really shitty tree textures in Roblox or something will pull this trick.

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/axialbb.gif "Pew!")

This was my first crack at implementing an axial billboard (Irrlicht, surprisingly, has no built-in feature for these, so if you want one, you better write it yourself), and it looks fine enough. You'll notice I don't *move*, though. My axial billboards kept screwing up *massively* with the ship's rotation, and bugged right the hell out. More to the point, they had the problem that from specific angles (i.e. looking directly at them) they didn't have any "volume" - the bullet trail would seemingly disappear.

The former I had no idea how to fix, so I capped it so that the bullets would always inherit the ship's rotation and left it at that for months and months. The other problem I fixed by creating *more* billboards, albeit at slightly different rotations, and capping the trail on the front-end with another bullet billboard, this one always facing the player. That gave it some volume from any angle. This is getting hard to explain, so look at this diagram.

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/axials.png "Goddamned MSPaint.")

This is about what a bullet trail looked like. Each of these intersecting planes was slightly transparent around the edges and had the same trail texture on them, so it looked like a solid beam of light. You can sort of see what that looks like in this screenshot:

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/bullet.png "Aaaack! Limitations!")

If you look at the right side, you can see a little bit of where the planes intersect, and if you look at the left, you can see the billboard capping off the bullet that was leaving the "trail". This is actually a thing you can do with a mesh instead of billboards, too; if you create a little mesh that's just a few intersecting planes that all have the same texture coordinates, you can get the same effect. Nearly every beam effect you'll ever see uses some varation on this method.

Thing is, the rotations involved with these planes was giving me *fits*. Every single goddamn thing I tried made the problem worse, or looked like shit, or both - every variation on rotating planes around either led to batshit insane bugs that caused starburst-like effects sometimes or just outright failed to work and left me with the original problem. But it's clearly possible - everyone else has done it before me, so there's *something* I'm doing wrong. For months, and months, I would occasionally try a fix here or there, have it fail to work, then shrug and go back to it. It's just a little particle effect issue, after all, right? I can keep the weight off that leg pretty easily. I was seriously considering dropping everything to pick up HLSL shaders, which work with Irrlicht and I intend to learn after I have a few more assets, just so I could create better trails for the bullets here. At least my other effects looked decent. 

In particular I was pretty proud of the engine light. I wrote this out early on - you can see it in the April screenshot.

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/engine.png "The engine light is on! Check the oil!")

That's a neat effect. It's not perfect, but it uses a texture animator and some volumetric lighting that stretches out depending on how fast the ship goes, and gives a little engine-lookin' effect. It looks really good at high speeds - almost like a laser. 

Exactly like a laser, in fact. 

A nice little trail behind the ship. Like a tracer round might leave.

...hey.

### Volumetric Lighting

I had a friend who used to refer to jokes that you took awhile to "get" as "tube-light jokes" (him not being very good with English yet and not knowing the term for a fluorescent light), since those take awhile to come on fully and hit full brightness. This was a certified tube-light moment, right here, when I caught myself staring at that engine effect, and it was staring back at me, something I looked at damn near every day. The lightbulb slowly switched on.

Yeah, I actually had this fucking problem figured out seven months ago and just completely failed to notice. The engine effect uses a volume light, which you can [read about here](https://irrlicht.sourceforge.io/docu/classirr_1_1scene_1_1_i_volume_light_scene_node.html) for the Irrlicht implementation, but the explanation of it is pretty simple. What a volume lighting node does is it takes a "light" effect and actually fills the space around it with tiny little meshes or textures that represent a portion of that light. In my case, that engine light uses a series of textures in a row and adjusts the "light" accordingly.

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/engine3.png "Engine texture.")

This is one of the textures in question. It's just a flat, spherical green ball, which the volume light effect stretches out and turns into a cylinder of light. I didn't have support for a texture animator on my guns (although it could easily be added if I wanted to), but that was fine, since I only had the one particle anyway - a bullet, or a blast of plasma, or something of that nature.

Nodes are way easier to rotate on their own. I flipped this node so it was facing the same direction as the firing direction given to the gun (fixing a weird Bullet3 bug along the way- I'd set it up so that projectiles used "sphere" bodies, but never bothered to change their rotation in any way because I never had a need), and we were in business. I did the same thing to my plasma blasts, too, not just the bullet trails.

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/newlights.gif "It was really irritating how easy that was.")

Not *only* does it look better, it also aligns perfectly with the flight path of the bullet and improves massively on the simple billboard effect that energy weapons were using before, and also allows me to create better effects for railguns and physics weapons and pretty much every goddamn thing that uses a projectile, since while I was at it I also introduced scaling for each given effect to dictate how big the effect in question was. It took me about an hour total.

And *that* is overthinking. Seven goddamned months of doing a particle effect incorrectly due to mistakenly assuming I needed the wrong tech, or overthinking rotations and various methods of handling things up to and including a shader when I had *already written something that did exactly what I needed.* The moral of the story is to never try to learn anything, ever, and never pick up any new tools, and don't learn shit for any reason. I'm joking, obviously, but really, sometimes it's way easier to just step back and try to find a solution from your previous work. There's usually something of value there. And sometimes it's a good idea to put a little weight on that leg, you know, to see if anything's changed, and to figure out if you just need to visit the fucking doctor already.