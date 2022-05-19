## Objectives and Flexibility

In a couple of the last posts I've talked about specific problems that I had with the way objectives got handled. At its core, it was a very simple system. There's a certain amount of objectives to tackle and when all those objectives are gone, the scenario is over and you win - or, you fail miserably, you die like a dog, and you start writing an angry post about how unfair this game is.

This system, however, is inflexible, and only really accounts for one type of objective - shooting people. The way it determines whether an objective has been "completed" is whether or not the entity ID attached to that particular objective is still valid, like so:

```cpp
bool GameController::checkRunningScenario()
{
	for (u32 i = 0; i < currentScenario.objectiveCount; ++i) {
		if (currentScenario.objectives[i] != INVALID_ENTITY && !sceneManager->scene.entityInUse(currentScenario.objectives[i])) {
			currentScenario.objectives[i] = INVALID_ENTITY;
			--currentScenario.activeObjectiveCount;
		}
	}
	if (currentScenario.activeObjectiveCount == 0) {
		return true;
	}
	return false;
}
```

This is, again, fine for shooting people, and if I really wanted to I'm sure I could set it up so that any sort of "salvage" mission works in this way as well (once a chunk of debris gets salvaged, it would be removed, after all), but for any other kind of objective it won't work. If I want to have an objective where I need to evac to a given point, it won't work, and it doesn't consider at _all_ mopping up any fighters that may have gotten away from a carrier before you blew it up. Objectives are static and unchanging - clearly, it needs work.

I set out to try and fix this, and immediately ran into a mental wall on just _how_ I wanted to do it. My first thought was some kind of objective structure in a list detailing what kind of objective it might have been and the associated ID with that entity - for instance, a DESTROY struct might have looked like this:

```cpp
enum TYPE
{
	DESTROY,
	GO_TO,
	SALVAGE
}
struct Objective
{
	TYPE type;
	EntityId id;
};
```

After that I could keep a list of those and eval them based on the type. This is sloppy and doesn't quite do what I want; the structure here would be keeping a list of objectives and removing them when they've been evaluated. This would result in objectives that have been completed being completely ignored - like, for instance, in the go-to example, a ship could go to the evac point, leave, _then_ shoot everyone, and when they were done they wouldn't need to move anywhere because the objective would have been considered completed. Furthermore, keeping a list of objectives is what got me into this crap in the first place.

I gnawed on the problem for awhile and then realized that, no, I was already keeping a damned list of objectives. The entities themselves. All I needed was a specific type of objective _component_ that could be attached or removed from entities and evaluated accordingly. That way when a carrier spawned additional ships, they would include the carrier's objective component and be considered that way. The objective component itself is stupidly simple.

```cpp
struct ObjectiveComponent
{
	OBJECTIVE_TYPE type;
};
```

That's it. That's all it does, and all it needs to do. Obstacle components (asteroids, debris, gas clouds, etc) have a similar setup. Once I have that, I can write functions to evaluate the conditions for a given type of objective as part of the system.

```cpp
bool objectiveSystem(f32 dt)
{
	bool ret = true;
	for (auto id : SceneView<ObjectiveComponent, IrrlichtComponent>(sceneManager->scene)) {
		if (!isObjectiveCompleted(id)) {
			ret = false;
			break;
		}
	}
	return ret;
}
```

This will return true if every objective in the scene returns true. The function to evaluate this is pretty easy as well:

```cpp
bool isObjectiveCompleted(EntityId id)
{
	//if there's no component / no entity available, either this function is unnecessary or the thing has been destroyed
	//in either case, return true and exit the premises as quickly as possible
	if (!sceneManager->scene.entityInUse(id)) return true;
	auto obj = sceneManager->scene.get<ObjectiveComponent>(id);
	if (!obj) return true;

	switch (obj->type) {
	case OBJ_DESTROY:
		return false; //if the entity is valid, clearly it hasn't been wrecked yet
	case OBJ_COLLECT:
		return collectObjective(id);
	case OBJ_GO_TO:
		return goToObjective(id);
	default:
		return false;
	}
}
```

My favorite piece of this is just it blatantly returning false on a destroy objective, since originally the system worked the same way - if the entity still _exists_, clearly it hasn't been destroyed yet, so returning false is the best method. Something about it just amuses me, I don't know why. This type of setup will work great in the future and will allow me to chain objectives together, generate new ones on the fly, and create more complex scenarios in general - since one of the great benefits of the ECS system is quickly adding and removing components. If, for example, I want the player to ignore all ships in the area and skedaddle, I'd write a function to peel off any objective component that was OBJ_DESTROY, and just keep the GO_TO objective as the evac point. Simple!

There is one problem I do have with my new setup, though; displaying it as part of the HUD. I'm not entirely sure how I want to handle showing these objectives, or keeping track of them in a way that's easy for the player to see. Obviously, it will take the form of a chunk of text floating on the HUD somewhere telling you how many enemies you have left to destroy, but updating that on the fly and adding new pieces might take some thinking as to how to do it properly and efficiently. The objective system will likely need more logic since, as it stands, it does so lazily; if _any_ objective has yet to be completed, the loop breaks and the scenario continues. I'd like to be able to loop through them and update the HUD accordingly. That's for later, though.

---

With the rework on objectives, the game is pretty much functionally done. I can't think of many other features that I still need to actually _add_ - having a construction bay on the carrier to craft / disassemble weapons and enhancements would be nice (as well as just, like, an enhancement system in general) but as far as large-scale shoot mans scenarios go this is about where I want to be - I can even set up larger "boss battles" at the end of a sector with the carrier code from earlier and the new objective system.

In the next few days the trick will be adding _more_ - more guns, more ships, more physics guns, better scenario generation, more complicated scenario objectives. All of these functions already exist and just want extension. One of the first things I'd like to tackle is some additional stupid weapons and a method to destroy carriers properly - whether that takes the form of a missile slot or additional goofy weapons remains to be seen.