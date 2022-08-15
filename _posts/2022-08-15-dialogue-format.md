## Dialogue Format: Constructing a Dialogue System

With the campaign rework out of the way and some work done on sector management (at present, the game generates the debris and asteroid sectors one after the other with a carrier battle as a bossfight ending each sector), there's a few other pieces of tech I need to get written. I'm trying to focus on making just one sector at a time so I can extend and tweak it later with future sectors - and what that means is in order to get it done *properly* I need all my systems up and ready to go. I also need to rework my audio, which is going to be the rest of this week (and probably the next - switching to OpenAL is going to be involved), but first I decided to take a swing at a dialogue system.

In order to make choices throughout the campaign and chatter at your wingmen, obviously I'm going to need dialogue - and I'm going to need some method of storing it to file. The first question is how the hell does a dialogue structure get *built?* There are a wide variety of ways to get this done, and I have *no* idea how other developers managed it, so I'm going to expand on my own thoughts and implementation.


Dialogue tends to get viewed as a "dialogue tree", where each specific line leads to another and each choice makes another branch on the tree. This structure is pretty close to what I've ended up using. Each line of dialogue or conversation can be thought of as a "node" in the tree. Each node would lead to one or more other pieces of dialogue, with a "choice" for each from thep layer, or they might just exit, and they might affect the campaign in some way. This is easier with a visual.

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/dialoguesketch.png "Professional diagrams got set back four decades by the development of MSPaint.")

This could represent a specific conversation and the various branches it might take. You may notice if you're inclined toward being a pedant that this isn't exactly a tree, and you'd be correct. This is more akin to a graph, where I might have various paths leading to loop back on the conversation and allowing you to go through the tree again. I'd actually argue that dialogue can be thought of as a finite-state machine, with each "node" being thought of as a specific "state" of conversation. But that's CS semantics; you're not here to read about CS theory. The question here is how the hell do we structure this under the hood?

I've gone with something simple. Each "node" - that is, a chunk of dialogue - will have its own unique identifier, like "debris_field_argument" or something along those lines. The node will store a list of choices available to the player. Each choice keeps in mind its text (what the player actually says) and the node of dialogue that it leads to - for example, one of the choices available in the "debris_field_argument" might lead to "debris_field_screaming_contest". The whole dialogue tree stores all of these nodes, and the nodes store all of their choices.

This actually makes my job pretty easy. Since each choice structure has an ID for the node it needs to go to next, all I have to do is set up a map. You can visualize that like this:
```cpp
struct Tree
{
	std::unordered_map<std::string, DialogueNode> nodes;
};

struct Node
{
	std::string text;
	std::vector<Choice> choices;
};

struct Choice
{
	std::string text;
	std::function<void()> consequences;
	std::string next_node;
};
```
Obviously this isn't how it actually is under the hood (mine is more complicated and has more utilities) but this is the basic idea for how that would work. I've also included a function pointer for "consequences" - so if a given choice has a campaign impact, I can set it up so that it calls the relevant function when that choice is made. It also keeps track of what node is next up after that choice is made (or the lack of one, in which case the conversation ends).

Good. Fine. We now have a structure for storing a particular conversation. Now, obviously I want to be able to load and edit this from file *without recompiling anything,* otherwise I am going to go utterly insane. I could write my own schizophrenic format for this, but at heart I am a lazy bastard, so I decided to use the file format that (in my opinion) is best suited for storing node-based formats - XML. Tags get stored, attributes get stored, they're all related to each other, and best of all it's *human readable,* so dialogue should be pretty easy to get down in this format.

So I wrote out a format where trees store nodes which store choices and all their respective attributes get added in, wrote some extra lines of code on top of my initial structure to load it from the XML file (not too intensive, but tedious), quickly wrote out a test chunk of dialogue and a "print" function for each node, and nervously compiled my code...

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/firsttry.png "FIRST TRY! SUCK IT, HEMINGWAY!")

...and it worked beautifully on the first try. Loaded it from file, spat out the appropriate pieces, and set up the functions under the hood to map the choice to a consequence. The hooks for dialogue nodes also provide easy access for future GUI work as well as easy traversal of the tree. In the future I'll be working on the GUI associated with dialogue, as well as the audio rework I mentioned, but that's for next week.

This is a quick and easy structure for getting a dialogue system set up for arbitrary usage, where all you care about is the choices associated with it and the text. You could easily add in extra hooks to associate sound files with each node if you have voiced dialogue (I don't). Getting this dead-on on the first try has swelled my ego to unsustainable proportions. I am reminded of the ending of an Asimov short story (The Feeling of Power), although with code structures rather than basic math.

> Nine times seven, thought Shuman with deep satisfaction, is sixty-three, and I don't need a computer to tell me so. The computer is in my own head.

> And it was amazing the feeling of power that gave him.