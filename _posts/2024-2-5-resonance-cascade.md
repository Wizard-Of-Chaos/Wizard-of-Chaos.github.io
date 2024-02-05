### If You Give A Function a Rework...

Debugging and UI updates continue apace -- we've got a hell of a lot more UI elements showing and fixes to various legibility issues, and the whole game has a lot more quality of life features, including dialogue logs to see where you're at in a conversation and much better readouts of what a ship is equipped with. Additionally the killboard makes more sense now, is sorted, and comes with an injury tracker.

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/killboard.png "Honestly, the random names for deaths is fun as hell for me.")

Sectors are also getting minor tweaks and changes (the gas sector was kinda boring and lacked any reason to fly around, so there's more to do there now), as well as multiplayer stuff cooking along. Suffice to say it's all going pretty well, but there's only so much to show. Today I wanted to talk about the butterfly effect as it relates to programming.

See, my friends know that I am somewhat prejudiced against UI designers, mostly as a result of every single app and program I use getting progressively crappier as updates happen and UI updates make everything harder to find. I will say things like "you should only do it once and then just extend on that" or "I could replace UI designers with an untrained gorilla" and even such sentiments as "we should turn whoever produced this new Discord mobile UI into sausage". I personally hate writing this stuff because I can spend a day placing things just so on a menu, and the user will look at it for maybe five seconds. The reward for the work is frustratingly low on the face of it.

But when it means the user spends five seconds looking at it instead of forty, that's a serious gain; it's just not always obvious. Today I wanted to examine how one simple UI change -- the ability to scroll lists, which previously I did not have -- can involve a whole hell of a lot of extra work.

## The Initial Problem

Right, so one of the quality-of-life features I wanted to add to the game was the ability to scroll lists. I can already hear someone wondering why the hell I didn't implement that in the first place, and to you, random imaginary strawman, I say -- *you are entirely correct.* Point is I didn't have it, and I wanted to fix it. So I figured that it'd be pretty easy to do, and I was completely wrong.

The GUI system I wrote is pretty simple. It checks for events from the internal Irrlicht event receiever, then, if an element currently showing has a callback for that event, it fires off the callback. For example, if it detects a GUI event, it will find out which element called that event, and then it will use the callback for, say, clicking a button. Simple, right?

```cpp
	if (activeDialog && event.EventType == EET_GUI_EVENT && callbacks.find(event.GUIEvent.Caller) != callbacks.end()) {
        //fire off the callback
    }
```

This works fine for hover effects, tooltips, button clicking, and display, all of which were accounted for. This was pretty easy. Some events were more finicky than others, requiring specific checks to see if it was a hover or a button click, but overall works fine. So I figured it'd be pretty simple -- except not so much.

See, right there to check, it makes sure that it's of the type EET_GUI_EVENT. Well, keyboard clicks (for up and down arrows) and mouse-wheel scrolling are not EET_GUI_EVENT. They are, in fact, EET_MOUSE_EVENT and EET_KEYBOARD_EVENT. So, yknow, whatever; I'll just add in some extra checks for those too. Then, when it detects a mouse event or a keyboard event, it will fire off those callbacks to the calling element.

```cpp
if (activeDialog && callbacks.find(event.GUIEvent.Caller) != callbacks.end()) {
	if (event.EventType == EET_GUI_EVENT) {
        //callbacks for that
    }
    if (event.EventType == EET_MOUSE_EVENT) {
        //callbacks for this
    }
}
```

Except... wait. This doesn't work either. What if I get a GUI event that also could parse a mouse event? Or something that uses both key and mouse, like a scroll wheel or arrow keys doing the same thing? How do I fix that logic?

## Denial, Anger, and Depression

Clearly, I need some extra detail in how I register these damn callbacks to elements to determine what type of events get tossed their way. But the function for `guiController->setCallback()` is used literally everywhere I have any kind of GUI at all. I don't wanna mess with that thing, it's a complete pain. But what I *can* do is a half-assed rework known as default parameters. Fun fact: literally every time you see a default parameter written in C/C++ code, this is why it exists -- because some god damn developer was lazy and didn't want to rewrite a function properly<sup>(citation needed)</sup>.

The events don't work as a bitmask themselves, so I need an additional enum to flag this stuff.

```cpp
enum GUICONTROLLER_EVENT_FLAG
{
	GUICONTROL_GUI = 1,
	GUICONTROL_MOUSE = 2,
	GUICONTROL_KEY = 4,
	GUICONTROL_JOYSTICK = 8
};
```
...which means my event controller register function looks like this:

```cpp
void setCallback(IGUIElement* elem, GuiCallback callback, onst u32 acceptedEvents = GUICONTROL_GUI);
```

The default parameter here means all my initial calls of this function will continue working, but I can now specify that accepted events can include mouse, key, or joystick events. Great. Now I can finally get around to writing what happens when a scroll wheel goes up or down on this list.

```cpp
if (activeDialog && callbacks.find(event.GUIEvent.Caller) != callbacks.end()) {
    if ((event.EventType == EET_GUI_EVENT && call.flag(GUICONTROL_GUI)) ||
        (event.EventType == EET_KEY_INPUT_EVENT && call.flag(GUICONTROL_KEY)) ||
        (event.EventType == EET_MOUSE_INPUT_EVENT && call.flag(GUICONTROL_MOUSE)) ||
        (event.EventType == EET_JOYSTICK_INPUT_EVENT && call.flag(GUICONTROL_JOYSTICK))
        ) {
        return callbacks[event.GUIEvent.Caller].cb(event);
    }
}
```

Wait, it isn't detecting any of these inputs -- keys and mouse aren't working at all. What the hell is going on? With mounting horror I investigate the logic again and find why it's not calling any of these -- the call to `callbacks.find(event.GUIEvent.Caller)`. There *is* no caller for key and mouse events; those aren't GUI specific. So wait, how the hell do I get the event to the appropriate function?

## Bargaining

I guess what I could do here is search the entire list of callbacks to see what's accepted... so it would just look through all the callbacks to see if any of the buttons require the inputs, right? Since any of them could be using keybinds or hotkeys in the future. So that would look more like an iterator going through the map...

No, that's a terrible idea. There's an ungodly amount of buttons on every screen, that'd be slow as hell, and searching that for *every* event is just stupid. Besides, what if some of those buttons are hidden with their dialog? They have no use for key events. So do we just go through there and make sure the element is *visible,* or... no, because that still includes the search...

The thing to do, really, is determine what screen is currently active and then do keybinds and mouse input for that menu specifically, since more than one button might want to register when I push the up arrow key at a time. We need to check and make sure that the dialog it's on is usable at the moment and that all buttons that might benefit from me pressing 'A' or whatever actually get that input. But how do we get to that...

## Acceptance

Yeah, so what ended up happening was that `setCallback` function I mentioned earlier needed an extra parameter and therefore a lot of crap needed an update.

```cpp
void setCallback(IGUIElement* elem, GuiCallback callback, MENU_TYPE which, const u32 acceptedEvents = GUICONTROL_GUI);
```

When a callback is registered, it now includes the element for said callback, what menu it came from (with a value for, like, in-game UI or something that isn't menu based) and what events it's willing to accept. This necessitated going through *every* call to this function on *every* menu and adding in this parameter to fit from the menu. Boring, boring tedium. Finally, at the end of all this, I was able to capture key events and mouse events from the list being displayed and then move the buttons on the list up or down respectively. Days of work went into this one simple feature.

It's not all drudgery, though, as you may have noticed this new structure allows for keybinds and other events on every menu. This allows for controller support, since controllers don't really have a "click" option, and allows for rapid menu interfaces through keybinds, which wasn't there before. So in the future you should be able to navigate to the fab menu by just hitting F or something instead of moving your mouse at all. So that'll be nice to have.

The moral here, if there is any, is to be nice to your UI designers. Tiny little updates can have huge cascade effects sometimes. Although you should still make fun of them for making stuff look worse.

## What's Next?

More UI updates, ironically, and more QoL features on menus, and finishing off dialogue writing and then just game balance tweaks. Maybe some more ship designs and weapons as well. Multiplayer scenarios too. Things are lookin' good.