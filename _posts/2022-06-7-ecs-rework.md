## ECS Refactoring

My esteemed partner on this project has finally returned from the land of the dead (he's finished his Master's) and is now available for some of the other crap we had planned on this project. First and foremost on the housekeeping list when we both had the time was a complete tearing out of the ECS system, given that our original ECS system, while fast, lacked a bunch of utility that we really wanted to be able to have. Hierarchies were a big one - our system didn't have much of a notion on parenthood, or one entity owning another, and the solution we had in place prior to that was pretty hacky. Henceforth we're going to be using FLECS, which is a fantastic ECS library that includes many other features that neither myself nor my partner have the wit to do... well, at least, not in any reasonable time frame.

Unfortunately, given how thoroughly the ECS is worked into *literally every aspect of the game*, this is akin to tearing out a game engine and replacing it. Suffice to say there's a lot of drudgery and a lot of re-writing where you can't help but think to yourself, "I could be replaced with a find-and-replace machine right about now". Where the hell is the fun in replacing "EntityId" with "flecs::entity"? There is none.

There is an upside, though, and that's that I'm getting to look at old chunks of code I haven't touched for at least a month and some features I haven't touched since the beginning of the project. There is much confusion and cursing of myself, and often the phrase comes up, "Why the hell did I write it like *that?* " I'd like to share, here, some of the stupider chunks of code that were *also* replaced while I was in the files.

```cpp
	rbc->rigidBody.setUserIndex(getEntityIndex(entityId));
	rbc->rigidBody.setUserIndex2(getEntityVersion(entityId));
	rbc->rigidBody.setUserIndex3(1);
```

This chunk of code appeared multiple times in the original version, whenever I set the entity ID on a rigid body for use with bullet physics. An entity is a 64-bit integer with a *version* and an *index*, each of which comprised 32 bits of the integer. It gets broken up into chunks and placed in 32-bit integers on the rigid body for use with user stuff.

You may notice the third piece there. What's it doing there? Hell if I know. I asked my partner, "Why the hell did we put the extra one there?" and he simply shrugged. The notion is lost to time. My guess is that we needed to use it to see if the rigid body *had* an associated entity, but it was literally not possible for the rigid body to *not* have an entity, so hell if I know.

Irritatingly, this chunk of code was also written down in I think four separate places instead of slapped into a function call. Goddamn it.

```cpp
	auto ghost = sceneManager->scene.get<BulletGhostComponent>(id);
	if (ghost) {

```

This pattern appeared a bunch of times, where I would get the associated component, and if it were there (and not a null pointer) I'd do some operations with it. This bit me in the ass in quite a few places where there was quite a bit of wasted logic due to not having a simple check for "Does this entity have the associated component?" Flecs has it, we didn't, and finally all these stupid little checks are rearranged into a more sane fashion.

```cpp
	if (proj->type == WEP_MISSILE) {
		explode(irr->node->getAbsolutePosition(), 1.f, 1.f, 20.f, proj->damage, 100.f);
	}
	if (proj->type == WEP_PHYS_IMPULSE) {
		gameController->registerSoundInstance(impacted, stateController->assets.getSoundAsset("physicsBlastSound"), 1.f, 200.f);
		explode(irr->node->getAbsolutePosition(), 1.f, 1.f, 80.f, proj->damage, 500.f);
	}
```

This was found in the collision checking system after identifying that there's been a projectile impact and is a great example of how to not write code. This offensive creature was found right *before* the system that handles individual effects for weapons, and left me *baffled* as to why it wasn't simply just put in the actual goddamned function *literally designed to handle these cases*. I think this is lingering slapdash code from the initial implementation of these weapons. The explosions have been moved to their proper places.

```cpp
	auto rbcA = sceneManager->scene.get<BulletRigidBodyComponent>(A);
	auto rbcB = sceneManager->scene.get<BulletRigidBodyComponent>(B);
	auto dmgA = sceneManager->scene.get<DamageTrackingComponent>(A);
	auto dmgB = sceneManager->scene.get<DamageTrackingComponent>(B);
	
	if (!rbcA || !rbcB) return;
```

Here's another example of where things were being handled somewhat stupidly. It got all the appropriate components for the system and then if the rigid body components weren't there, it just exited the function. Why the hell bother calling the functions for the tracking components, then? Don't we only need those later? This is just poor organization on my part.

```cpp
	auto wepParent = sceneManager->scene.get<ParentComponent>(proj);
	if (!wepParent) return true;
	if (!sceneManager->scene.entityInUse(wepParent->parentId)) return true; //set up like this because im not sure if checking them both in the same if will throw a segfault
	auto shipParent = sceneManager->scene.get<ParentComponent>(wepParent->parentId);
	if (!shipParent) return true;
	if (!sceneManager->scene.entityInUse(shipParent->parentId)) return true;
```

The collision code really is full of egregious shit.

This is what I was talking about with the hacky parentage patch job. This was to check whether or not a projectile would be hitting the ship that fired it. Thing is, the way it was set up, the projectile was the child of the weapon, which was the child of the ship, and the IDs for each were stored in "parent components", so you end up with this weird logic chain where first you need to check if the parent component even *exists*, and then you need to check if the *second* parent component exists. This got called on every collision, by the way. Every single one. Past-me, you *piece of shit.*

```cpp
	if (event.EventType == EET_KEY_INPUT_EVENT) {
		for(auto entityId : SceneView<InputComponent>(sceneManager->scene)) { //Passes key input to the input components

```

Another chunk of really silly logic. This got ran on every key event; it would iterate through all entities with an InputComponent and adjust their values accordingly.

The silly part here is... the input component was only ever on the one entity: the player. There's no need for a loop here. There's no need for *anything* except a cached ID. My only defense here is that I was in the throes of ECS worship at the time and wanted to make *everything* a system... even things that only ran *once.*

```cpp
EntityId getPlayer()
{
	for (auto id : SceneView<PlayerComponent, IrrlichtComponent>(sceneManager->scene)) {
		return id; //just grab the first thing you see
	}
	return INVALID_ENTITY;
}
```

Hahahaha, oh wait, no, I *did* have a function to get the player entity! It, too, was a system! Well, at least it's *consistent.* Except for the part where I didn't use this function.

```cpp
auto particles = scene.get<ShipParticleComponent>(entityId); //need to check to make sure this exists
```

I never checked to make sure it exists.

```cpp
it = gameController->sounds.erase(it); //I want to re-work the std library so instead of "erase" it's "whip". Whip it! Whip it good!
```

I think I was drunk when that comment was written.

---
### ECS rework is nearly done, though

Hopefully we'll be able to get started on networking soon. Tearing out the whole system is going to lead to many, many stupid bugs, and I'm not looking forward to spending all my time stomping the new and strange errors, but we'll get it working before long!