### Conceptualization: Encounter Design

Work continues on making the game actually look good, including new and fun menus as well as shader updates and in-game skybox updates! The game actually starts to look *good!* Astonishing.

Most recently it's been the loadout screen. I still haven't shuffled the buttons exactly where I'd like them, but the background is excellent.

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/loadoutScreen.png "My god! Actual shade!")

On top of that, the map screen also got a facelift.

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/mapscreen.png "Whoa! The characters don't look like shit!")

Featuring the previously featured art for Steven and a re-worked UI as well as better UI coloring. Some of the buttons and fonts and such might still get changed around a little, but this is just about what I want. The dialogue screen got a similar facelift.

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/dialoguescreen.png "Someone's been playing Disco Elysium...")

Again, this is subject to change (that top bar needs to have a color change and a font change), and it needs some animations and scrolling text to make it really work, but again -- far and away improved.

Last and *certainly* not least, I'm finally getting my shit together both with environment design and with skybox design.

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/ingamescreen.png "Clooooooooouds!")

Again, the HUD is subject to some color changes or redos to make it fade less into green, but hey, there's a real skybox now, and my shaders are running quite excellent well -- better than the built in shaders, which is a first for me, actually. But the design here is what I want to talk about today.

## What Design?

The last time I talked about sector design last August, [I'd just gotten implementations working for various types of debris](https://wizard-of-chaos.github.io/2022/08/26/debris-sector.html) (It's been almost a year since I wrote that post, jeez, I need to get a hobby-- wait) and was looking towards other crap besides aesthetics and actual, functional design. As I recall, my generation was effectively placing crap in a sphere. I also railed against random generation not being inherently fun.

> Around this time I briefly considered random generation of debris / asteroid meshes - procedural generation for obstacles - but rejected it here because no matter how much I change the shape of a given chunk of debris, it's going to be the same type of debris. Randomness does not necessarily translate well to "fun". For reference, witness every single god damn landable world in Elite: Dangerous. There's nothing there, despite each landscape being completely unique. I might go back and actually implement this later, but it's no substitute for gameplay.

Later that year in October, I [again railed against it](https://wizard-of-chaos.github.io/2022/10/14/roid-generation.html) when talking about asteroids.

> To paraphrase myself from a few weeks ago, random generation is not, in itself, fundamentally interesting, to the continued outrage of Elite: Dangerous players. Sure, I can generate a big-assed field of rocks, but what the hell's the point if they're just there to be background noise? Or don't do anything or impair the player in anyway? Might as well set up a big damn skybox at that point.

Wise words, me! 

All that aside, there still needs to be some randomness and unpredictability, but the question is how can you make this *fun?* Throwing crap in a big goddamn sphere is simply not going to cut it -- the player is never going to interact with or care about most of it. In order to make the player interact with the environment, I need to make sure they're moving *through* the environment, interacting with it and dealing with it appropriately, and my original scale of 5 or 6 kilometer spheres simply wasn't going to cut it. There's just not enough travel time between objectives for the player to care.

More to the point, we're working in space here, a fully three-dimensional space with clear paths from point A to point B. If I show the player a marker on their HUD for an objective, they are going to point their ship at it and fly straight toward it. I need things to get in their flight path and screw it up, or patrolling enemy ships, or just roadblocks.

How do I combine this with the need for distance, and more to the point, how do I *generate* some distance? And how do I make it so that objectives can chain together and keep the player on a reasonably steady flight path? I don't know how other people design their environments, but this was how I thought about it: as a series of rings rotating and shrinking inward.

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/torii.png "MSPaint to the rescue!")

The player will spawn somewhere in an outer ring 20 kilometers in radius or so, and the first objective will spawn in a second ring perpendicular to the first ring, thus guaranteeing some distance between the player and the objective. These rings can grow or shrink in size based on the objective and projected travel time (obviously, a salvage mission will have lots more points of interest than a simple "go kill these guys" mission), but with subsequent objectives they will *always* shrink inward so the scenario doesn't drag on too long. In the worst case, two objectives might generate where the rings "meet", but in that case the objective immediately *after* that one is guaranteed to have a ton more distance.

Great, so this is how to space out objectives randomly with some minimum distance. What about all the crap in the debris sector? Malfunctioning turrets, mines, engines that fly off and explode, ship-to-ship missiles? How do we make sure the player interacts with that?

## Where The Player Is And Where They're Going

