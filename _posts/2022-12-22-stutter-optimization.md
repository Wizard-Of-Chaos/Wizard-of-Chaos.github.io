### Asset Preloading and Scenario Setup

I haven't had much to say recently for a variety of reasons -- the first and foremost one being, I'm pretty much done with programming for this game! 

Barring scenario-specific stuff and a few extra functions that will need adding -- relatively minor in the grand scheme of things -- the game's pretty much done on the backend. All I've been doing in the codebase recently is the equivalent of Legos; sticking together all my little spawn-this-thing functions together and coming up with generation algorithms to make scenario environments and all that noise. Plus a few quality of life changes. What there is to add still is going to be mostly special effects code, as in like with the gas sector where it slows down ships that go through specific areas and whatnot.

This isn't to say I haven't been busy, though. I've been stuffing in more assets, guns, and ships, along with writing dialogue, but all of those things take me a hell of a longer than actual programming.  Briefly, though, let's go through what got added!

## Gas Sector

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/gassector.jpg "Pretty!")

The gas sector, which is the next sector in the game after the asteroid sector and the debris sector, has been stuffed in and it has some weirder mechanics to go along with it.

When designing these sectors I usually like to try and take the time to figure out what the hell the gameplay design is going to look like for each individual sector. For debris sectors, I wanted a fairly active environment that was hostile to the player, but which didn't *move* much, allowing the player to take their time and figure out the environment and the area. As an intro sector, I wanted it to be relatively fun and engaging while still allowing the player to take their time with it.

With the asteroid sector I wanted to throw some fastballs...

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/randy.gif "I'm not gonna stop posting this.")

...but not too many. The guns and ships are pretty much the same as with the debris sector, but now instead of the environment being hostile to you, now it doesn't give a shit about you in a hostile way. The asteroids *move.* They move in a way that forces the player to think out their flight plans a little more while still allowing some time to look around and figure things out. The enemies are almost identical to the debris sector and so is the relative challenge level - this one focuses a bit more on flying and lets the player have fun with their newfound arsenal as they get more comfortable with the game.

The gas sector is *weird* by comparison, and it's where I wanted to start throwing curveballs. After two bossfights and two sectors, it's time to bump up the challenge. My initial idea for this sector was to enforce high quality aiming, by making explosive gas clouds that blew up an entire field of gas when shot too much.

This turned out not to be very fun when playtesting. The AI doesn't watch its shots, so what ended up happening repeatedly is instant death at all times. I tried tweaking it a bit, but it either wasn't threatening or was too busted, and even if I got it balanced well I don't think it'd be fun just because *you can't control what the AI shoots at.* Taking away player agency is a bad idea. 

So there was a design pivot. The gas sector now encourages you to bait the AI around and force them into bad scenarios. Different types of gas clouds have different effects when flying through them -- one will give you a massive speed boost, one will drain shields, another slows you down, and some explosive clouds are still there, but without the apocalyptic consequences (only a piece blows up at a time). Additionally there are gravitational anomalies designed to yank and hold your ship down. All of these are useful to bait the AI through and around. It's also very different visually, which is a step in the right direction, too.

On top of that, the ships need some extra challenge. It's time for...

## Gunships

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/gunship.png "Cluttered screenshot, but hey.")

This was a pretty fun achievement but it was boring to talk about, mostly because I already had the pieces in place and just needed to shuffle them around a bit. Carriers have turrets, after all. It's a quick hop to shuffle some code around. Along the way I also implemented scaling a bit better and included some better mass calculations to make them fly less like shit.

Gunships are just big damn ships that move at regular speeds but have turrets, making them a bitch to engage on even if you're chasing them down well. Coordinated efforts can bring them down pretty easily with your wingmen, but they're still a significant threat. Right now I have the Wasp gunship there, but there will of course be many more varieties in the future.

## Extra Dialogue

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/arnoldchat.jpg "Arnold!")

Writing dialogue is *haaaaaaard*. I've complained about it before, and I'm gonna do it again. My esteemed writing partner and I have been hard at work on this crap, and it just sucks the life out of you super fast. 

This is partially due to the fairly deep dialogue system I came up with (our sound designer commented that it "wouldn't be out of place in a CRPG"). Briefly: Dialogue itself, two characters talking? Fun to write. I like writing dialogue like that for short stories and whatnot -- it's simple and makes sense. Writing dialogue trees? *Exponentially harder.* It's not two characters talking -- you have to write several different conversations and stuff them together and make them relate to each other, and consider the consequences of various responses, while *all the time* trying to come up with reasonable options for the player to choose.

Even a twenty-node structure (i.e., a dialogue tree with twenty lines from whoever you're talking to) gets wicked nasty super fast when you consider player choice and how to get these twenty lines to link up believably. It's really hard, and I didn't expect it to be as hard as it is when I started out, but we're trucking along for all that. 

My partner here has written up a wonderful wingman named ARTHUR as well as the event for said wingman -- he's a debatably-sentient drone who links up with the AHR in exchange for a ride out of the system. We've written several more exchanges between characters, a bunch of individual conversations, and a few extras alongside events. I've added James Lavovar, who is an immortal mercenary (well, *he* thinks he's immortal, anyway).

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/james.png "James!")

