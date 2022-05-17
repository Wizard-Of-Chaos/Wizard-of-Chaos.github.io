## Carrier Implementation

Today I decided to take a crack at a basic carrier implementation. What I want out of carriers, effectively, is effectively an oversized ship that also spawns lesser ships. This is pretty simple. Ideally, a carrier would also have turrets that swivel and shoot at anything nearby, but I'm not ready to touch that until a serious overhaul of the ECS system we're using (the current plan is to start using flecs instead of our current, markedly ghetto setup).

So, what exactly does a carrier need? Ideally it needs some ships, and it needs to know what _kind_ of ships it can spawn, and how fast, and just where the hell to put the suckers anyway. Those are all pretty easy to achieve, and were done with a couple quick variables and the use of a ShipInstance class that keeps track of weapons, ammo, and other stuff. In this case, though, the instance is being used as a _template_ that other ships can come out of. All pretty simple to think about; I want a ship that spawns a bunch of identical ships and it does it at this rate from this point. Easy.

Problems started arising, as they tend to do, when actually _implementing_ my brilliant-yet-simple plan. How the hell do I load one of these things from memory? Well, I need to overload the ship loader and make it so that it can also load a carrier. But it also needs to be able to load _more_ than basic ship info, so we need to tack on some extra stuff to do that too, yadda yadda yadda, effectively it turned into a whole damned production even before I could write any actual code governing carriers.

### Carrier System

```cpp
const u32 CARRIER_MAX_SHIP_TYPES = 4;
const u32 CARRIER_MAX_TURRETS = 8;

struct CarrierComponent
{
	f32 spawnRate;
	u32 shipTypeCount;
	ShipInstance spawnShipTypes[CARRIER_MAX_SHIP_TYPES];
	u32 reserveShips;
	vector3df scale;
	f32 spawnTimer;

	EntityId turrets[CARRIER_MAX_TURRETS];
	vector3df turretPositions[CARRIER_MAX_TURRETS];
	vector3df turretRotations[CARRIER_MAX_TURRETS];
};

void carrierUpdateSystem(f32 dt)
{
	for (auto id : SceneView<CarrierComponent, ShipComponent, FactionComponent, IrrlichtComponent>(sceneManager->scene)) {
		auto carr = sceneManager->scene.get<CarrierComponent>(id);
		auto irr = sceneManager->scene.get<IrrlichtComponent>(id);
		auto fac = sceneManager->scene.get<FactionComponent>(id);
		carr->spawnTimer += dt;

		if (carr->spawnTimer >= carr->spawnRate && carr->reserveShips > 0) {
			ShipInstance inst = carr->spawnShipTypes[0];
			vector3df spawnPos = getNodeDown(irr->node) * 15.f * carr->scale.Y;
			vector3df spawnRot = irr->node->getRotation();
			carrierSpawnShip(inst, spawnPos, spawnRot, fac);
			--carr->reserveShips;
			carr->spawnTimer = 0;
		}
	}
}
```

This is pretty close to what I said initially; obviously the loading and implementation thereof led to much frustration and howling of invective, primarily at my own incompetence. It gets a ship instance (since I've set it up to be able to spawn more than one type of ship) from an internal array of ship types it has and then spits it out at a specific point below it. Easy.

Most of the wailing and gnashing of teeth in functions like these occurs not on the actual code itself, but on loading attributes for a specific carrier and getting the whole damned thing to render properly. Actual game logic is pretty easy to come up with, but unfortunately for myself I'm not writing Dwarf Fortress and can't get away with minimalistic graphics. The world weeps. Or maybe just me.

---
### Check out this cool bug

One odd bug I did notice while dealing with this arose from my idiotic usage of my own health component. See, I have a function set up to initialize health components and make it so they play nice with the damage tracking system.
```cpp
void initializeHealth(EntityId id, f32 healthpool)
{
	auto hp = sceneManager->scene.assign<HealthComponent>(id);
	auto dmg = sceneManager->scene.assign<DamageTrackingComponent>(id);
	hp->health = healthpool;
	hp->maxHealth = healthpool;
}
```
Bam. It couldn't be simpler. The problem was that I was not actually _using_ this function on my initial crack at carrier generation and getting it displayed in the scene, and was instead just assigning a raw health component to the ship. This broke a few things because now the system had no idea how the hell to handle damage for this particular carrier entity, and threw me a couple crashes, whereupon I slapped my forehead cartoon-style and fixed it. It horked up an error at a specific line, though, that allowed me to catch an entirely different bug in my impact-damage code:

```cpp
	if (dmgA) dmgA->registerDamageInstance(DamageInstance(B, A, DAMAGE_TYPE::IMPACT, kinetic / 2, device->getTimer()->getTime()));
	if (dmgB) dmgA->registerDamageInstance(DamageInstance(A, B, DAMAGE_TYPE::IMPACT, kinetic / 2, device->getTimer()->getTime()));
```
The explanation here is that dmgA and dmgB are the respective damage tracking components of their entities, and this is the damage code for impact damage. It registers an instance from the entity to each other, sets the type as impact damage, sets an amount, and says when this took place.

You see it, right? There's a very, very simple and silly bug causing it to break here, and it's the fact that in both cases dmgA is taking the damage, so one entity is taking _all_ the damage from the impact (this is why "kinetic" is split in two, so both entities take impact damage) and the other gets nothin'. This swallows any and all errors except in one case - where dmgA doesn't exist and it tries to register a damage instance for dmgB, but on dmgA because of the bug. So this stupid typo actually enabled me to catch a much more serious bug. This sort of thing is frustrating, yet oddly satisfying.

---

What's next? Updated scenarios. Right now it's just shooting a whole bunch of hostiles and I want to expand on that a little more.
