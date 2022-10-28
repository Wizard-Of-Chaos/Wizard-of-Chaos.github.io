## Dialogue Systems and Content Updates

I've been adding so many damned features recently that my head's spinning but, like I said in the last post, none of them have been particularly inventive code-wise, and have simply been building on my existing infrastructure. They're cool as all get-out, but not, like, programming-wise. Briefly, though - the settings menu is now working, mostly (just needs a few more fiddly bits added onto it), there were a few AI range value updates (i.e. making sure the AI doesn't waste ammo if you're out of range of its guns), some extra UI added on, a whole *boatload* of new guns and ships, additional wingmen and dialogue, and an entire litany of bugfixes and tweaks to make sure the campaign goes down smoothly. 

I'll talk about some of the changes there in a second, but I wanna mention that carrier upgrades are a thing now and because of that there are no more placeholders on the UI menu as far as buttons go. Everything you can click does something, now, which is a neat little milestone (if a bit weird).

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/fabmenucarrier.png "Not all upgrades are visible here - not all upgrades can be crafted.")

Carrier upgrades come in tiers and allow for some incremental changes to your carrier that you can dump spare supplies into, increasing damage resistances and buffing turrets or speed values and the like. Not all upgrades are available to build, though - some of them come from events that happen in the campaign.

Hey, that's a nice little segway to talk about...

### Events and Flags

My campaign didn't have the set-up for events happening or flags of any variety prior to a rework, and it was something I knew I needed to add, but I had more pressing concerns. When I say flags here I'm talking about various things like "was Arnold Kenmar recruited?" as a true/false value, and then doing something based on that value. In this case, my first use for the flag system was making sure that Arnold was the first wingman recruited, since he's a fairly important wingman character-wise. These are simply loaded from a file so I don't have to bake them into the actual code. Cool.

Events are similar. Basically, if there's a "plot" event (i.e. something triggered by conversation or by wingmen or some other damn thing like a boss fight), those take priority, and Steven will talk to you on the launch screen about them when you get back from a mission (this is, in fact, the entire purpose for the launch screen, aside from looking cool). There are also a couple random events that I'm working on that happen if all the plot events are out, like pulling in an alien stasis pod and having the option of either strapping it to the hull (health bonus!) or throwing it back out (detection chance less likely).

You'll notice these are all things that would inherently use the dialogue system and would require flags to be set or un-set as necessary as well as possibly running with their own, special effects. This is not a thing that my prior system was set up to do, so that was the first thing on my menu as far as "Things What Need Doing" go recently.

### Dialogue Rework

Behold.

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/dialoguetooleditor_rework.png "Neat!")

