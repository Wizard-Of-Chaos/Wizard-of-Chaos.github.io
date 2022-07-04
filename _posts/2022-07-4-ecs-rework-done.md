## ECS Rework Completed (mostly)

It has been a rather irritating month. First yours truly moved to a different state, which ate up most of the month, and after that there were many, many bugs to be found with the rework of the ECS system. Far, far more, actually. Turns out when you're designing something to be thread-safe, you take certain precautions that you wouldn't otherwise.

One of the big things that the flecs system we're using now does is a deferred changes system. If I decide I want to make changes to a given component on an entity, it'll give me a *copy* of the data which it will then merge with the original at the next sync point. This busted a lot of things in irritating and hard to debug ways - my prior system was pretty much single threaded, so I could be damned sure that everything happened in the order I gave it. This is also much less *safe*, given that I was just throwing around mutable data willy-nilly, but, y'know, when you're doing your code yourself you tend not to notice these things. It should make it less of a pain in the future for me, but present me is goddamned annoyed that someone put up some fucking safety railings.

The first and most irritating thing this broke was the actual control systems for the ship. My input (from the keyboard) and my input components (that tell the ship how to behave given the keyboard status) were not cooperating, and my first compilation of the new systems caused my ship to spin wildly, shoot wildly, thrust wildly, yadda yadda yadda because everything was profoundly hectic. Eventually I wrestled control of my ship again.

Second thing this broke that took me a hell of a time to notice was carriers. Spawning in new entities had changed drastically. I've shelved this for later, probably within the next few days or so, but my guess is that data being initialized sucks and my attribute readers for carriers are broken (the things that pull data from file and insert it into the game).

Third major thing this broke was the AI. My system for the AI involved mutating the ship systems (telling it to pitch up, pitch down, that sort of thing) based on the input. This simply refused to work as intended and I had a hell of a time trying to get my AI to actually fight me again.

Solving the AI problem was a long, frustrating process, and I actually had no idea that flecs did the deferred thing on data at first, so I had no idea what the hell was going wrong. In the middle of this I pulled all of my crap out of my previous abode and dumped it into a new abode up in a different state, so not a hell of a lot of work got done for a few weeks while I scrambled around fighting an entirely *different* sort of problem.

Eventually I actually read the documentation, you know, like some kind of goddamned *peasant*, found out about the deferred changes so it can play nice with threading, and the rest of it clicked into place. Since then it's just been bug-swatting on wherever I'm trying to mutate my data to make sure that it's working as intended and playing nice with how flecs wants it to be. As it stands as of about... thirty minutes ago (written at 5:40 PM EST, July 4, 2022), I can now do goddamned near everything I used to be able to (outside of the carrier brawls, but again, next few days!). Happy to say the ECS refactor has been completed. God willing, I will never have to do any other rework of this scale on this project *ever again.*

---
### What's next?

I have seen the future, and it is a cold beer and a nice sit-down, and some Stargate: Atlantis on the TV. After that I'm going to go watch the fireworks that some hick is going to set off, and potentially get some of my own to eliminate this goddamn groundhog under my deck *once and for all.*

After that I'm going to fix carriers and then get cracking on more scenario work. My esteemed partner will be working on networking code to see if we can get co-op running. Specifically I'm going to take a look at environment generation to improve it a bit, and maybe adding some more complicated objectives to work with. Once I've done some work there to make sure it's extensible. Some more alien ship designs wouldn't hurt, either, and I'd like to see if I can randomize the alien guns and ships so it's not all so goddamned same-y.

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/construction.png "This button has been on the scenario screen for months. It's had it too good for too long."

Once I've looked at that, I'm going to decide what to do with this button. Ideally I'd like a system where you can break down weapons and ammo and turn them into scrap for different weapons, ships, whatever. Then probably boss fights after that... but that might be a ways off. We're getting there. I'm Dr. Rodney McKay up in this house. Everything is doable.