James and ARTHUR both bring some particularly odd perspectives to the table, but you'll have to play the damn game to hear more about 'em. All the wingmen are fun to write and we're of course going to continue to add more -- I'm thinking like twenty possible wingmen. We hope you enjoy talkin' to em.

## More Ships! More Stations!

While I was at it, I also implemented a new type of sniper ship with a minor AI modification so that it'll stay at long range. It does exactly what it sounds like, but I needed a telegraph for the thing so that the player doesn't just get sniped out of nowhere -- so now a little noise plays when an enemy ship locks onto you.

Additionally, I finally got off my ass and implemented a heavy human fighter that's slow and turns like crap, but has more guns than the other two types of ship. In the works is a really fast ship with less health as well. The heavy fighters are probably better for sniper AIs like Arnold and Tauran.

A station has also been added for use in the gas sector's boss battle, but more on that later.

## Better Scenario Design

Scenarios will now have multiple radio signals, only one of which has the actual objective at it, to encourage the player to fly around a bit more and explore the environment and get exposed to its hazards. Objectives needed a slight rework here, but it wasn't a major thing. Scenario playing fields are now significantly bigger as a result.

## Special Effects

Ships now light on fire at low health, providing some visual feedback to whatever damage you're dishing out, and I extended the engine's built-in effects with a couple of other basic ones that were necessary for gas sectors and whatnot. None of it was interesting to talk about, but I did get some new tools to build said effects so they don't look like crap, which is a win overall.

## Supply Depot Sector

The next sector will involve supply depot raids. This necessitates new mechanics and a *ton* of new assets, which is what's been occupying my time. Effectively, for the supply depot sector, I want to allow the player to both capture neutral weapons platforms (that will then engage enemy fighters) and have to defend captured stations or points against waves of enemy fighters.

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/captureship.png "Hey! It's a plane!")

This also means that I need a small amount of escorting done. Another ship added is effectively a transport shuttle, as part of the Chaos Theory's arsenal, that will go to an enemy station and then capture it. While it's doing that, the player will need to swat down several fighter wings that spawn in to prevent this. I just got this done for the gas sector, actually, which is what I was *originally* gonna talk about here.

## So Talk About It

Yes. Good.

My objective system was equipped to support additional ships spawning in -- I wasn't concerned about that. The problem there was that it caused a *helllll* of a stutter when it did so and effectively halted the game for half a second while it loaded in everyone's various models and whatnot. If I'm spawning in several wings, one after the other, repeatedly, this won't do. It'll piss off absolutely anyone who has to interact with it -- it pissed *me* off just testing the damn thing.

So what's the problem here? Loading extra ships and enemies and crap in isn't a huge performance hit -- other games, more complicated, pull this all the time, and my ECS system is built to support that kind of on-the-fly addition basically instantly. No, the problem here is pulling files off of disk. No matter how fast your hard drive is, it's going to be slower than RAM every time. I try to compensate for this (and so does Irrlicht), but it was still causing a problem.

This isn't just a me problem, by the way -- [actual developer studios struggle with this](https://www.dsogaming.com/news/the-callisto-protocol-has-shader-compilation-stutters-and-optimization-issues-on-pc/) sometimes.

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/shaderstutter.png "My jaw dropped a bit when I saw this, honestly.")

This is effectively the same exact problem I was having, although on a much more terrifying and baffling scale (nobody's going to crucify *me* if I launch with a slight stutter, but these guys...). Usually it's more of an oversight than anything else, and that's exactly what the Callisto Protocol devs claimed it was - a quick clerical error where the shaders were not being pre-loaded and were instead loaded on the fly. Disk loading is *slow*, and you want to do as little of it as possible while still keeping your memory relatively clean and not requiring a hundred gigs of RAM at all times. It's a balancing act, and for this I was on the wrong side of it.

Every time you load in a new model or texture in this setup, it gets stored for later usage, so we don't have to deal with this crap more than once - the model and texture are kept in memory so that adding in a new one can happen very quickly. This works fine for things like bullets and explosions -- that way there's no noticeable lag whenever you open fire with a machine gun. The problem here was that when spawning in a wing, the assets for said ships and weapons had *not* been loaded right when the scenario started like everything else -- therefore causing the performance hit.

The solution here is pretty obvious, frankly. The spawned ships need to be loaded in at scenario load time and handled the same as that, just like everything else. So I spent some time writing out a pre-loader for given scenarios. While I was at it, I stopped handling spawned wings in code with magic numbers and just threw it straight into the damn environment loader, so it's easier to edit what the player will encounter on any given scenario now.

## What's Next?

I gotta get the supply depot sector done and I gotta catch up on dialogue. The supply depot sector necessitates a lot more models, textures, and mechanics, so work has been slow for that reason and also [for other reasons.](https://store.steampowered.com/app/975370/Dwarf_Fortress/) But it's coming along! I look forward to sharing more capture mechanics and additional sectors.