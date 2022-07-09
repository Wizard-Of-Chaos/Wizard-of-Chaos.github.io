## Polymorphism and AI Updates

I'm starting in on wingmen, now, and I realized pretty much immediately that it would require some AI updates. The quick solution for adding wingmen would be simply throwing in some friendly ships and calling it a day, but that's pretty boring. Realistically the allied AI should treat the player as their "commander", staying close, providing cover, and forming up on a wing. The AI presently did not have the capacity to accomplish this, but that was a fixable thing.

I did notice that my AI code had some problems and was certainly not very extensible. A friend of mine had recently warned me about this, actually, after looking at a different piece of code, but on AI it became readily apparent that I was falling into an anti-pattern, using a switch case and enums where polymorphism should be doing its thing. Here's what the AI code initially looked like:

```cpp
void AIUpdateSystem(flecs::iter it, 
    AIComponent* aic, IrrlichtComponent* irrc, BulletRigidBodyComponent* rbcs, ShipComponent* shipc, SensorComponent* sensc, HealthComponent* hpc)
{
    for (auto i : it) {
        auto ai = &aic[i];
        auto ship = &shipc[i];
        auto rbc = &rbcs[i];
        auto irr = &irrc[i];
        auto sensors = &sensc[i];
        auto hp = &hpc[i];

        switch (ai->AIType) {
        case AI_TYPE_DEFAULT:
            defaultAIUpdateSystem(ai, irr, rbc, ship, sensors, hp, it.delta_time());
            break;
        default:
            break;
        }
    }
```
This then feeds into the "default AI update system", which looked like this:

```cpp
void defaultAIStateCheck(AIComponent* aiComp, SensorComponent* sensors, HealthComponent* hp)
{
    if (sensors->closestHostileContact == INVALID_ENTITY) {
        aiComp->state = AI_STATE_IDLE;
        return;
    }

    else if (hp->health <= (hp->maxHealth * aiComp->damageTolerance)) {
        //there's a hostile, but I'm low on health!
        aiComp->state = AI_STATE_FLEE; //aaaaieeeee!
        return;
    }

    //there's a hostile and I can take him!
    aiComp->state = AI_STATE_PURSUIT;
    //whoop its ass!
}

void defaultAIUpdateSystem(
    AIComponent* ai, IrrlichtComponent* irr, BulletRigidBodyComponent* rbc, ShipComponent* ship, SensorComponent* sensors, HealthComponent* hp,
    f32 dt)
{
    ai->timeSinceLastStateCheck += dt;
    if (ai->timeSinceLastStateCheck >= ai->reactionSpeed) {
        defaultAIStateCheck(ai, sensors, hp);
        ai->timeSinceLastStateCheck = 0;
    }
    
    switch (ai->state) {
    case AI_STATE_IDLE:
        defaultIdleBehavior(ship, rbc, dt);
        break;
    case AI_STATE_FLEE:
        defaultFleeBehavior(irr, rbc, ship, sensors->closestHostileContact, dt);
        break;
    case AI_STATE_PURSUIT:
        defaultPursuitBehavior(sensors, rbc, ship, irr, sensors->closestHostileContact, dt);
        break;
    default:
        break;
    }
}
```

You see where this is going, unfortunately. I'd written myself into a hole. If I wanted to add in a new type of AI, like, say, AI for an ace pilot or squadron-based AI, I would first have to create a new "type" of AI then define *every single switch statement all over again*. This is way too much rewriting and would have to be done for even incremental changes. The friend in question wrote an excellent blog of his own [which you can see right here.](https://blog.thelonepole.com/2015/5/polymorphism-antipatterns-in-php) He gives a better overview of anti-patterns than I can, so I won't even bother.

Point is, this is a situation where the AI should be defining its own behaviors, not a switch statement. So I need, effectively, to create extensions to the behavior and create some virtual functions that can be overridden. The result is that I ended up with a new class that was effectively just function pointers:
```cpp
class AIType
{
public:
	virtual void stateCheck(AIComponent* aiComp, SensorComponent* sensors, HealthComponent* hp) = 0;
	virtual bool distanceToWingCheck(AIComponent* aiComp, BulletRigidBodyComponent* rbc) = 0;
	virtual bool combatDistanceToWingCheck(AIComponent* aiComp, BulletRigidBodyComponent* rbc) = 0;
	virtual void idle(ShipComponent* ship, BulletRigidBodyComponent* rbc) = 0;
	virtual void flee(ShipComponent* ship, BulletRigidBodyComponent* rbc, IrrlichtComponent* irr, flecs::entity fleeTarget) = 0;
	virtual void pursue(ShipComponent* ship, BulletRigidBodyComponent* rbc, IrrlichtComponent* irr, SensorComponent* sensors, flecs::entity pursuitTarget, f32 dt) = 0;
	virtual void pursueOnWing(AIComponent* aiComp, ShipComponent* ship, BulletRigidBodyComponent* rbc, IrrlichtComponent* irr, SensorComponent* sensors, flecs::entity pursuitTarget, f32 dt) = 0;
	virtual void formOnWing(AIComponent* aiComp, ShipComponent* ship, BulletRigidBodyComponent* rbc, f32 dt) = 0;
};
```
A pointer to this base class is included in the AI component, and suddenly I can use inheritance. There are two classes that presently inherit, the default ship AI and the ace ship AI, with the ace ship inheriting from default ship and overriding only a few things. Much less typing for me, which is a good thing! I still have the AI states - flee, pursue, idle, and so on - but now they just automatically call the appropriate behavior based on what type of AI it has.

With that out of the way, the question became what exactly do I want aces to be able to do? Obviously wingmen would treat the player as an ace, but what does that entail? For now I've gone with a few simple rules - when the AI is idle, it forms up around the ace pretty closely, and when it's in combat that distance extends - but if they stray too far from their ace, they'll form up again before renewing their assault. The ace itself doesn't run at low health (the other AI will start fleeing at about 40% health), and eventually I'll override their pursuit behavior to maek them better-than-average pilots. For now it stays simple.

After I'd gotten these behaviors down and set it up so that scenarios now spawn an ace pilot along with the rest of the squad, I gave it a test and went to pit myself versus my newly updated AI.

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/acedead.png "My end-scenario taunt stung more than usual this time.")

They *kicked my ass.* It wasn't even *close-* I got zero kills out of the engagement. A few simple rules added onto the AI and suddenly it's *much* more terrifying. I even got to see the new behavior in action, where after I desperately accelerated away and swung around to try and pick one or two off, I saw the AI ships fly towards their ace, form up as one, and then all of them shot towards me and opened fire. I'm not gonna lie - that was pretty goddamned cool. Even small bits of coordination make it much more of a challenge, and it was super cool to get my ass handed to me by something I made.

I have yet to implement player-friendly wings - that will come soon, with the ability to choose your wing's ships and weapons, which involves *yet more* UI work - but now that the basic principles are down it should be fairly easy. As I said, the player will be designated as the friendly "ace". Eventually the ace behavior will be updated even more to make it more of a challenge, but so far I'm really happy with how the AI is going.