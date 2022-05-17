## Scenarios and Enemy Carriers

I'm trying to iron out some issues I have with the current scenario generation. There's no real problem with it, outside of the fact that there's not _enough_ of it, which is going to be a recurring theme in game development. "What, you mean I have to _write_ this? It doesn't just come out of the box?" Alas, no. The basic problem can be explained with a quick code snippet that I use for scenario setup:

```cpp
void setScenarioType(Scenario& scenario)
{
	std::cout << "Setting up scenario type... \n";
	switch (scenario.type) {
	case SCENARIO_KILL_HOSTILES:
		setKillHostilesScenario(scenario);
		break;
	default:
		std::cout << "No valid type! Defaulting to kill hostiles. \n";
		setKillHostilesScenario(scenario);
		break;
	}
}
```

You'll notice there's only one type of scenario and if it's literally anything else it will default back to a kill hostiles scenario. That means that right now, there's only one way to play the game - you see a bunch of bad men, you send them directly to Hell (or vice versa), you win.

This doesn't work for a carrier objective. If I have an alien ship that's spawning a bunch of smaller ships, and my objective is to blow the enemy carrier out of the sky, the _second_ I'm done shooting it, the scenario is considered over and I just win. That's it? What about all the leftovers? There's a whole goddamned horde of ships that I still have to deal with... or rather, that I think I should have to deal with.

So what's the first thing I need to do? Define some parameters to generate a carrier-killing scenario? Set up the scramble scenario so it finally makes sense? No. The _first_ thing I need to do is make a placeholder model for an enemy ship. To my eternal frustration, once again I will need to open Blender and do something creative.

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/ayycarriercap.png "I swear I'm a better programmer than modeller.")

Sure, fine. Modelling isn't really my strength, and any time I'm forced to do it I get annoyed with it. Realistically, once I'm dead certain that the actual game is fine and I don't have to write any serious code, I'm going to go back and give these things a pass, but for now we're all just going to have to suffer through the model equivalent of a sketch. Textures, too.

Now that I have my preschooler-art model, I can actually work on functionality. Given that my scramble scenario necessarily requires an enemy carrier to shoot at, we can start there. I need to add in a new handler for the case of a scramble - that's easy enough to set up. Now, this might get really complicated and hard to follow, so bear with me:

```cpp
void setScrambleScenario(Scenario& scenario)
{
	std::cout << "Setting up a scramble...";
	scenario.objectives[0] = createAlienCarrier(1, scenario.enemyStartPos, vector3df(0, 90, 0));
	std::cout << "Done.\n";
}
```

Uh, no, that's... that's it really. The environment is handled elsewhere, and so is the player, so really the only important factor in a scramble is the enemy carrier. Trouble is, this is the exact problem I mentioned at the start: the enemies who spawn in from the carrier are not considered... but scramble scenarios DO work!

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/ingameayycarrier.png "'Is that a Romulan ship?' my friend asks me.")

---

There is, of course, the elephant in the room of not tracking the damned fighters - but following the logic completely is going to be a big damn production in its own right worthy of a separate post later.

However, actually having a big ol' carrier to shoot at and deal with led to its own problems. I started shooting at the thing, and was barely making a dent in its health bar, and just parking your ass somewhere and firing thousands of shots at a carrier to bring it down isn't really all that fun. In the future I'm planning for turrets and whatnot, as I said last time, which should probably mark the carrier's health bar their own selves. I started looking for other solutions on how the hell to kill the things effectively. I used the ship's physics weapon to fire an asteroid at the carrier, which made an amusing noise and a significant dent in the health bar, so maybe that's an option for the future. The trouble with that is given the variety of environments I intend to have, it might not always be practical to have some big damn rocks around to throw. The situation bears thinking upon.

The most entertaining part of the design and seeing it happen came from a funny interaction with the AI. My current AI does its level best to get behind you and shoot you to death. The carriers are running the default AI, but they don't have any guns, so they just accelerate towards you and try to get behind you. Carriers, however, are _way_ too big to pull this properly, so what actually happened when I saw the human carrier try it was that it just rammed straight into two enemy fighters and crushed them like beer cans. As soon as it pivoted to try and deal with the third fighter, it accidentally swatted it with the forward section and sent the enemy ship flying off into the distance, and an instant later the scenario was over as the fighter slammed into something. This behavior is so incredibly entertaining that I'm keeping it. As far as I'm concerned, if you're slow enough to let a carrier try and park in your backseat, that's _your_ problem.