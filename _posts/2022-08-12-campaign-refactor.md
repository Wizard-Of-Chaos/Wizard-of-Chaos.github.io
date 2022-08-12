## Refactor on Campaign

Refactors are probably one of the more irritating aspects of programming, but they're vitally necessary. In this case alongside getting started at a new job and continued home renovations, I've had to do some of the soul-crushing labor inherent to refactoring my campaign management - but hey, it's *done* now and I can go do something interesting with my time. It needed to get done, but I'm not going to talk about it for too long because the details are... well, details. There's nothing cool there, just lines of code that got chagned around and reorganized.

If you're not much of a programmer, you might not be familiar with the term "technical debt". Technical debt is what happens when you decide on a quick slapdash job to get something to *work* rather than taking the time to do it properly. Tech debt accumulates everywhere, all the time. Maybe you have a deadline. Maybe you don't *care* how it works, just that it *does*. Everyone wants to choose the quick and simple fix even if it does mean you shoot yourself in the foot later. In my case, I initially wrote out some campaign design four months ago and I've been working with that basic implementation ever since. 

Up till I started the refactor, my campaign looked like this:

```cpp
struct Campaign
{
    Campaign() : currentDifficulty(1), currentEncounter(0), totalAmmunition(10), totalRepairCapacity(100) {
    }

    u32 currentDifficulty;
    u32 currentEncounter;
    Scenario possibleScenarios[NUM_SCENARIO_OPTIONS];
    Scenario currentScenario;
    u32 totalAmmunition;
    f32 totalRepairCapacity;

    bool moved = false;

    WingmanData* player;
    std::vector<WingmanData*> wingmen;
    WingmanData* assignedWingmen[3];
    ShipInstance* assignedShips[3];
    std::vector<ShipInstance*> ships;
    std::vector<WeaponInfoComponent> availableWeapons;
    std::vector<WeaponInfoComponent> availablePhysWeapons;

    u32 shipCount = 0;
};
```

That was good enough for the months where I was just trying to get some continuity going - being able to swap ships, weapons, wingmen, and whatnot. You might notice a very, very distinct problem with this if you're familiar with C++, though - this is a *struct*, not a class. While, technically, there's no practical difference between the two (aside from default public / private nonsense), *in general* it's good form to use structs for something that's just a grab bag of variables. Classes are for more complicated things that involve actual chunks of code. In my case, here, the campaign *does not have any code associated with the actual structure.*

This is bad for the very simple reason of "my code is scattered literally everywhere around the repository". There were like thirty large functions that all adjusted the campaign in some capacity and they were *everywhere.* Some of it was in the gui code. Some of it was in a generic "game functions" file. Some of it was in carrier utilities. It was scattered *everywhere*, so one of the first things necessary was to throw as much of it as possible in the same spot.

Alongside that I ran into a problem from earlier in development where I was doing far too much with "controller" classes. This is a common anti-pattern that a lot of people fall into where classes are assigned to each other and inherit from each other for no reason. The basic structure of the game for me involves three things - I have a state controller that controls the overall state of the game (whether we're in menus, the game, dead, etc), a game controller that handles going into and out of scenarios and running scenarios, and a gui controller that handles all my menus. In the near future I will *probably* end up adding an audio controller just because audio management is in a similar state to the campaign, but that's it. It's really common to want to try and make one big class that handles *everything,* even if it's a bad idea. Hell, I know I wanted to.

In my case, my original design involved all other controllers and globals being stored directly in the state controller, since it's technically the most important one. I broke *that* part of it over my knee months ago, but there were a few leftover things that I'd kept in the state controller, like global data. The data for ships, weapons, scenarios, etc. was all stored straight in the state controller when, honestly, it didn't need to be. So those got set up as global variables. The upshot of all this is that a line of code that may have initially looked like this:

```cpp
stateController->campaign.assignedWingmen[currentSlot]->assigned = false;
stateController->campaign.assignedShips[currentSlot] = nullptr;
```

...now looks like this:

```cpp
campaign->removeWingman(currentSlot);
```

I cannot tell you what a *relief* this is when dealing with campaign code from *anywhere else* in the codebase.

The last thing I needed to do was split off some scenario management tools that I had into a "Sector" class that the campaign used to keep track of what sector it's currently in. The working plan for the larger game is to have a series of sectors, as I mentioned in the last post, with a set amount of scenarios in each, with each sector having some distinct characteristics and challenges. A lot of scenario and advancement code got moved here, and now I can use the sector class to generate the first sector in the game - the debris field. This will make future sector management much easier.

So after eleven days of work on this, refactoring and tweaking and fixing and shuffling code around, I finally compiled and ran the game... [and not a goddamned thing had changed](https://www.youtube.com/watch?v=aTKRQKhi3cE). The behavior was still exactly the same - it had only changed under the hood for future work. And *that* is the most frustrating part of reworks.

