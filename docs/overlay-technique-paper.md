# Creating Functional Buttons in the Minecraft Bedrock HUD Screen and Discovering Algorithm Bugs

**Developer:** YasserNull

**Project Launch Date:** August 12, 2023

This document documents the **Server Form Overlay** project for the community, and also includes a bug report discovered during its development, prepared for submission to the [Mojira Bug Tracker](https://report.bugs.mojang.com/servicedesk/customer/portals).

## Project Overview

On August 12, 2023, I launched a project called **Server Form Overlay**. This project was simply about adding buttons to the play screen (`hud_screen`) with the ability to execute code (API Scripting) in Minecraft Bedrock.

For the record, this is considered impossible to this day because there is no official way to do it.

> **Fun fact:** This feature exists in the Chinese version of Minecraft, but it is not available in Minecraft Bedrock.

However, I managed to be the first person to do this, and I shared it and made it open-source for developers to benefit from.

How did I manage to do this?

![Screenshot showing buttons on the play screen](https://github.com/YasserNull/server-form-overlay/raw/main/docs/images/screenshot2.jpg)
![Screenshot showing buttons on the play screen](https://github.com/YasserNull/server-form-overlay/raw/main/docs/images/screenshot3.jpg)

## How the Idea Started: Ty-el's Project

The whole thing started when a developer named **Ty-el's** developed an add-on called **Settings Overlay Ui Pack**. This add-on makes the settings appear over the play screen. As the name suggests, it makes the settings screen an overlay, meaning it becomes a layer on top of the play screen, allowing you, for example, to change settings while the play screen is active.

The settings screen acts as a layer above the play screen. Note that you cannot change settings directly on the play screen, which is why they are two separate screens that do not receive each other's events. However, you can customize the settings screen, for example, to add a change perspective button. This is exactly what Ty-el's did in this project, where he:

- Created a button to close the settings screen (to remove its layer from on top of the play screen).
- Created a button to show and hide the settings screen so it doesn't cover the play screen.
- Created a change perspective button, which had been requested by the community for many years.

The change perspective button he added is a shortcut for the perspective drop-down button, allowing the user to change settings via a button that appears on the play screen (even though this button is literally on the settings screen, not the play screen).

## Applying the Idea to the Server Form Screen

I had an idea: why not apply the same concept to another screen called the **server form**? This screen belongs to the menus created via the Scripting API through the `@minecraft/server-ui` library. You can add buttons to it and make them execute code, and the server form screen can be customized to change its appearance entirely, such as removing the menu layout and creating a completely custom design.

This would enable us to create buttons on the play screen with the ability to execute instructions.

I started analyzing the code of Ty-el's project and understood how he made the settings screen an overlay. I then started applying the idea and actually succeeded. Afterwards, I contacted Ty-el's and told him that I had taken his specific code to reuse in my project. I sent him a picture of the project, and he told me I could add a certain property to reduce issues.

## Technical Details

### `is_showing_menu`

It turned out that the property that makes the screen an overlay is `is_showing_menu`. It is a property that accepts a boolean value and determines the screen type; when enabled, it makes the screen an overlay. That's all there is to it, just a single line.

For example, the start screen and the settings screen: when you are on the main menu and press the settings button, the settings screen opens and the start screen disappears because you transitioned to another screen. But when this property is added to the settings screen and disabled, the settings screen becomes an overlay that appears on top of the current screen.

!(https://github.com/YasserNull/server-form-overlay/raw/main/docs/images/screenshot1.jpg)

### Solving Control and Performance Issues

After that, several issues appeared, including the controls not working and the inability to move the camera. But the solution was simple via the `absorbs_input` property, which allows touch events to pass through the screen. For example, if you tap an empty spot on the overlay screen, the touch event won't reach the play screen because it stops at the overlay screen. This property solves this issue and allows touch events to pass through.

Subsequently, the following properties were added:

- **`close_on_hurt`** (disabled): To prevent the screen from closing when the player takes damage.
- **`render_only_when_topmost`**: Determines whether the GPU will render the screen only if it's the topmost layer among screens.
- **`low_frequency_rendering`** (enabled): Tells the GPU to render the screen at a lower framerate. This improves performance since there is no point in rendering the overlay screen at the same framerate as the play screen, reducing the load on the GPU.
- **`render_game_behind`** (enabled): Determines whether the GPU will continue rendering the game (the 3D world) behind the screen; we don't want to see the game freeze when opening the overlay screen.
- **`$screen_animations`** and **`$background_animations`** (with an empty array value): To remove the screen's animations, and to eliminate the delay when opening and closing it.

Then an issue emerged where most of the play screen components disappeared, such as the touch circle (crosshair) and the hotbar. The solution to this problem was the **`force_render_below`** property, which Ty-el's told me would reduce issues if added. It forces the GPU to render game components when opening a screen over it (as happens with the pause screen and chest screen).

After all this, it was noticed that the overlay screen buttons did not work, and anything you clicked would "pierce" through the screen to reach the screen behind it. Because of this, I added the **`prevent_touch_input`** property and enabled it, which blocks touch reception. Why enable it? This might seem strange at first, but adding it to the buttons that touches pass through solves the problem; afterwards, the touch event stops at the overlay screen and does not reach what's behind it.

## Result and Publishing

Thus, all issues were resolved, and the project became complete and ready for use. I published it and made it open-source for add-on developers to utilize. However, it didn't gain widespread popularity due to the issues, and because it only works on touch screens; mouse, keyboard, and controllers are incompatible with it, as the buttons added to the screen cannot be clicked with them.

## Testing the Idea on Other Screens

### `npc_interact` Screen

I tested the same algorithm on other screens like `npc_interact`, and I was able to do the exact same thing: adding buttons to the screen with the ability to make them execute commands, but it has issues like the inability to change the hotbar slot.

!(https://github.com/YasserNull/server-form-overlay/raw/main/docs/images/screenshot11.jpg)

### Bed Screen (`bed`)

I also tested it on the sleep screen `bed`, but the game hides the control buttons in the new oreui interface for this screen. However, when reverting to a pre-oreui update to use the old bed screen, the control buttons appear, making it possible to hit and build while sleeping.

!(https://github.com/YasserNull/server-form-overlay/raw/main/docs/images/screenshot9.jpg)

However, movement is not possible because the player remains fixed in place. As shown in the following images, you can use a bow to shoot arrows while sleeping, as well as throw snowballs, and use emotes.

![Animated image showing the use of emotes while sleeping](https://github.com/YasserNull/server-form-overlay/raw/main/docs/images/emotes.gif)
![Animated image from another player's perspective showing the use of emotes while sleeping](https://github.com/YasserNull/server-form-overlay/raw/main/docs/images/emotes2.gif)
![Animated image from the player's perspective showing throwing snowballs while sleeping](https://github.com/YasserNull/server-form-overlay/raw/main/docs/images/snowball.gif) 
![Animated image from another player's perspective showing throwing snowballs while sleeping](https://github.com/YasserNull/server-form-overlay/raw/main/docs/images/snowball2.gif)

I also discovered another bug where if I tap on the screen (right click), it clicks on the bed and tries to sleep in it, and a `this bed is occupied` message appears in the chat even though the player is already sleeping in the same bed.

!https://encrypted-tbn1.gstatic.com/licensed-image?q=tbn:ANd9GcQr7mSCgZbSlQUoRlE3ZHIPlcbKCM3RZ2ZFsEOjf47vZfhPXRgQfWLuULWAAUuV1AURCY4TaLC5FGs9B98(https://github.com/YasserNull/server-form-overlay/raw/main/docs/images/this_bed_is_occupied.gif)

### Crafting Screen (`crafting`)

The idea was also tested on the crafting screen, which is the inventory screen, and it became possible to move while using the inventory, but this is almost useless.

### Chest Screen (`chest`) — The Most Dangerous Results

When testing it on the chest screen, the most dangerous results emerged.

A hack add-on was created that works purely via a Resource Pack, and it became possible to use the chest while moving, building, and hitting simultaneously. This is considered cheating, especially if used on servers, because it deceives other players, making them believe the player is only opening a chest while in reality they are moving, building, and attacking.

![Animated image showing how I deceived a player on the CubeCraft server by opening a chest and dropping it while they approached me](https://github.com/YasserNull/server-form-overlay/raw/main/docs/images/cubecraft.gif)

A bug was also discovered in the game's algorithms: it doesn't check if the player has already opened a chest, allowing multiple chests to be opened at the same time.

Stranger yet, there is another bug, which is that chests don't close after the chest screen is closed, and even after moving away from them, they remain open. This means the player can no longer reach them and is not considered to be currently opening them, but an error occurs and the chests stay visually open. This affects the game; chests remain visible as if they are permanently open, and a chest like a Shulker becomes two blocks instead of one block.

> **Note:** When opening more than one chest and then closing the chest screen, only the last opened chest closes. The same applies if more than one chest is opened and the player moves away from them; only the last opened chest closes and the rest of the chests remain open.

![Animated image showing multiple chests open simultaneously](https://github.com/YasserNull/server-form-overlay/raw/main/docs/images/chests.gif)

When testing this on servers, it doesn't work the same way; you cannot open more than one chest, as trying to open another chest immediately closes the previous one and opens the new one. This looks weird to other players because they see the player opening chests in less than a second.

Another error appears when opening the chest, where two messages appear in the chat:


Something went wrong!
(errcode 1)

![Screenshot showing the error message in the chat](https://github.com/YasserNull/server-form-overlay/raw/main/docs/images/screenshot4.jpg)

There is another bug: when interacting with the screen (right click), the bed is touched as if the player clicked it to sleep, and the message "this bed is occupied" appears, meaning someone is already sleeping in it — which is the very same player trying to sleep in it despite already being asleep in it. This is a very strange bug.

## Summary of Discovered Bugs (For submission to Mojira)

1. **Bed Screen:** Using the old bed interface (pre-oreui) with the overlay screen technique, control buttons remain visible while sleeping, allowing shooting arrows, throwing snowballs, and using emotes while the player sleeps, while remaining fixed in place.
2. **Chest Screen — Potential Cheat:** Via a resource pack only, the chest screen can be kept open while moving, building, and hitting. This could be used to deceive other players on servers regarding the player's true state.
3. **Lack of check for previously opened chest:** The game doesn't check if the player has already opened a chest, allowing multiple chests to be opened simultaneously (in single-player mode).
4. **Chests remaining visually open:** After closing the chest screen or moving away from it, chests remain visible as if they are open (a Shulker box becomes two blocks instead of one block); only the last opened chest is actually closed.
5. **Error message in chat:** The message `Something went wrong! (errcode 1)` appears when opening the chest.
6. **Warning message when clicking the bed the player is sleeping in:** The message `this bed is occupied` appears when you tap the screen to touch the bed as a sleep attempt, showing a message that the bed is occupied by the very same player who is already sleeping in it.

## Links and Sources

- **Project Repository (GitHub):** https://github.com/YasserNull/server-form-overlay
- **Technical Source for Screen Properties:** https://mc.163.com/mcstudio/mc-dev/MCDocs/2-ModSDK%E6%A8%A1%E7%BB%84%E5%BC%80%E5%8F%91/60-UI/4-UI%E8%AF%B4%E6%98%8E%E6%96%87%E6%A1%A3.html
- **Mojira Bug Tracker (To submit the report):** https://report.bugs.mojang.com/servicedesk/customer/portals
