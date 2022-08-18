## Process: Audio Refactor
Over the past week or so I've been putting in time on reworking the way audio gets managed - as I said in a prior post, the audio system is in a similar state to the campaign, with random calls scattered all over the damn place and seemingly irrelevant functions scattered everywhere. The audio library I'm presently using doesn't *quite* do enough, either, and I want some more fine-grained control over effects.

So I'm switching to [OpenAL.](https://www.openal.org/) This particular post is going to be written stream-of-consciousness style scattered over a few days. At the time of writing this sentence, all I know is that OpenAL has more options than my current library, but will likely be more of a pain to implement. Hopefully this can provide some insight on what a pain a refactor or a given feature might be, under the hood. Hell, maybe it'll be useful for other people implementing OpenAL. I have no idea.

> LATER NOTE: This was honestly less painful than I expected, given that I knew absolutely nothing about the library I was getting into. Most of the major aggravation was in trying to figure out how to load .ogg files into a buffer, which is fairly minor in the grand scheme of things.

### Day 1

Lot of time today reading the OpenAL spec and programmer's guide. Allow me to present the most useless goddamn diagram I've ever seen in my life:

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/awfuldiagram.png "Neat.")

Today I was trying to figure out just how exactly OpenAL is structured. Here's the TL;DR for anyone looking to implement this library:

1. The initial, basic structure is that it sets up an audio device through some function calls.
2. This audio device is given a "context" - you can think of that as the game itself - to play sounds in.
3. In the context, you place "sources" - again, in game terms, think of this as players or obstacles or whatever the hell is making noise.
4. The sources then play audio from "buffers", which are loaded sound files.
5. The listener then hears audio played from the sources depending on things like distance, volume, gain, etc.

Sound simple enough? It does to me. That said, the diagram there was presented *before* a full explanation. I hate it because I wasn't able to understand what the hell it was saying until I already understood what the layout was, at which point the diagram was entirely unnecessary.
```cpp
void AudioDriver::init()
{
	device = alcOpenDevice(nullptr);
	if (device) {
		context = alcCreateContext(device, nullptr);
		if (context) {
			alcMakeContextCurrent(context);
		}
	}
}
```
Great. I was able to get the library set up and working, compile the project, loaded in a device, etc, etc. Now I need to find out just how the hell to initialize audio data to a buffer. 

I've realized that OpenAL doesn't really provide you with any tools to load files. It doesn't give a shit how you load the files. All it cares about is that the PCM data is stuffed into a buffer somewhere. I've got other stuff to do today, like go to the gym, feed myself, and attend a Lancer session, so we'll pick this up tomorrow.

