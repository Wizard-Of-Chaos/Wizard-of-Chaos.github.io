### Managing Your Copy Constructors With Scaling

For most of the C/C++ style developers out there, they already know where this particular post is going just based on the title, so in the interest of allowing them to empathize with *something* in this post, let me start off by saying this -- I just tried to fix a bug I had in my heads-up display with radio contacts, and got it *almost* working, right? Not quite, though. I had to get up and drive for two hours in about twenty minutes, so I just sort of shrugged, called it there for now and got up to head out. Fifteen minutes into the drive I realized my error and just pounded my steering wheel in frustration because I wasn't going to get back to my goddamn PC to be able to *fix* it for hours and hours. We've all been there.

Anyway, the game's done.

No, really. It's *done*. With an asterisk that it needs polish -- I have a list of things I want to get done with it and a couple of extra additions, but the reality is that right now I can sit down and play the entire game beginning to end and hit the credits with all the scenario types. The rest is just polishing, bug swatting, efficiency, and tweaking. My web-cam wasn't on when I beat the final boss and went to the credits, but I was there, and as I recall I looked kinda like this:

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/whiskerfish.png "Why did we call these things catfish? I like 'whisker-fish' myself.")

The final few sectors involved some color-scheme changes for the background, and the finale sector is something I'm very proud of aesthetically. It's fun as hell to fly through, and while I don't feel the urge to spoil any of it, I do think it looks pretty cool.

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/finale_sector.png "Fug, dude, I'm purble.")

Actually, that screenshot is a bit out of date, because one of the things I also finished up was throwing out every god damn chunk of placeholder modellign and art, and that's the old Meteor design. The new one looks more like this.

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/meteor.png "Budget A-wing!")

On top of that all the art for the characters and backgrounds is done as well! Woo-hoo! Here's a shot of all of 'em:

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/headshots.png "Tag yourself. I'm the ugly rat thing.")

Very pleased with all of it, honestly. I imagine the next month or two will be primarily grunt work, but it's a relief to have every sector and fight available for testing and tweaking. But the tweak in question here is the importance of managing your copy constructors.

## Copy Constructors

For those of you who aren't experienced with coding, a *copy constructor* is something that defines how to copy (duh) a chunk of data around. For example, I might have the class `Cat`, with attributes of `weight`, `height`, and `purrVolume`. It'd look like this:

```cpp
class Cat {
    float weight;
    float height;
    float purrVolume;
};
```

For most simple classes such as this one, the copy constructor is implicitly defined because it's absurdly easy to know what needs copying and how. If the class had some other pointers or resources, though, I would need to define a constructor that describes how you can build one `Cat` from another - a blueprint of existing design.

There's a classic rule in computer science called the Rule of Three, where the logic is this: if your class is complicated enough to need a destructor (to tell it how to delete itself), a copy constructor, or a copy assignment operator (how to copy a `Cat` into another, existing `Cat`) you need to define all three because they all have the same problem.

## How It Relates

This cropped up as a problem when I was designing the fleet sector, which by necessity involves a hell of a lot of ships flying around and doing things all at the same time, and I noticed my FPS would slow down pretty massively during these brawls but wasn't really sure why. There weren't actually all that many things on the screen by comparison to the debris sector where there's just garbage *everywhere*. This called for a tool I hadn't needed to pick up yet: CPU profiling.

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/cpu_profile.png "Oooh, charts!")

CPU profiling is a great debugging tool, and all it really does is just keep track of how much time any given function spends on the CPU. If used correctly this will allow you to pinpoint exactly what's so inefficient about your code. Prior to this, I used it to catch an extremely stupid string error that was slowing gameplay to a crawl just by virtue of slow string editing where it had no business being edited. A useful way to visualize what's going on is called a *flame graph* where it extends outward based on what nested function is calling what. Here's one from the current build of the game:

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/flamegraph.png "This sort of bugs me. I think I've got a few other issues with game logic in terms of CPU time. I might need to thread some of it better.")

In this function it tells me there are two major things eating up CPU time under `mainLoop`: some Irrlicht .dll (graphics and other management under the hood), and my own logic update for `GameController`. This is not particularly surprising information. In this case I followed the chain of logic all the way down to see just what the hell was eating so much time and found it was a call to `get_mut` for my `WeaponInfoComponent`.

In the Flecs setup, the `get_mut` call returns a *copy* of the component you ask for that will then get merged into the full component at a later point in time, in order to preserve general thread safety. You can get a constant pointer with regular `get`, but if you want to change it, a copy ends up getting created.

## The Inefficiency

What that told me was that something about the way it got copied was massively inefficient and was eating up most of the time in the logic loop. So just what the hell was going wrong? You may remember that in [an earlier post](https://wizard-of-chaos.github.io/2023/10/02/wepfire-rework.html) I rewrote the weapon firing systems to use a list of extra functions on hit and whatnot. Let's go take a look at the internals of that component (from an earlier commit):

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/weaponinfocomponent.png "LENGTH JUMPSCARE!")

What the hell? This thing's huge! There's a lot of data that gets copied around whenever something wants to fire. It goes further down, too, for a list of update callbacks, hit callbacks, and a firing function. It may not look like much, but when forty of these are calling those up to eight times per tick each (six mounted guns, one heavy, one physics) it gets spooky. List copying is pretty expensive, and so is just copying a lot of this crap all at once. So what can be done?

Well, let's go back to the difference between `get` and `get_mut`. There's a lot of data there that *isn't going to change*, or at least not often (I have yet to write a weapon that fundamentally reorganizes itself whenever it shoots). The hardpoint type, weapon type, and damage type are not going to change. Neither is the accuracy or lifetime or speed. Nor the max clip or max ammo or the reload time -- hey, wait a minute, *none* of this changes very frequently except for a few exceptions. The vast majority of this can be used for a `get` call, which is much faster. `get_mut` only needs to apply to a few of these things, like whether or not it's fired or the current firing direction.

## Solution

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/solution_commit.png "My commit logs usually aren't this useful.")

In this case, the solution is easy -- we need to split off the commonly changed data from the mostly-constant data, and then only call copy constructors on *that* part of it. The really expensive copying, the lists in the original, don't need to be copied at all.

```cpp
struct WeaponFiringComponent
{
	bool isFiring = false;
	bool hasFired = false;
	vector3df firingDirection = vector3df(0, 0, 1);
	vector3df spawnPosition = vector3df(0);

	u32 ammunition = 0;
	u32 clip = 0;
	f32 timeSinceLastShot = 0.f;
	f32 timeReloading = 0.f;

	IParticleSystemSceneNode* muzzleFlashEmitter;
	IMeshSceneNode* muzzleFlash;
	ILightSceneNode* muzzleFlashLight = nullptr;
	f32 flashTimer = 0.f;
};
```

These are all the things that will change: firing direction, the muzzle flash from the gun, various timers, and ammunition counts. *This* is what `get_mut` needs to be called on, and the original info component is now left mostly alone. As such, the speed of the game increased pretty massively as soon as this went down for the fleet sector, and it had some benefits for the other sectors as well (notably, with turret efficiency). The moral of the story is always make sure to consider *exactly* what you need for something, and then, if possible, get exactly that and no more. This particular slowdown was the equivalent of hauling around my laptop everywhere I went because I needed a calculator.

## What's Next?

Release, soon. Hoping around spring sometime, depending on how much of a pain in the ass polishing is. I got bugs to swat, other lurking efficiency problems to catch, Steam to set up and some network issues to iron out (testing for Steam socket use cases is near impossible when you... aren't on Steam). We press on.