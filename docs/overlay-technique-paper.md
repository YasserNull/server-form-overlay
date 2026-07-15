# Creating Functional Buttons on the Gameplay Screen in Minecraft Bedrock Edition and Discovering Bugs in the Algorithm

**Developer:** YasserNull

**Project Release Date:** August 12, 2023

This document documents the **Server Form Overlay** project for the community, and also includes a report on the bugs discovered during its development, prepared for submission to the [Mojira Bug Tracker](https://report.bugs.mojang.com/servicedesk/customer/portals).

## Project Overview

On August 12, 2023, I launched a project called **Server Form Overlay**. This project was simply about adding buttons to the `hud_screen` gameplay screen with the ability to execute code (API Scripting) in Minecraft Bedrock Edition.

Note that this is considered impossible to this day, since there is no official way to do it.

> **Fun fact:** This exists in the Chinese version of Minecraft, but it is not available in Minecraft Bedrock Edition.

Despite that, I managed to be the first person to do this, and I shared it and made it open source so developers could benefit from it.

How did I manage to do this?

![Screenshot showing buttons on the gameplay screen](https://github.com/YasserNull/server-form-overlay/raw/main/docs/images/screenshot2.jpg)
![Screenshot showing buttons on the gameplay screen](https://github.com/YasserNull/server-form-overlay/raw/main/docs/images/screenshot3.jpg)

## How the Idea Started: Ty-el's Project

The whole thing started when a developer named **Ty-el's** developed an addon called **Settings Overlay Ui Pack**. This addon makes the settings screen appear over the gameplay screen. As the name suggests, it makes the settings screen an overlay, meaning it becomes a layer on top of the gameplay screen, and you can, for example, change the settings while the gameplay screen is running.

The settings screen becomes a layer over the gameplay screen, and note that you can't change settings from within the gameplay screen itself, so they remain two separate screens that don't receive each other's events. However, you can customize the settings screen — for example, to add a perspective-change button. This is exactly what Ty-el's did in this project, where he:

- Created a button to close the settings screen (removing its layer from over the gameplay screen).
- Created a button to show and hide the settings screen so it wouldn't cover the gameplay screen.
- Created a perspective-change button, which had been requested by the community for many years.

The perspective-change button he added is essentially a shortcut for the drop-down perspective button, allowing settings to be changed via a button that appears on the gameplay screen (even though this button literally exists on the settings screen, not the gameplay screen).

## Applying the Idea to the Server Form Screen

An idea occurred to me: why not apply the same idea to another screen called **server form**? This screen belongs to the menus created via the Scripting API through the `@minecraft/server-ui` library, and you can add buttons to it with the ability to make them execute code. The server form screen can also be customized to completely change its appearance — for example, removing the menu shape entirely and creating a fully custom shape.

This would let us create buttons on the gameplay screen with the ability to execute instructions.

I started analyzing Ty-el's project code and understood how he made the settings screen an overlay, so I started applying the idea, and it actually succeeded. Afterward, I contacted Ty-el's and told him that I had taken his own code to reuse it in my project, so I sent him a picture of the project, and he told me I could add one of the properties to reduce the problems.

## Technical Details

### `is_showing_menu`

It turned out that the property that makes a screen an overlay is `is_showing_menu`, a property that accepts a boolean value and determines the screen's class; when enabled, it makes the screen an overlay. That's all there is to it — just a single line.

For example, take the start screen and the settings screen: when you're on the main menu and press the settings button, the settings screen opens and the start screen disappears because you've moved to another screen. But when you add this property to the settings screen and disable it, the settings screen becomes an overlay screen that appears on top of the current screen.

![Image showing how the server form screen became an overlay over the gameplay screen](https://github.com/YasserNull/server-form-overlay/raw/main/docs/images/screenshot1.jpg)

### Solving Control and Performance Issues

Afterward, several problems appeared, including that controls didn't work and the player couldn't move. But the solution was simple, through the `absorbs_input` property, which allows touch events to pass through the screen. For example, if you tap an empty spot on the overlay screen, the touch event won't reach the gameplay screen because it stops at the overlay screen; this property solves that problem and lets touch events pass through the screen.

After that, the following properties were added:

- **`close_on_hurt`** (disabled): to prevent the screen from closing when the player takes damage.
- **`render_only_when_topmost`**: determines whether the rendering engine will draw the screen only when it is the topmost layer among the screens.
- **`low_frequency_rendering`** (enabled): tells the rendering engine to draw the screen at a low frame rate. This improves performance, since there's no benefit to drawing the overlay screen at the same frame rate as the gameplay screen, reducing the load on the rendering engine.
- **`render_game_behind`** (enabled): determines whether the rendering engine keeps drawing the game (the 3D world) behind the screen; we don't want the game to look frozen when the overlay screen is open.
- **`$screen_animations`** and **`$background_animations`** (set to an empty array): to remove the screen's animations and eliminate the delay when opening and closing it.

A problem also appeared where most of the gameplay screen's components disappeared, such as the touch circle (cursor) and the hotbar. The fix for this was the **`force_render_below`** property, which Ty-el's told me would reduce the problems if added; it forces the rendering engine to draw the game's components when a screen opens on top of them (as happens with the pause screen and the chest screen).

After all this, it was observed that the overlay screen's buttons didn't work, and anything you pressed would "pass through" the screen to reach the screen behind it. That's why I added the **`prevent_touch_input`** property and enabled it — it blocks touch input from being received. Why enable it? This may seem strange at first, but adding it to the buttons that touches were passing through solves the problem; afterward, the touch event stops at the overlay screen and doesn't reach what's behind it.

## Result and Release

With that, all the problems were solved and the project became complete and ready for use, so I released it and made it open source for addon developers to benefit from. However, it didn't gain much traction because of these issues, and because it only works on touchscreens — mouse, keyboard, and controller aren't compatible with it, since the added buttons on the screen can't be pressed using them.

## Testing the Idea on Other Screens

### The `npc_interact` Screen

I tested the same technique on other screens such as `npc_interact`, and managed to do the same thing: add buttons to the screen with the ability to make them execute commands. However, it has issues, such as not being able to switch the hotbar slot.

![Image showing the npc screen overlaid over the gameplay screen](https://github.com/YasserNull/server-form-overlay/raw/main/docs/images/screenshot11.jpg)

### The Sleep Screen (`bed`)

I also tested it on the `bed` sleep screen, but the game hides the control buttons in the new oreui-based interface for this screen. However, going back to the update before oreui, to use the old bed screen, the control buttons appear and it becomes possible to hit and build while sleeping.

![Image showing the control buttons while I'm asleep](https://github.com/YasserNull/server-form-overlay/raw/main/docs/images/screenshot9.jpg)

However, movement isn't possible, since the player stays fixed in place. As shown in the following images, the bow can be used to shoot arrows while asleep, along with throwing snowballs and using emotes.

![Animated image showing emotes being used while sleeping](https://github.com/YasserNull/server-form-overlay/raw/main/docs/images/emotes.gif)
![Animated image from another player's perspective showing emotes being used while sleeping](https://github.com/YasserNull/server-form-overlay/raw/main/docs/images/emotes2.gif)
![Animated image from the player's perspective showing snowballs being thrown while sleeping](https://github.com/YasserNull/server-form-overlay/raw/main/docs/images/snowball.gif)
![Animated image from another player's perspective showing snowballs being thrown while sleeping](https://github.com/YasserNull/server-form-overlay/raw/main/docs/images/snowball2.gif)

I also discovered another bug: when I tap the screen (right click), it touches the bed as though attempting to sleep in it, and the message `this bed is occupied` appears in chat, even though the player is already asleep in that very bed.

![Image showing the error message that appears in chat](https://github.com/YasserNull/server-form-overlay/raw/main/docs/images/this_bed_is_occupied.gif)

### The Crafting Screen (`crafting`)

The idea was also tested on the crafting screen, which is the inventory screen, and it became possible to move while using the inventory — but this is nearly useless.

### The Chest Screen (`chest`) — The Most Dangerous Results

When testing it on the chest screen, the most dangerous results appeared.

A hack addon was created that works only through a Resource Pack, making it possible to use a chest while moving, building, and hitting at the same time. This amounts to cheating, especially if used on servers, since it deceives other players into believing the player has only opened a chest, when in reality they're moving, building, and attacking.

![Animated image showing how I tricked a player on a CubeCraft server by opening a chest and taking them down while they were advancing on me](https://github.com/YasserNull/server-form-overlay/raw/main/docs/images/cubecraft.gif)

A bug was also discovered in the game's algorithms: it doesn't check whether the player already has a chest open, which allows several chests to be opened at the same time.

Stranger still, there's another bug: chests don't close after the chest screen is closed, and even after moving away from them they stay open — meaning the player no longer has access to them and is no longer considered to have them open, but an error occurs and the chests stay visually open. This affects the game, since the chests remain visibly appearing open all the time, and a chest like a shulker box becomes two blocks instead of one.

> **Note:** When more than one chest is opened and the chest screen is then closed, only the last chest that was opened actually closes. The same happens if more than one chest is opened and the player then moves away from them — only the last one opened closes, and the rest stay open.

![Animated image showing several chests open at the same time](https://github.com/YasserNull/server-form-overlay/raw/main/docs/images/chests.gif)

When tested on servers, it doesn't work the same way — it isn't possible to open more than one chest, since attempting to open another chest immediately closes the previous one and opens the new one. This looks strange to other players, since they see the player opening chests in under a second.

Another error appears when opening the chest, with two messages showing in chat:

```
Something went wrong!
(errcode 1)
```

![Screenshot showing the error message in chat](https://github.com/YasserNull/server-form-overlay/raw/main/docs/images/screenshot4.jpg)

There's yet another bug: when interacting with the screen (right click), the bed gets touched as though the player pressed it to sleep, so the message "this bed is occupied" appears — meaning someone is already sleeping in it, even though it's the very same player who's trying to sleep in it while already asleep there. This is a very strange bug.

## Summary of Discovered Bugs (For Submission to Mojira)

1. **Sleep screen:** Using the old bed interface (pre-oreui) together with the overlay screen technique, the control buttons stay visible while sleeping, allowing arrows and snowballs to be thrown and emotes to be used while the player is asleep, while they remain fixed in place.
2. **Chest screen — potential cheating:** Through a resource pack alone, the chest screen can be kept open while moving, building, and hitting, which could be used to deceive other players on servers about the player's true state.
3. **No check for an already-open chest:** The game doesn't check whether the player already has a chest open, allowing several chests to be opened at once (in single-player mode).
4. **Chests staying visually open:** After the chest screen is closed or the player moves away, chests remain appearing open (a shulker box becomes two blocks instead of one); only the last chest that was opened actually closes.
5. **Chat error message:** The message `Something went wrong! (errcode 1)` appears when opening a chest.
6. **Chat message when touching the bed the player is sleeping in:** The message `this bed is occupied` appears when the screen tap touches the bed as a sleep attempt, showing that the bed is occupied — by the very same player already asleep in it.

## Links and Sources

- **Project Repository (GitHub):** https://github.com/YasserNull/server-form-overlay
- **Technical Source for Screen Properties:** https://mc.163.com/mcstudio/mc-dev/MCDocs/2-ModSDK%E6%A8%A1%E7%BB%84%E5%BC%80%E5%8F%91/60-UI/4-UI%E8%AF%B4%E6%98%8E%E6%96%87%E6%A1%A3.html
- **Mojira Bug Tracker (for submitting the report):** https://report.bugs.mojang.com/servicedesk/customer/portals
