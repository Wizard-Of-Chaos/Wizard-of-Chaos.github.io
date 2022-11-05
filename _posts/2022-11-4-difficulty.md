## Difficulty Settings and Other UI

Saving and loading the game is possible, now, which is nice, although it's the ugliest format imaginable. I thought it was gonna be more of a pain, but turns out it's just tedious, not mind-busting. Lots of bugs got swatted and some more dialogue has been added - the past few days have been mostly bug crushing related to saving and loading the campaign. On top of that I'm finally getting off my ass and writing out the settings menu.

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/videosettings.png "Classic.")

Settings aren't, like, *hard*, they're just a pain in the ass to implement when you haven't been planning them from the start. Going through the game and making sure that all objects had shadows and filtering and that particles played nice with the various levels (not to mention variable render distance) was pretty annoying. Wasn't *hard* - just annoying. The one quirk Irrlicht has in this regard is that borderless window isn't a possibility. Implementing it would require me actually changing the engine, and that's too much of a headache to do right this second, so I won't be doing it.

I do want to point out here that you probably shouldn't turn on shadows. Lighting is one thing, but having objects cast *shadows* is quite another. That requires a ton of math to do properly, based on light sources and mesh values, and honestly it should be done with a *simplified* mesh rather than the mesh itself so you have less math to work with. Right now, my shadows are really badly optimized and use the mesh themselves, which generate okay-ish shadows at the cost of utterly tanking performance. C'est la vie. I'll leave that toggle to the freaks with RTX 4080s, for now.

I also want to mention the stupidest settings bug I ever encountered - back when I was still implementing sound and such, having v-sync turned on meant that music would never change. It was some sort of utterly odd framerate-dependent thing. Strangest bug I've encountered yet while developing. Took me a hell of a time to figure out why the hell that was happening.

### Difficulty Settings

Right. So, what makes a game difficult? The trashy answer is "bigger numbers and fewer resources", but, shit, I think we're a *bit* better than that, don't you? That's as simple as turning a number up or down, and is the laziest possible way to make a game more difficult. It's also the most common way, primarily because it's the laziest way. When you ask a regular dude what he wants out of a difficulty setting, he'll usually say he wants the game to be "harder" without giving a clear description of what that means. After some heated questioning, though, he'll probably say that he wants the enemies to be "smarter". Great. That also tells us nothing.

I mentioned [in an earlier post](https://wizard-of-chaos.github.io/2022/07/18/ai-design.html) that it's really easy to say that the "AI should be smarter", and really quite hard to define what the hell that means. In my case, AI intelligence correlates to the behaviors it's allowed to use - whether or not it actually uses its boost, or how effectively it flees (like if it does a random-walk on a flee, for instance), or the range it engages from, or any number of other behaviors. This is pretty simple and has some decent results with how well the AI flies (there's also a behavior to make the AI fly like an idiot).

You can fine-tune the AI all you like... or rather, *I* can. In this case I'm setting it to some predefined values in the interest of accessibility and recognition that not everyone wants to perfectly tune the AI to be a challenge (or not, in the case of game journalists). A simple value for "intelligence" is probably enough for most people.

But why let me do all the work for you? After all *some* customization can be pretty fun.

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/gamesettings.png "Pictured: the game journalist settings.")

You're allowed to tweak the game's difficulty in a number of ways related to the "bigger numbers" concept, first and foremost being able to scale player / wingman health against enemy health. If you're some kind of goddamn pansy, you can set the AI's health to minimal and your own to unfair to have a serious health and shield advantage right off the bat.

The other numbers here are a bit more interesting. There's the AI intelligence setting I mentioned, obviously, with predetermined values on what constitutes "smart" for an AI. On top of that, you're allowed to control how well the AI aims (turns out having AI with laser-precise shots makes things really unfun, really quickly - it's super easy to feel like a game is bullshit when the AI aims like the computer it is), as well as when the AI is likely to turn and run away from you. In the interest of those goddamn freaks who also simply want to see numers go up as a difficulty setting, you're also allowed to tweak how many AI ships are on a given wing - or rather, how many you see per scenario.

A few of the toggles are self explanatory, but some aren't. Friendly fire and impact damage both determine how much your wingmen are allowed to shoot themselves in the foot, and also how much you're allowed to shoot your wingmen in the foot (or vice versa). Space friction is a *weird* setting, but it's been in the game pretty early, primarily because it feels weird as *hell* to play a game without it. Human brains are simply not built for the concept of movement in a vacuum, nor for [piloting ships in a vacuum](https://elite-dangerous.fandom.com/wiki/Flight_Assist). It's far more natural to assume that you're going to slow down after awhile, which 'space friction' lets you do, and ditto for your rotations. Turning this *off* means that shit doesn't happen, and you can go careening off into the void... or perform some incredibly wacky zero-G maneuvers if you so desire. It's possible! But this setting is best left on for newcomers.

Constant thrust is my attempt at implementing a "throttle" setting - normally, you hold down the 'forward' button, and you move forward, and you slow down when you stop - unless space friction is off, in which case you will [keep going until you hit something.](https://www.youtube.com/watch?v=pga_SnS3lPw) With constant thrust on, your throttle ranges up and down and will attempt to keep you at that speed - say, at 70 meters a second in the forward direction. It's useful for Wing Commander types, but for me personally it feels more natural to just be holding down the button to move forward.

The upshot of all this is that you should be able to tweak your gameplay experience based on what you find challenging. If you want to breeze through the game and talk to the fun wingmen, that's your prerogative. I think you're a goddamned *wuss*, but y'know, you do you.

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/taurandialogue_1.png "He's a weeb, but for humanity!")

On the other hand, if you're the type of person to demand that the game devs give you stronger battles to fight, I'm more than happy to accomodate you and allow you to throw yourself into completely unfair and disadvantaged situations... and the game can and will mock you every time you eat shit as a result of your own arrogance.

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/mock.png "Sucker.")

### I Want To See The Keybinds Tab!

Man, it's a goddamn keybinds tab. You know what keybinds are. You select a button to do a thing. If you *don't* know what a keybind is, I can't help you. [Maybe this helps.](https://en.wiktionary.org/wiki/key_binding)

### So What's Next?

I'd like to finish off a few fiddly bits with the settings menu here (I'm sure there's *some* bug lurking in here, too) and then get cracking on wingmen orders (i.e. "attack my target", "give me a hand", "back off", etc). That shouldn't take me too long. Then I can get back to more asset management with writing dialogue for wingmen and more guns / ships, and get started on the gas sector.

Honestly as far as systems go, I'm pretty much done. Like, actually and unironically. Dialogue, events, all the gameplay, the various guns, firing methods, settings, wingmen, saving and loading a game, all of it - it's done. Now we're just adding more actual *game*. That should be a hell of a lot of fun. I'm already having a blast just getting to *play my own game*, and that alone has made the past ten or eleven months worth it. 