First and foremost, we know *damn* sure that the player is going to be driving towards an objective, so we can place debris and mines and what-not around the objective to force an interaction. Enemy ship wings as well and such; you can set these up in shells or spheres around the objective marker itself.

Second, we can make a reasonable guess that the player is just going to drive in a straight line toward said objective. To this end, we can place debris along the flight path from point A to point B, along the projected flight path in the earlier diagram, forcing them to fly around it and engage with it.

Since we can make that guess, we can then put up roadblocks on that straight line. Minefields can generate, or fields of missiles, and malfunctioning turrets along the flight path to the objective. Enemy ship wings can also be set up to patrol that 'corridor' of space, since we can assume that the player and their wingmen will *probably* be there. After some serious playtesting this strategy seems to be a good one; I was forced into actually managing my ammo and using my physics weapon, as well as using the environment in a fight rather than just shooting the bastards. It was a hell of a lot more fun than the old system and pretty engaging on the whole.

## Flying In A Straight Line Sounds Boring!

You're right, it is, despite the road-blocks set up, despite the debris, and despite the enemy ships in your way, you're still going to end up with a lot more time simply flying your ship. This appeals to the Elite: Dangerous crowd, I'm sure, but not really to me. To that end, I've set it up so that your wingmen on your crew will banter back and forth with each other during downtime, in much the style of older RPGs where your companions would talk at each other while you were simply wandering around. Writing the dialogue and the process there didn't involve any fun new tech, so I'll ignore it here.

## What If I Want To Leave The Corridor?

Excellent question. The corridor has about a five-kilometer diameter (about the size of the original map, actually), but clever players will eventually notice there's a ton of empty space to fly unmolested through at the cost of sheer boredom for optimization. Also, anyone who reads these blogs, you *fiends*.

To that end, a stick will be applied.

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/carrotandstick.jpg "Carrots are for amateurs.")

The old concept of carrot-and-stick applies very heavily to game design, where you want to reward good behaviors and punish bad behaviors. In this case, there's no real way to incentivize making your life more difficult (beyond "it's more fun!"), so I have to smack the potential minmaxer player on the ass with a stick. Going to the edge of the corridor will result in your wingmen saying "hey, we should stay hidden near the obstacles" as a warning, and then going outside the corridor will have enemy ships spawn on you, and going *way* outside the corridor you're just going to get sniped by a big laser or something. This does not seem unreasonable given that the plot of the game as it stands is *hiding from the goddamned enemy*, so it makes sense to stay in the place where you can, yknow, *hide* instead of parking your easily-targeted ass in open space.

## This Seems Really Simple!

So does the cat-flap, but someone had to think of it first. Ideas don't just spring out of nowhere; usually they need to cook for a bit.

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/catflap.jpg "Go read Dirk Gently's Holistic Detective Agency.")

In this case conceptualizing the whole layout for a given encounter gave me a week or two of headache just *thinking* with no actual code written, and also involved some rewiring of my map builder functions and my objective functions. It's a hell of a lot of fun, though, and I can't wait for other people to give it a whirl.

## Takeaway and What's Next

The key takeaway here for anyone reading this is that *random generation is not inherently fun and never will be.* You need to think out how and where you use your randomness, how it gets placed, and just how damned random you want it anyway and what the purpose of it is, and you *still* need special things to put in all of your new-found randomness. Whether you place your crap in toruses or invent biomes or whatever you do, you still need *some* patterns so your game is actually fun. Next up is continued aesthetic work and adjusting the generation of the other sectors to match -- models, textures, and finalized sprites, oh my.

That's not as fun as seeing me fall on my ass, though, so here's me completely screwing up the implementation of fog on a shader.

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/fogfuck.png "Very trippy but ultimately useless.")

Yeesh. Shader errors look *really* psychedelic. Here's another airball of my initial cloud art for the skybox background. I welcome you to the *fried shrimp dimension.*

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/friedshrimp.png "At least I got more of a handle on the painting methods.")

The top part being a bar of light is designed so that it warps into a sphere of light (a sun!), but the actual clouds... could use some love. The final version had more color contrast as well.

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/noshrimp.png "Damn RIGHT!")

Skyboxes tend to look weird as hell because of the warping involved turning it into a sphere or a box; in this case while the green stuff *looks* like it's centered on the actual piece of art, there really isn't a "center" at all.

And now back to the asset mines. Work to be done, but fun to be had.