I talked [in a previous post](https://wizard-of-chaos.github.io/2022/09/11/menus-and-dialogue.html) about a dialogue tool I'd created alongside the game to help me out with working on dialogue. You'll notice this one is a little more sophisticated. It allows for required trees and flags on any given choice, and allows the choice to set flags on top of having the effect it was previously allowed to have. The actual details are pretty boring, but re-working the tool and the xml parsers to account for this (as well as the campaign and menu options paying *attention* to flags) took me some time to get done. (The tool, like before, is [available right here](https://github.com/Wizard-Of-Chaos/SimpleDialogue) if you want to compile it and fiddle with it yourself.)

I'm downplaying this as far as the amount of work that went into the flags and event systems, but really, it was just *boring*, if tedious - I knew exactly what I wanted to do, but setting up all the hooks for this took quite a while regardless and had a couple of insane bugs that went along with it just based on the extra complexity. Flags are just a damned bool value, true or false. Nothing interesting there.

I did get curious, though, about how *other* systems do dialogue work and other games.

### Note on Dev Snobbery

I'm kind of a snob as far as programming goes. I'm a fresh college grad with a degree in computer science (at the time of writing, I got my degree less than a year ago), and I'm pretty good at it, specifically system design and algorithmic design. If my idiot classmates were any indicator of the type of person going into the CS field, I'm way better than average at it (and I can't wait for someone to eventually read this after I publish the open-source code and quote me here just to shit on me for some god-awful chunk of code somewhere in the game). It's what I do for my day job, too, which I guess makes it sort of freakish to also do development for fun. I'm built different.

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/builtdifferent.png "Believe yourself to be special?")

I have a tendency to sneer when I use the term "indie" or "indie dev" (usually because it's part of a sentence that runs, "some fucking indie roguelike trash"), despite the fact that the term also applies to *me* and I am worthy of my own contempt. The average connotation when you think of indie code is garbage like whatever the hell YanDev puts out, or funny Dwarf Fortress bugs, or some imbecile trying to make a 2D Earthbound-inspired JRPG in Unity. As far as professional programming goes and experience goes, I'm still new, but I'm pretty good for my relative experience, and definitely better than most in this area.

That said, as far as professional devs go or developers from back in the day when 2 MB of memory usage actually *mattered*, I am a fucking lightweight and I don't think any of them would do much more than spit on me.

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/smasher.png "You think you're special 'cause you like C++? DON'T MAKE ME LAUGH!")

### Dialogue Storage: Testosterone Edition

For reference, my own dialogue system uses an XML format, which loads individual trees and their flags / consequences and stores them in a code structure that's basically just a map. The trees themselves are also stored in a map. Load times are pretty fast, access times even faster due to the efficiency of hash maps. It's fairly user friendly to look at, too. XML formatting is pretty heavy, though, and adds a lot of bloat to any given file, and loading it can definitely be slow at times if you have a *huge* file (think a Dwarf Fortress legends history export).

Let's take a look at how some actual heavyweights do it. BioWare games have a ton of dialogue in them as RPGs, so they presumably have a pretty robust system. I knew from prior experience [installing the single stupidest KOTOR mod ever created](https://deadlystream.com/files/file/1313-kotor-dialogue-fixes/) that dialogue was somehow related to a .tlk file associated with the game. This is as good a place to start as any.

I had a deeply unsettling realization when I looked at it that this was a 5 or 6 megabyte file simply labelled "dialog.tlk", and given how text is pretty small as far as files go, 5 or 6 mb is probably *the entire set of dialogue for the game.* So how the hell do they store that? I pried it open in Notepad++, and after reading past the metadata, I see this...

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/kotordialog.png "This is actually from KOTOR 2's .tlk file, but you get the idea.")

Dialogue in this format is stored as *one single, long, uninterrupted string of characters.* I damn near fell out of my chair when I saw this. This is completely incomprehensible to look at without a tool, which I guess is my fault for not trying to find a tool first to look at it. Let's go do a quick search to find a talk-table editor...

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/talktableeditor.png "Holy 90s, Batman!")

There's a few things to look at here. I now know that strings are simply stored as a number with a reference to that string. I also know that they are loaded based on their *position* in this long uninterrupted string of characters as well as the *length*. No delimiters are on the string (I found out later that some strings have de-limiters and some don't, so you have to use length - I've got no idea what they use delimiters for internally). That means that any parsing for this must, necessarily, simply open the file, grab the appropriate data based on the string reference it has, then move to the position and yank the string before closing the file.

See, this is how *men* did it, back in the day. 5 MB of memory or disk space doesn't seem like all that much nowadays, but keep in mind we're looking at a system from twenty years back, minimum - resources were a little tighter back then. A friend pointed out that the original Xbox had just 64 MB of memory, and KOTOR was designed to run on that console. This is a far cry from my own 16 GB of memory. They had to manage this stuff way more harshly. So how's this stack up to what I'm doing?

### Comparing Professionals To Some Idiot CS Grad

We'll need some benchmarks to get a rough estimate. 5 MB of straight text is just going to be stored as a big, long array of characters, and will probably be about 5 MB in memory. By comparison, my set-up uses a bit more infrastructure and `std::unordered_map` as well as `std::vector` as its major components. [This source](https://chromium.googlesource.com/chromium/src/+/HEAD/base/containers/README.md) tells me that:

> The empty size is sizeof(std::unordered_map) = 64 + the initial hash table size which is 8 pointers. The per-item overhead in the table above counts the list node (2 pointers on Windows, 1 pointer in libc++), plus amortizes the hash table assuming a 0.5 load factor on average.

