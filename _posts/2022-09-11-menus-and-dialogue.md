## UI Work and Dialogue Tool

Just a quick update this time!

I mentioned last time that I wanted to take a crack at the construction menu and some other GUI design. That was absolutely true. I went to get started on construction - wrote out a quick couple of classes to represent enhancements for ships and weapons and such - and realized that I needed to revamp my loadout and wingman menus *first* to be able to represent that at all. So that's what I've been doin'.

GUI work is the most tedious of all possible hells, especially since I have to create all of the assets involved myself. The campaign menu looks better, now, though, and so does the loadout menu! But I expect work to continue irritatingly slow for awhile.

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/newcampaign.png "Look at that gradient fly!")

I'm not much of an artist, but this looks better than the previous bland-as-hell look. That thing took me two or three days, with the new nav-bar included, for the total process of design, art, and implementation. Better yet, whenever I get around to making *better* assets, all of the new stuff here should be easy to replace, since this is how I want it to look for the future.

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/newloadout.gif "So much effort, so little time spent enjoying it.")

Ditto for the new loadout menu. This alone has taken four days and it's going to keep going - you see those tabs for loadouts, weapons, and ships? Yeah. That'll be fun.

I do wanna make a point here about UI design. The paradigm I'm going for is to use as few clicks and mouse movements as humanly possible. That gif is about ten seconds and I dumped the better part of a week into it (although a decent chunk of that was simple art). The UI here includes setting up wingmen, assigning ships, and setting it all up on a tab with repair buttons and the ability to switch the loadout of that ship. There's like fifteen moving parts that go into any of the convenience buttons, and you, as a user, will *never see it-* but you *will* be annoyed by it if that whole process took *twenty* seconds instead of ten. It's pretty thankless, but also vitally necessary. Good UI is satisfying when you get it right, though.

### Dialogue Tool

Oh yeah, on top of that, I wrote a dialogue tool to load and save dialogue trees from XML. [In a previous post](https://wizard-of-chaos.github.io/2022/08/15/dialogue-format.html) I talked about building a dialogue format and how it worked under the hood. Writing XML files got pretty damn tedious pretty fast, so I wrote a tool with [Qt 6.3](https://www.qt.io/product/qt6) (which is a GUI library among many, many other things) to make it easier.

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/dialoguetooleditor.png "Qt is a great library.")

As I mentioned in that post, a dialogue tree can be thought of as a series of nodes of dialogue, with each node keeping track of its own ID and its text. It also keeps track of the choices associated with that node, which in turn keep track of their own text, the next node that choice would lead to, and any effects the choice might have. Pretty simple stuff, and I actually did write some dialogue UI to use this, which I'll show off later when I have more art and dialogue to show off alongside it.

What's more, you, too, can build your own dialogue trees with this tool. [It's available right here](https://github.com/Wizard-Of-Chaos/SimpleDialogue) with an explanation of the XML format that gets used and some build instructions with CMake. You should be able to compile it with Qt Creator using 6.3 on whatever platform you want. It's pretty easy to use, because I am a lazy shit who doesn't wish to look at XML any more than he has to.

It should be noted that this doesn't come with any class structures to load a tree into; I'll be releasing those eventually as their own library for Irrlicht (or maybe just in general - we'll see!). How you manage this XML document in your own code is your own work; I have no idea what your UI or structure or choices might be, so it's left pretty generic. But at least actually writing the damn tree is easier now.

### Forward with the GUI work

Woohoo. I gotta write out the loadout tab, the weapon management tab, and then the carrier tab, and then finally I can write the construction bay and get back to game design above and beyond the enhancements. That should be, however, some of the absolute last menu work I need to do for quite a long time (knock on wood...), so getting it out of the way is nice and will allow me to move faster in the future with actual in-game development. Wish me luck.