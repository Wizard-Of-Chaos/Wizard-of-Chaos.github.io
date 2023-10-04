### Rework on Weapon Firing Systems

As autumn falls and we see the first few hints of winter beginning to entomb us where I live (well, it's colder, anyway) I find myself once again in the throes of a [strange mood](https://dwarffortresswiki.org/index.php/Strange_mood). I don't know what it is; this happened last year too (during the months of September to November 2022 I was averaging six commits daily), but it bodes well for the game being released by the end of the year! Primary development is pretty much done; I just need to tweak the last few sectors and come up with a halfway-decent final bossfight... and then I get into the hell of bugfixing, more asset creation (despite all my efforts, I'm still only about halfway done there), and dialogue writing. Release changes, basically.

There are a few other changes, visually and design wise. Aliens now have their own modular stations! They look pretty cool, and you'll start running into them in the asteroid sector with "Destroy" missions. There's some new UI for them too.

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/alien_station.png "This is probably my favorite screenshot so far.")

There's more ships, and the ones that existed prior look notably better. This is one of the designs; a fast, single-gun fighter with resistance to impact damage.

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/gougar.png "In-game it's the Cougar, and I always mutter to myself 'oh fug, a gougar' every time I see it.")

ARTHUR, our resident drone-pilot-thing, has also received a facelift. I'm very proud of how this came out.

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/arthurianrework.png "Friend! Hello, friend!")

The mess hall isn't entirely complete yet, but my artist is very hard at work and has done some excellent sprite work for the wingmen. The background is still being worked on, so in-game it looks a little funky at this very moment, but it's a tremendous step up. Tauran's never looked better.

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/tauran_new.png "Pay no attention to the poorly drawn art from a programmer in the background!")

And, finally, I've added some new weapons, namely lasers with a hitscan mechanic. That's the one I want to talk about today.

## Reworks and When Not to Do Them

[Several posts back](https://wizard-of-chaos.github.io/2022/11/09/overthinking.html) I made a comment about how being aware of bugs in your code is similar to walking on an injured leg.

> (...) So I marked it down, couldn’t immediately see a problem with it, and simply just stopped trying to do that ever since I implemented the feature initially. It’s like trying to keep your weight off an injured leg. You know it’s injured, but you’ve grown accustomed to limping a bit. Then, to stretch the metaphor, someone else takes over your body for a bit, puts some weight on the leg and snaps the damn thing in half.

Code that needs to be reworked is similar, but the difference there is that usually, in the present ugly state, it at least *works*, and can be safely ignored -- *for now*. Your car might be ugly and have nearly-bald tires, but it *will* get you to work on time. However, if you want to take it up a mountain into a snowy ski resort, you're gonna need to make some upgrades. My projectile code was the ugly car, and it needed to be replaced so I could go to more places and do wilder things with it.

There's another thing to be said for delaying reworks as long as you can, aside from the classic notion of "if it ain't broke, don't fix it". My projectile code was *old*, and the original version was actually one of the first things I ever wrote for the damn project, nearly two years ago now, and frankly I was a lot shittier at programming back then -- or, well, shittier in the specific field of writing projectile code and game design and all the other garbage I've had to pick up for writing this game. Delaying the rework as long as I have means that a more experienced programmer gets to make the final decision about how it functions. And that's what needed to happen.

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/flawed.png "In retrospect, the best github commit lines I've ever made.")

## What Are Guns?

Programming, much of the time, is like explaining a concept to a very, very stupid person -- namely, you have to break down whatever you're talking about into its most fundamental pieces. Let's break down 'gun' into its most basic form. What, exactly, *is* a gun, and what does it *do*? What it *is* varies, but what it *does* is consistent.

1. Shoot. It needs to *fire* somehow. This varies per gun, but generally you pull the trigger and something mean comes out the other end.
2. Hit. When whatever came out the end of the gun hits something, it needs to do something. Usually, it's damage, but it can be things like "explode", "slow", or others.
3. Update. For example, if I'm out of ammo, every tick the gun needs to add to its 'reloading' counter to determine when it's done reloading. This applies to other functions as well -- what's the gun doing every tick?

Great! We have established the fundamental properties of 'gun'. A gun is a collection of variable Shoot/Hit/Update behaviors, and data assisting those behaviors (such as ammunition counters). Someone ship that off to Congress for aid in Second Amendment arguments. Now let's look at the original way I handled this.

## Original "Design"

The original "design" for this work was less a 'design' and more a panicked amalgam while I was figuring out just what the hell I was doing, but you can still see these fundamental parts... if scattered. For 'Shoot', this was handled exclusively in the weapon firing system.

# Shoot

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/oldwep_0.png "Eugh.")

Right away I have problems with this. There are only four possible methods to determine whether a gun can "fire", and those are ammo-exclusive, power-exclusive, power AND ammo, or neither. Nothing else is taken into consideration, and even those three devolve into jangled messes when you actually go to look at the functions they're calling. Specifically I take issue with this line:

