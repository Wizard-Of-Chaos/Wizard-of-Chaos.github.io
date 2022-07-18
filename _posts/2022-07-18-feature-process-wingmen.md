## Feature Process: Wingmen

After the rework of the AI that I just did to make it more extensible and allow for weirder behaviors than "see man: kill", I started in on 
figuring out how I wanted to add in AI wingmen to the game. I haven't really shared the development *process* as far as a feature goes, mostly because it typically involves an excess of profanity on my part, but I thought it might be interesting for this one.

I started off with a couple of basic questions. How many wingmen should the player be allowed to have? Let's go with three, max. Should the wingmen be randomly generated like the scenarios are, or should they be static across campaigns? Static, weirdly enough, gives me more flexibility, I think - if you know a certain wingman will *always* be better with, say, close-range weaponry, that allows you to try and go for specific builds more than just repeatedly yanking on a slot machine. It was around this point that given the AI parameters I already had like aggressiveness, cowardice, gunnery skill and some others I was effectively creating personalities... and that's a short stretch to characters, which means I might need to do some actual *writing*. Possibly even dialogue. More on that later, I think.

Other basic questions included what additional attributes should I include (very few), should I track anything with them (I went with kills and injuries - if your wingman gets shot down, they're injured for the next 3 sorties), and how smart should they be (depends on the wingman).

Good. Basic questions down. Now, what used to happen was I would immediately set up some rules to spawn in friendly fighters and work backwards from there. Not so much anymore, given that I'm making a game here and need some player interaction.

### UI work

The first thing I wanted to get down was what info does the player need and how can they adjust their wingmen? I had a rough idea of how I wanted the UI to look - you select a wingman, select a ship for that wingman, and you can see a description of the wingman / ship that you have your mouse hovered over.

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/wingmanuisketch.png "All glory to MSPaint.")

It was at this point that I procrastinated for a day or so because I fucking hate UI work. I have to generate a list of buttons on this screen, which is always kind of obnoxious to try and implement (especially when tied to specific instances of garbage, what with how Irrlicht's GUI code works), and besides which there are several buttons that all need to do different things. I hate it. It's awful. It's like this for every damn menu and it's why I haven't touched the settings menu in months despite really needing to at some point (so many *options*, augh).

Eventually I got a handle on my pique and set it up so it looked roughly the way I planned it.

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/wingmanuisketch.png "All glory to the UI assets I made ahead of time.")

At this point none of the buttons work and it's all effectively placeholder stuff. This was the point at which I needed to decide how wingmen and ship instances relate to each other and get stored. Obviously, what I want here is to have the "wingman" structure relate to the "ship" structure in some capacity, and I want to be able to change that on the fly. I spent another day puzzling this out and re-working the storage of specific wingman / ship instance things in order to get what I want, and noted a few places that I really need to just rewire the campaign structure along the way.

The upshot of all this is that in this specific case, I did things ass-backwards and made function follow form. Having the UI laid out ahead of time helped me structure in my head how it would actually work. Some time later I tied code to all the buttons to make it work as intended, and set up some more code to recognize a selected wingman. 

### Slapping it all together

The very last thing I needed to do was take the instance, take the wingman, and spawn in the ship next to the player allied with the player. I had most of the code laid out for this ahead of time, so it was pretty simple to set up (instanced ships are how carriers spawn ships, and faction stuff I already had to deal with to make the AI shoot you in the first place). Scenarios now check the assigned wingmen, see if there are any and then spawn them on your wing.

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/wingmanuigif.gif "All glory to ME!")

And there you have it. Specific wingmen can now be spawned in with specific ship instances from the list of ships that you have. I left the hover-text options at the time of writing because that's the work of two minutes, and I was more excited about the underlying code. Only a few changes are left: I need to set it up so that wingmen get tracked at the end of a scenario (same as the player - ammo, health, and kill-count), which again shouldn't take all that long. I'm probably also going to add a killboard somewhere to see who's the most effective on a given run.

Looking back I think the major realization was the fact that these wingmen are going to have their own personalities, and that I really need to start considering things like *plot*, and *setting* - hell, even RNG games like FTL have a minimal plot and dialogue interactions to keep you engaged. The setting I've been going with prior to this was just that your fleet carrier was the only survivor of a horribly lost battle, and now you needed to escape an enemy-filled solar system. I'm gonna have to do more than that, though; characters and motives and what have you, so that you can stay invested in your wingmen. Terrifying thought.

### Next up

Scenario generation work is still on the big list of "Shit I Need To Do", but since that's more a tweaking / adjusting thing I'm holding off on it for a bit until I implement a few additional features. I need to add in boss battles at the end of a sector, I need to add "sectors" as a concept (for example, at the initial getaway, you'd be in a more "ship debris" type sector and gradually move throughout the system, I need to add turret code (for carriers, and also for obstacles), and I need to fix up some more UI for the construction bay and implement that (along with scrollbars on the various lists). 

On top of that... damn, I need to add a dialogue system or something, figure out how I want to put the characters on display. I need a whiteboard for that last one. Feature creep is a bitch. Sometimes I empathize with Chris Roberts as far as space game design goes, and that disgusts me. I think I might need some head trauma or at minimum a stern talking-to. Gotta keep myself from babbling to myself about virtual drinks and perfect simulated galaxies *somehow.*