The same source also tells me that the overhead per item on `std::unordered_map` is about 16 to 24 bytes, and that the size doubles every time the load factor (how much crap is stored in the table) exceeds 1 when you have more than 64 items. If you want a detailed explanation on how, exactly, a hash map works, [there are many resources for that](https://en.wikipedia.org/wiki/Hash_table), but if you're unfamiliar and just want a quick explanation, a hash table takes up a bunch of memory and then ties a value (in this case, my dialogue tree) to a specific point in that memory. 

Think of it as though I have up to a thousand chairs at a party. You walk in and, because your name is Adam, I give you chair number 275 (ASCII values for each letter in the name). Some other guy walks in and gets chair number 779 because of his name, and so on. This means that if I know someone's name, I know exactly where they're sitting - someone comes in later looking for Adam, I direct them straight to chair 275.

The finicky bit happens when there are simply too many people and not enough chairs to make this algorithm work properly - if I have a thousand and one guests, or if two guys have the same name. To correct this, I simply double the amount of chairs to 2000 and adjust the algorithm a bit. In this manner hash maps continually kick the can down the road at the cost of eating memory - if I have two guests instead of a thousand, I still have a thousand chairs. Obviously this is better for large datasets, like, dialogue, for example.

All this means that my system, storing trees and nodes in `std::unordered_map`s with their underlying hash-table, will eat a lot more memory than it actually costs to store every piece of data in the name of efficient access. It's hard to beat hash-maps in terms of efficiency on a variable dataset. 

The thing that makes the BioWare devs smarter than me is that they *know* their dialogue isn't actually a variable data-set: the strings involved do not actually *change* in the game. So they effectively wrote their own map with their own function to know exactly where a given string might be, and thereby avoided the massive overhead. Their "map" stores the position and length of a string, and when given a key, it finds the position and length associated with that key and goes to yank it out.

Storing it all in one big damned character array, too, is a hell of a lot more efficient than an XML file. They just have the raw data and a few bits of metadata. For reference, let's look at a chunk of text in my system:

```xml
<node id="root">
    <speaker name="Steven Mitchell"/>
    <text text="This is a quick test to demonstrate the new dialogue tool."/>
    <choice text="Tell me more!" next="tellmemore" effect="none">
        <required_flag id="XML_REQUIREDFLAG_1"/>
        <required_tree id="XML_PRIORDIALOGUE_1"/>
        <sets_flag id="XML_FLAGSET_1"/>
    </choice>
    <choice text="I don't really care." next="none" effect="exit">
        <sets_flag id="XML_DONTCARE"/>
    </choice>
</node>
```

You'll notice a lot of this is padding for the XML format - tags and required things and end-tags and so on and so forth. This is all necessary for the format, but eats up way more space than just the raw data. The talk table that KOTOR uses has a lot less padding (as little as possible, actually) and so takes up much less disk space and memory space on top of that. 

I'd be willing to bet my access time on dialogue strings is around as fast as their system, just because maps really are damned efficient, but they blow me out of the water both in memory usage and in file size (almost certainly on load times, too, XML parsing is slower than a simple `fopen`). The singular point I have in my favor is that XMLs are human-readable, so I can just go and edit the file without any fiddling if I want to. I'm benefitting from being several decades in the future with *far* fewer resource constraints, so I can piss away a few extra milliseconds or megabytes if I want to... but they couldn't. I conceivably could store *all* my dialogue in memory and it would take a bare fraction of that 16 GB I have - certainly much less than the rest of the game. Previous devs had to squeeze far harder for efficiency.

Yeah. "Lightweight" is the proper term for myself, if not for my system.

### What's This Have To Do With Your Game?

Very little. I just thought it was interesting. Research work on how *other* people do things sometimes lets me save time on implementations - why reinvent wheels if you don't have to?

In this case, I think I'll be sticking with my own format - a couple extra megabytes isn't going to kill me, because I'm not developing for a system with 64 MB of RAM total. I mean to get into you, the player, for at least a gigabyte, probably more. I'm not too concerned about, say, 25 MB versus 5 MB as far as disk space goes, either. Ditto load times - on more recent hardware, it should be a few extra milliseconds, where the load time back in 2003 (or the 1990s - this system was also used for Baldur's Gate) might have been way, way longer. The readability and ease-of-use tradeoff is worth it for me.

But it is neat to look at problems that had to be overcome on prior hardware with more constraints - and it certainly is humbling to realize just how much worse I am at this compared to *real* pros.

### So What's Next?

Finishing off the event structure, fiddling with the AI a bit more, finishing off the settings menu and then just writing more and more game... and stuffing more and more into my dialogue system. Should be fun!