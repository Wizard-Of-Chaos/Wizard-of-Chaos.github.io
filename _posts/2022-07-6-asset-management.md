## Asset Management and Sound Updates

One of the first things I wanted to get started on, now that the game is actually functional and playable again, was a rework of asset management and an update to the sound-system infrastructure. The sounds prior basically used the same sound for every sort of weapon available, and I wanted to have the option to customize that slightly - every weapon should have its own individual firing noises and impact noises. My original solution was... half-assed, shall we say.

```cpp
if (wepInfo->isFiring && (wepInfo->timeSinceLastShot > wepInfo->firingSpeed))
{
	if (wepInfo->usesAmmunition)
	{
		bool firing = ammoFire(it.entity(i), wepInfo, it.delta_time());
		if (wepInfo->type == WEP_KINETIC && firing) gameController->registerSoundInstance(entityId, stateController->assets.getSoundAsset("gunSound"), .3f, 10.f);
	}
	else
	{
		gameController->registerSoundInstance(entityId, stateController->assets.getSoundAsset("laserSound"), .7f, 10.f);
		createProjectileEntity(wepInfo->spawnPosition, wepInfo->firingDirection, entityId);
	}
	wepInfo->timeSinceLastShot = 0;
}
```

Mmm. You know, I have the distinct impression this could have been written a bit better. I actually can tell you my thought process on this, and it's a pretty common one in programming. Essentially, when I initially wrote out the firing noises for weapons, I only had the one weapon - the plasma blaster. So I made it so that every time the weapon actually fired properly, it shot off a laser noise when doing so. But then I wrote out the machine gun and the shotgun, and I didn't want those making a laser noise, so I instead added a quick check to make the gun noise instead. This is a horrible patch job that needs addressing. Thankfully, it's pretty easy; I just set it up so that the sound correctly fires when spawning the projectile and uses the weapon's appropriate sound.

There was another problem that I needed to solve here, too, and it was that of asset management. If you're not all that big into game development, what I mean by "assets" is models, textures (images), and sounds, all of which take up memory in the program. Unfortunately for myself and every other game dev out there, memory is one of those "finite resources" people are always whinging about, and the files you use for textures and models can get *huge,* so making sure that you manage your memory properly is extremely important. This isn't a problem for me right now, because the textures and models I've made are fairly small in file-size (mostly because they look like shit and were designed by an amateur), but it absolutely can be in the future.

Thankfully, the Irrlicht engine was written by smart people. When I want to grab, say, a texture, I would write it out like this:

```cpp
ITexture* myTexture = IVideoDriver->getTexture("textures/myTexture.png");
```

This gives me a pointer to the texture in memory that I can do what I please with. What happens if I call it twice, though? Does it load the texture twice into memory and piss away memory? Nah. Irrlicht's smarter than that. When I call the function a second time it will give me back the same pointer - Irrlicht has cached the texture for later so I don't have to repeatedly load it. Useful if you're creating, say, an armada of space ships. It does the same with models and my sound engine does the same with audio. This is pretty typical in order to not waste space.

What happens when I shoot everyone in sight and finish the scenario, though? I might have gotten rid of all the asteroids and debris and ships from the entity-component system, but their textures and models are still loaded into memory - even though I'm just on the campaign menu. They've got no reason to be in memory. Unfortunately the Irrlicht devs aren't writing my game for me and don't care about my specific use case. That means I have to get a little smarter with this.

What I have to do is keep track of this stuff myself and tie it to a specific identifier for... well, whatever is using it. In the case of my Tuxedo ship, maybe that's just the string "Tuxedo". So what I've done is I've written an asset manager for just this sort of thing. Now, when I load a specific texture for a ship, I can load it as normal, but I'll also track it in my asset manager. Then, every time the game closes, I can flip through all the assets I've loaded for that scenario and take them out of memory.

---
### Was this a good use of my time? 

Well... at the present moment... kinda? The entire game presently takes up maybe ~50 MB of disk space. If I loaded every single asset I had into memory and kept it there, it would eat up about ~40 MB. When I run a scenario, that eats up maybe ~600 MB - most of which is managing the actual *entities themselves,* which got deleted *anyway* when the scenario ended. So I've effectively made sure to be able to save myself 40 MB at the present moment. However, as I keep adding assets for use (and hopefully improving on existing ones), that number is going to keep going up, so hopefully I won't have any leaked memory in the future. Or at least, *less.*

---
### So why bother with this now?
Because I had most of this in place already, but sounds were poorly handled. Because of how flecs shuffles entities and components around in memory I kept running into errors when I simply stored a pointer to a sound source as part of the weapon info component - say, the firing noise. However, my asset management system was built for loading in stuff at the *start* of a scenario, and was inefficient for doing it on the fly, and moreover had no specific functions for making sure I could grab the appropriate sound for a *specific* weapon. It has all this now, and my bullets go BLAM and my lasers go PEW and my impulse blaster goes THOOM.

Besides, this crap needed a rewrite eventually. Might as well do it now when it's less of an issue as opposed to later when it's horribly entrenched.

---
### Other additions

Besides pissing away time on fixing problems for my future self, I also slightly re-worked explosions. Previously, the system for explosions was just an emitter that spewed a generic particle effect. The effect faded over time before removing itself from existence. This was quite bad at showing how big the radius of an explosion was or even really portraying an explosion in general. What it needed was a sense of volume.

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/newexplosion.gif "Ka-BANG!")

I've kept the particles here (for now), but also added a small sphere that expands outward and gradually disappears as the explosion goes onward. I can make the textures on the sphere more interesting (and add more particle stuff later), but this is much closer to what I want in terms of demonstrating how close you should get to things blowing up.

In addition to this, many of the present sound effects were made by doing little "pew!" noises with my mouth into a headset mic. This has been rectified and the guns no longer sound like they have a learning disability. Many thanks to the dude helping me with sound design. [Check out their stuff here.](https://what.bandcamp.com/)

### Next up

Scenario work. The generation functions have had it too good for too long. I need to expand the scenarios to involve more flying, trickier obstacles, and different *types* of obstacles. My esteemed partner, meanwhile, will get cracking on serialization for our networking challenges.