```cpp
if (wep->type != WEP_HEAVY_RAILGUN) createProjectileEntity(wep->spawnPosition, wep->firingDirection, id, true);
```

So if it's *not* a railgun, it creates a projectile outright, but if it *is*, it just eats the ammo and does nothing. This line is in all three of those functions. This implies, further, that if I wanted to add more distinct types of weapons I would once again need to repeat myself *three times* before they were ready. And, shit, what if they don't *have* projectiles? Not every gun uses them...

This brings the question up of where *does* the railgun fire? Well, of course, it has its own little specialized part of the function, because why wouldn't it? Plainly, this will not do, and we haven't even gotten to the actual creation of the goddamn projectile. Once the function has decided that yes, the gun will fire, it goes into the `createProjectileEntity` function.

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/oldwep_1.png "Eugh!!")

I got problems with this! I got lots of problems with this, but most of them boil down to the *same* problem -- this is a chain of if/elses where if I want to add a new gun I have to add *individualized exceptions for every gun*. This is, put simply, bad design; it's not flexible and it's not very extensible.

# Hit

Alright, let's look at where the projectile actually hits. That's handled as part of the collision code.

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/oldwep_2.png "There's a bomb in here. Can you spot it?")

When it hits, it calls the `handleProjectileImpact` function, then applies the damage from the projectile and removes it. Seems reasonable-- wait. What's in that `handleProjectileImpact` function? What's in your mouth? SPIT IT OUT! DROP IT!

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/oldwep_3.png "Bomb has been planted")

Yup, it's *more of the same!* Once again if I want a gun to do anything different on impact, I have to go *here* and specifically define it. It's the exact same problem as earlier! And *all* guns deal damage, whether I like it or not -- maybe I just want a gun to *slow* instead of deal damage. It's the same crappy pattern.

# Update

I'm sure you can guess by now what the problem was with Update as a behavior, because yes, it's more of the same. In this case, bits and pieces were handled in the actual `weaponFiringSystem` function, but again it went to a function with a switch case in it. Honestly, it looks almost identical to Hit as far as bad design goes.

Now, it's important to note that all of this *worked*, despite being hideous and inflexible, so there was no reason to rework it until I needed to start making other weapon behaviors. So how do we fix this ugly mess?

## Back to Basics

We go back to basics of Shoot, Hit, and Update. Now, the fact that all these guns have individualized behavior isn't bad *in itself* -- but they need to be split up better, and they need to reuse code between them (for all intents and purposes, the railgun is effectively just a regular gun with a delay on it). If we tried object-oriented behavior, we wind up with a similar problem where we have to define lists of classes for each category of weapon and inheritance and other garbage. This is a job for *functional programming*, and more specifically lambdas and function captures.

If you don't know what functional programming is as a design pattern, I can summarize it very quickly: imagine if functions did *everything* and there were no classes or any of that crap. Lambda functions are effectively an answer to the question, "What if functions were variables?" For a lambda expression, you define a function and then you can box it up and hurl it around like you would a variable. In C++, this is neatly captured under `std::function`.

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/typedefs.png "I love std::function. It is my favorite piece of the STL.")

I define my *callbacks*, which are just functions that would get called *back* when something happens, and then go shove them into my weapon component.

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/wepcallbacks.png "Oooh, clean!")

Guns only fire in one way, so that's just listed as a variable, but they might have multiple things they want to do on hit and on update, so we keep lists of those. For example, the list of things that the railgun might do is "damage" and "knockback" functions.

When we shoot the gun, we call the `fire` function that was loaded from file, and it handles creating a projectile or other crap, and there are several subfunctions there for creating the bullet stuff for each projectile so that all the fire functions can reuse code where possible -- but if I edit one, I don't have to edit the rest. On hit, it calls all the `hitEffects`, and every tick, it calls all the `updates`. That's all there is to it! All of these features are compartmentalized and the flexibility of it allows me to do things I couldn't before.

## Like What?

Hitscan weapons, for one -- *all* guns called `createProjectileEntity`, and that's no longer the case, because it's not so rigidly defined anymore. The weapon firing function for a laser weapon is just fundamentally not the same as the other projectile-creating functions, and necessitated different behavior (which it now has). This also opens the door for other types of heavy weapon and physical weapon where the 'fire' might have absolutely nothing to do with projectiles. There are laser guns, now. I'm very happy.

If you take away one thing from reading about this process, make it this: don't fear refactors, but don't rush into them either. I had no idea how to do this two years ago, but I do now, and it worked satisfactorily in the meantime. It's okay to delay this sort of thing until you have to make decisions, because when you delay, you'll be handing it off to a more experienced programmer -- yourself, in however-long.

## What's Next

More modelling, texturing, asset creation, yadda yadda yadda I'm sure it's obvious by now. Work continues, though, and I'm very pleased with the current state. Onward we go.