## UI Design Completed

Yeah, we're done here.

The UI redesign (and actual design, in a few spots!) has been pretty much completed. Only took me the better part of two weeks. Last time I wrote about how I essentially had two principles for the design of UI. I want to click as few times as possible, and I want to move my mouse as infrequently as possible. On top of that I also wouldn't mind if it, y'know, *looks* good, but functionality is a bigger concern. Let's see what I'm jabbering about!

> NOTE: A lot of the assets you are seeing are placeholders, since I'm not much of an artist. The actual design (e.g. where it is on the screen) and functionality of the menus involved is almost exactly how I want it, though. The only thing that would really change is maybe some of the buttons and better background art.

### Loadouts

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/ui1.png "Newgrounds aesthetic?")

[In my last post](https://wizard-of-chaos.github.io/2022/09/11/menus-and-dialogue.html) I had a nice little gif of this menu in action, but there have been a few changes! Notably, there's a sizeable graphic of a *dude* occupying some screen space. Who's this asshole? 

That would be Martin Hally, your ship's chief engineer - savant and profanity enthusiast. He's very unhappy when people abuse their machines. That includes you getting shot. On top of that, he's been assigned to explain this menu and general ship design, so he's pretty grouchy about having to teach kindergarten too.

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/martin.png "Really, really grouchy, actually.")

Martin's tutorial dialogue - which explains the difference between the types of weapon and ship design and wingmen selection and all that - was my first test run of using the dialogue tool from last time, where I actually wrote a pretty significant amount of dialogue nodes and choices and everything. I'm happy to say it worked out beautifully. When I used it to load up some previous trees I'd written so I could edit and save them, it actually fixed errors in my (handwritten) XML for me, so that was pretty funny too.

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/ui2.png "Ship loadouts!")

On top of this we also have ship loadouts available as a screen now, which allow you to select a ship and edit the loadouts of its respective regular, physics, and heavy weapons. If a weapon is equipped, it'll show a little button next to it that'll take you to the loadout for that weapon (upgrades and such). If the weapon uses ammo and isn't actually full on ammo, it'll show a little reload button for your convenience. The arrows on the side are for ship upgrades, and will fill in if they have an upgrade currently selected. As I've said, I want all this to take as little time as possible.

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/ui3.png "Weapon loadouts!")

The weapon UI is probably the worst-looking of any of them, unfortunately. It has the same upgrade arrows as the ship menu and a button for reloading the thing if you're out of ammo, but otherwise it's just a basic view of what's going on with the ship. There's honestly not too much to worry about with it, so this screen has a lot of dead space.

### What about the "Carrier" tab there?

Yeah, carrier upgrades and management are going to be a "later" thing. That's the last chunk of UI I have to do, but I'm putting it off because I don't have a coherent idea on how to do carrier upgrades yet - and more importantly, given that I've been doing UI and not game design, I don't have a good way to load up the Chaos Theory outside of a basic carrier like the enemy one that shows up sometimes. As soon as we have boss fights, I'll come back to that one.

### Campaign menu

Speaking of dudes on the screen and tutorials, there's another new face on the command screen!

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/ui6.png "That coffee is dangerously close to spilling.")

Steven Mitchell is your personal gopher and relays orders around the ship. His purpose as far as explanations go is to explain the general goings-on of the game and where you can find help with various tasks. He's the level-headed sort and is happy to help with your questions.

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/steven.png "Steven is not, however, the responsible sort.")

There's another NPC that will be on this screen too, your comms officer, who will answer more specific questions about scenarios and orders, but I haven't drawn her yet or written her dialogue.

### Takeaways

The thing to note with all of these menus is that there's a lot compressed into a fairly small space with the buttons attempting to be as easy to understand as possible. In my opinion, the purpose of a UI is to get the hell out of the user's way. I've endeavored to provide a bunch of information for as little work as I possibly can. So far I'm pretty happy with the results.

### Fabrication Bay

This one took a hell of a lot of time because of the sheer amount of crap involved in it.

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/ui4.png "Well, at least it won't spill.")

The fab bay is a top-down screen that you're looking at and includes a lot of its own crap. You'll be able to toggle between building and scrapping ships / weapons / upgrades, as well as carrier upgrades (again, when I get around to that). The list is as close to the "Confirm" button as was feasible with the layout, again so you have to move around as little as possible.

The weapons loadout looks pretty similar to this one, although it has a selection for regular / physics / heavy weapons, as it did on the loadout screen. I do want to point out the upgrade tab, though:

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/ui5.png "Oooh, customizable!")

I don't have too many upgrades written out yet, but you'll notice here that you're able to decrease or increase the effect of a given upgrade for a similarly scaling cost. This should allow you to make a choice between high-powered and expensive upgrades and "right now" upgrades. Again, I wanted this to be close to the "Confirm" button so I would have to do as little moving around the screen as humanly possible.

All three of these tabs took a lot of time (the upgrade one in particular forced me to finally stop procrastinating and figure out how to implement those in-game) but I'm pretty happy with the end product as far as layout goes. What's kind of annoying, though, is that I've written twice as much damned GUI code as I have *actual game code* by this point. Nobody told me this would be the case, so I'm issuing a warning to any other solo devs out there - *you will spend twice as long on the user interface as you will the actual game* if you want your UI to be any good at all.

### What's next?

Well, I said I needed to get all my ducks in a row for the debris sector before starting in on other sectors. I've pretty much done that. It's back to game design - the AI needs some fiddling, there needs to be another chunk of UI (AAAAAAAAAA) for the mess hall so you can talk to your wingmen (and an in-game system for recovering wingmen), the objectives need a bit of work and more variety, and I need a boss fight. Once I've got all that I'll be done with the first sector of the game and can move on to future sectors... only those will be much, much faster to produce since I have all the basic stuff.

Forward!