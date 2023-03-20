### Heads-Up Display Design

The trend of fixing up the aesthetic for future screenshotting continues. The past week or so has been HUD design. This one's extremely important, both aesthetically and informationally, because the way the HUD is designed is going to influence how the player gets their information at all times and whether or not they enjoy doing so (by which I mean, don't get annoyed that it looks godawful). Unfortunately, a HUD is also one of those things that can get screwed up extremely easily by a tiny piece of bad design. I thought it might be a good idea to go over some general design principles with regards to how the player should be looking at their screen.

## The BAEDS Development Number-One Rule Of UI Design

UI is not your game. I might get arguments for this, but your user interface is *not your game--* your user interface is what allows people to actually *interact* with your game. All your systems and information have to get filtered through the UI, like trying to shine a light through a window. If you want as much light (or, that is, as much *gameplay*) as possible to get through said window, it needs to be as transparent as possible and it needs to not distort anything. To that end, the number-one rule of UI design in my mind is that your UI's job is to get out of your way as quickly as possible so that the player can continue playing the game.

To that end, the HUD for a game should display all the information it needs to and it should do it from a sane position (i.e., relay it all as clearly as possible). To that end, when designing the HUD and its aesthetic, the first thing I did was make a list of every piece of information that needs to be displayed on the screen for the player.

*Disclaimer: That's not actually the first thing I did. The* first *thing I did was to go get screenshots of every other space game I own to determine how* they *do it. There's nothing wrong with ripping off your peers once in awhile. A lot of them fell into a common theme for reasons I'll outline now.*

1. Health and shields. The number-one chunk of information for the player is whether or not they're about to turn into space debris. To that end, it should be clearly displayed in a fairly central position. 
2. Targeting data. The crosshairs, selection, enemy health, distance, all that jazz. This will pretty much always be at or near the center of the screen.
3. Ammunition. This is not a time critical thing, so we can afford to shove this off to the side a bit. The player needs to know whether or not they're about to run out of ammo.
4. Energy. This is ammo for energy weapons and your boost capacity. Important, but not too important, so it should be smaller and near the health bar since it's another resource.
5. Speed. How fast are you going, and is it fast enough to kill yourself? How quick can you slow down? Again, important but not *too* important, so same deal as energy.
6. Wingman information. This is orders and their current health pool to see whether or not they're in trouble. The player won't look at this often, so this gets shoved to the side.
7. Pop-ups. This is communications from your wingmen, mission-critical data and hazard markers. These are all once-in-a-while things and can be shoved to the least important section of the screen.

Armed with this data, I grabbed my tablet and did a quick sketch on the general whereabouts of all this information.

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/uisketch.png "It's blue, because everyone knows that space is blue.")

Health, energy, and speed are all in the bottom chunk where the player can see them quickly. Ammunition information is off to the right-center of the screen with a fill bar at closest to the center so the player can see their ammunition for each weapon without looking too much. Comm panel off to the bottom left as a sometimes information, and all forms of popups scroll down from the top since most people don't look at the top of their screen all that often. The comm panel there flips between wingmen health bars and the orders menu.

Now that I had a general idea of where all my meters should go on the screen, I needed to think about how I wanted it to look aesthetically. The problem here is my own damnably bad art skills, so I can't really do an in-cockpit view in the style of something like Wing Commander (nor should I, the damn game is third-person). One thing I was pretty sure of right from the start was that most of the UI elements needed to be transparent where they could be so the player's screen didn't get clogged. Remember that at all times we're trying to get out of the player's way -- we don't want to clog the player's screen with additional crap that we don't have to.

The aesthetics came out of a real-world source (and some soundboarding from my esteemed partner on this project):

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/aircrafthud.png "Once again, the US military is proven to be the best game designer.")

The military is well aware that their pilots vision needs to be as free as possible, but they also have a ton of information they need to process. To that end, actual aircraft HUDs look like this -- lots of sharp lines, plain text, and all of it laid out around the center of the screen. They don't have health bars (well, as far as *I* know), but they do have a ton of other crap.

This actually works out pretty nicely for me as well. Straight, glowing lines are *easy.* And they're easy to lay out in a way that looks just a bit futuristic and practical at the same time.

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/freshhud.png "No, wait, it's actually just me.")

Bam. Information laid out where the player can easily get to it. It's amazing how many people screw this one up. There's still a bit more work to be done aesthetic wise, but I'm pretty happy with how it looks as is, with all the data laid out according to how directly relevant it is. I'm honestly a bit astonished at how many devs screw this up -- not aesthetic wise, just positioning wise. If you sit down and think about *where things should go* on your screen for even a minute, this sort of thing gets insanely obvious.

## What's next?

More aesthetic work. More behind the scenes bureaucracy. More assets, ships, designs... eurgh. I did get docking behavior down pat, and I'm working on the supply depot boss fight now, and also I fixed the AI's aim to be a little less silly (the Arachnid kept missing because the shots weren't aimed like the players -- they were all in a straight line and it was literally impossible for it to get off a full blast). Plenty of stuff. Never the slightest peace. I'll update next probably with some notes on large-scale battle design, the gas giant sector, player choice and some other things.