> LATER NOTE: PCM data means [Pulse-code modulation](https://en.wikipedia.org/wiki/Pulse-code_modulation), which is effectively how an analog signal gets translated into a digital format. Audio that you actually hear is a wave in the air, and this particular type of data translates the motion of wiggly air into ones and zeroes. I did some work regarding this back when I was in college, specifically some research work on a different method of audio encoding. Most of what I learned was "audio management is a pain in the ass". [You can read up more on Fourier transforms and methods of transforming analog to digital](https://en.wikipedia.org/wiki/Fourier_transform) if you're curious about this sort of thing.

### Day 2

Okay. Turns out there are a whole bunch of specs available to load files from. The first and foremost one I found was the [OpenAL Utility Toolkit](http://distro.ibiblio.org/rootlinux/rootlinux-ports/more/freealut/freealut-1.1.0/doc/alut.html).

Thing is, this doesn't quite cover my use case. I want my sound files to all be using the .ogg format. As far as I can tell (and someone leave me a nasty comment if I'm wrong) it doesn't work for .ogg files - the best reference I found was a 404'd link to something on the topic. It's also 15 years old, and trying to figure out how to compile the damn thing for Windows 64-bit without killing a man gave me fits, so I dropped it.

> LATER NOTE: A lot of audio libraries really like using the .wav format because it's about as simple as a file can possibly get - it's raw, uncompressed audio data, which means the file size can get huge very quickly. There were many wonderful options for loading a .wav file into a data buffer, but having 50-megabyte files for a 3-minute menu track is a waste of space. The trick here is that you want to be using a compressed audio format that's also lossless so that your game's file size doesn't expand to stellar proportions - a big reason games like Destiny 2 are so huge is because of the massive amount of unique audio files, and it's also the reason that games tend to re-use audio when and where they can (Valve games, for example, do all sorts of tricks with audio so that you don't notice it's the same goddamn sound as something else). There are some good options available, but I happen to like .ogg for this. The trouble is since I don't know the algorithm involved very well, it was hard to try and figure out how to turn the data into usable PCM data, so I had to do the research and find the .ogg libraries. If I'd used a .wav file, it would have been extremely easy.

> It's also worth asking why lossless audio is so important when you can compress the shit out of textures and nobody will care. The answer is that the human ear is a lot more sensitive than human eyeballs. You're probably not going to notice compression on a texture, but if an audio file has even light loss due to compression you WILL hear it almost instantly. That's why some games have uncompressed audio even if the file size is correspondingly huge. Now, if you have uncompressed TEXTURES, you're stupid and I'm going to laugh at you.

After some frustrating google searches, further research showed me a few options for loading .ogg files, one of which looked like it was written by a madman and all of which required me to find, compile, and link the .ogg and Vorbis libraries as part of my project. Fine. [This random github gist from tilkinsc](https://gist.github.com/tilkinsc/f91d2a74cff62cc3760a7c9291290b29) proved to be exactly what I was looking for (albeit written more like C than C++, but that's minor). Thank you very much, dude.

A short time later I've made my game play an explosion noise using OpenAL when it first starts up as a test. Awesome. The ogg parser, sources, and buffers are all functioning as expected at this point.

Great! So now it's time for me to figure out structure and fiddle with the various bits and pieces. I'm thinking I have a class that stores references to various audio buffers - like, say, all the menu sounds, or the music, or in-game sound effects - and allocates / removes them accordingly. In particular I want all the in-game audio effects deloaded from memory when I exit a given scenario, and I want the music to be de-loaded after it changes (since music won't change that often, I think I can get away with loading it from file). 

I'm also going to want a structure for the sources I mentioned earlier. I have my device, I have my context, I have buffers; I need sources. Ideally I want one "source" for music and menu sounds that is always exactly where the listener is, which can probably be stored as part of the driver. Manipulating the listener should only happen in-scenario, and it should go back to position 0,0,0 when exiting a scenario.

Good. Fine. Writing up an .ogg parser and the buffer / source structures seems like a good stopping point. Larger-scale structure tomorrow. I'm going out for a beer.

> LATER NOTE: It was around this point that I decided that once I wrapped all this up I was going to split it into a separate library so that other people can compile and play .ogg files in a simple format for their games. After writing out the loadOgg stuff, I do not ever want to see the damn thing again, and I don't want others to be forced to either.

### Day 2 Evening

It is presently 10 PM, and I have just finished splitting up sources and buffers into their own files. I have made the discovery that audio sources are only capable of playing one buffer at a time - that is to say, one sound at a time, unless I do some weird and horrific mashup of buffers. This changes things, but not by much. This just means I'm going to need to generate a lot more sources instead of just the one. Definitely going to want some form of audio component so these can be tracked properly.

My current sound system actually does this in a really hacky fashion:
```cpp
void registerSoundInstance(flecs::entity id, ISoundSource* snd, f32 volume, f32 radius) {
	if (!id.has<IrrlichtComponent>()) return;
	auto irr = id.get<IrrlichtComponent>();
	ISound* sound = soundEngine->play3D(snd, irr->node->getAbsolutePosition(), false, true);
	if (sound) {
		sound->setVolume(volume);
		sound->setMinDistance(radius);
		sounds.push_back({ id.id(), sound });
		sound->setIsPaused(false);
	}
}
```
 but it should be properly managed by the audio driver itself rather than my state controller. Right now it keeps a list of sounds and the entities that caused that sound and updates their position accordingly. If the sound's done, it gets removed from the list. If the entity is dead, the sound stops moving, then finishes as normal. Ideally this behavior gets kept, albeit reorganized and de-stupided.
 
 11 PM. I've discovered that there is also an effects extension for things like reverb, and wasted 30 minutes googling where to get this extension before realizing it came with the SDK. I typed in `#include <efx.h>` and closed Visual Studio.
 
 > LATER NOTE: Pro tip for programmers out there: You are not going to commit anything useful to your repository past, like, 11 PM (assuming you sleep like a normal person instead of some nocturnal freak). I've lost count of the number of "git reset --hard" commands I've entered at midnight after writing some absolute garbage. Give up and go to bed. It will be there when you wake up.

### Day 3

> LATER NOTE: This entire mistake and waste-of-time debugging process was a direct result of the prior day's 11 PM commit.

"Silent fail" is the worst method of error handling.

I'm adding in some of the structure now - mapping filenames to buffer handles and whatnot, setting it up for later so that there are menu and music sources - and ran into a baffling error that took me *far* longer than it should have to track down due to OpenAL deciding it's better to silently fail than throw an exception. Essentially what was happening was this: in my constructor for audio sources, I used some OpenAL functions to assign various attributes to the source and generate the source. The problem *here* is that I also included a few sources as part of my driver class - for menu and music sources. The problem is that it uses those constructors *before* the audio driver itself gets initialized, which means that it spews errors trying to assign a source while the device itself doesn't exist.

The problem is that when my game broke as a result, it threw errors from the .ogg libraries because I was trying to de-allocate memory that I hadn't allocated in the first place, as a result of openAL failing to create a buffer because earlier it had tried to create sources on a nonexistent device. I found this out eventually with some error-checking on OpenAL, but couldn't figure out *why* it was running into issues until I went back and looked at some of the constructors. It's an easy fix and a rookie mistake, but I hate it when this crap takes longer than it should to debug.

> LATER NOTE: This section was written in a fit of frustration. In all honestly, having the audio library not crash my program every time it runs into an error is extremely beneficial, and so long as I'm using the alGetError() function correctly and flushing the error before trying to get a new one, it works great. There are actual benefits to having something simply swallow errors whole, if it doesn't break your program. It just made debugging a little more of an issue, and frankly that's due to my own negligence in error handling while using the library.

Awesome. Menu music now loads and de-loads appropriately, although there's a significant hang time there that makes me want to split it off into a thread or something (or come up with a slightly smarter method of when to de-allocate the associated memory). The structure for that was actually pretty simple; the in-game audio will be more of a pain, but it will be *basically* the same thing, albeit with a quick management system to be able to tie a given entity to the noise it's making. I've already written a version of that for my current system, so I think we're done here!

The last thing to be done is to give [Irrklang](https://www.ambiera.com/irrklang/) the boot (my previous audio library). Get *outta* here!

> LATER NOTE: This isn't to say Irrklang is a bad library by any means; it just doesn't quite provide the control that I wanted out of my audio management. If you're working on a smaller project or something that needs a much simpler sound library, I would highly recommend Irrklang. It was far, far easier to set up Irrklang than OpenAL. 

### Wrap-up

The audio driver is quite simple. It has a list of what it's currently loaded and buffers for each of those, and it keeps track of what music is currently playing. As I said, it has a buffer for game sounds and a buffer for menu sounds and will allocate / de-allocate sounds appropriately based on that.

The audio buffer keeps an internal map of the buffers to their filename strings. If given the same filename (trying to load the same sound twice), it will simply return the buffer that was already loaded. I *could* end up loading the same buffer twice in both the menu buffer and the game buffer, but eh. That shouldn't be happening in any case.

The audio sources keep track of a given source and its present values, and can play or stop sources. More features will presumably be added (like rewinding) as we keep going, such as reverb or various other sound effects, and all of that shouuuld be relatively simple to set up.

Overall this process was much less of a headache than I expected it to be; normally tearing out libraries and implementing new ones is a giant pain in the ass that takes way more time than I expect, but in this case it actually took *less* time than I expected and I got a lot more functionality out of it. I *will* be releasing my basic audio driver as its own library in the future; it might take a second to generic-ify it to make sure that it'll work as expected, but it should have some relatively simple functions for anyone looking to use this library.

### What's next?

More GUI work. More campaign work. Setting up GUI to go with the dialogue system. Writing out the construction bay mechanics and the associated dialogue. A whole bunch of really dull work, really. Bleagh. 

After that, though, there *is* a light at the end of the tunnel and I can *finally* get back to scenario design and weapon/ship design. Onward! *Onward, I say!*