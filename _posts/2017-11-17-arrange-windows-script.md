---
title: Auto-arranging windows on a 2nd display with AppleScript
---

If you have both a Mac laptop and an external display at your desk, you may know this routine: sit down, plug in the cables, drag each application's windows to your preferred spot on each display, get up for a coffee break, come back and find all your open windows have congregated on a single screen again after your display went to sleep.

Maybe you've noticed that hidden applications aren't affected, and so you try to remember to hide all apps when you step away from your desk. Maybe you've created a key combo to switch to Finder and hide other apps using a utility like [Butler](https://manytricks.com/butler/). Maybe you've even considered paying for a utility like [Stay](https://cordlessdog.com/stay/) to solve this exact problem.

But you can do better than that, can't you? You're a hacker, after all.

So went my thought process when I finally tired of going through that click-drag click-drag click-drag routine multiple times per day, eventually resulting in the AppleScript below:

> [arrange windows.applescript](https://github.com/EricFromCanada/byte-bucket/blob/master/applescript/arrange%20windows.applescript)

## Installation

1. Open Script Editor in `/Applications/Utilites` and go to its Preferences. Under the General pane, check the box to show the Script menu in the menu bar.

2. Create a new script in Script Editor and paste in the text from the link above, then save as `arrange windows.scpt` in `~/Library/Scripts`.

	Creating a new `.scpt` file instead of running a plain-text `.applescript` file allows the system to save a compiled version of the script, instead of having to recompile it on each run. (Note that simply changing a `.applescript` file's extension to `.scpt` won't work.)

3. Edit the two lists under `(*** SETTINGS ***)` to include the applications you want to reposition.

	For example, to move Mail and Photos to the built-in display, set **windowPositionsBuiltIn** to:

	```applescript
	{appName:"Photos"}, ¬
	{appName:"Mail"} ¬
	```

	And to move Safari, Firefox, iTunes, and Terminal to the four corners of the external display, set **windowPositionsExternal** to:

	```applescript
	{appName:"Safari", y:"top", x:"left"}, ¬
	{appName:"Firefox", y:"top", x:"right"}, ¬
	{appName:"iTunes", y:"bottom", x:"right"}, ¬
	{appName:"Terminal", y:"bottom", x:"left"} ¬
	```

	Remember not to have a comma after the last item of each list.

After saving, selecting "arrange windows" from the Script menu will make it loop through each listed application (if running, switching spaces if necessary) and arrange the windows for each, tiling them horizontally if an app has more than one window open, then hiding it if the application was originally hidden. When done, it returns to the application you originally had in front.

## Nerdy Details

- You probably don't need this script if you have "Displays have separate Spaces" enabled in _System Preferences > Mission Control_, which allows windows to remember their positions on external displays. This is for the rest of us who prefer a single menu bar and the `⌃ ←` and `⌃ →` keyboard shortcuts to switch between Spaces.

- Getting the resolutions of all connected displays is done by scraping the output of the `system_profiler SPDisplaysDataType` shell command. One problem here is that for a built-in Retina display, the command returns its full hardware resolution, rather than the scaled-down display resolution. (For example, a 15" Retina display will always show "2880 x 1800", whether the display is set to 1440 x 900 [Default] or 1920 x 1200 [More Space], or any other scaled resolution.) Conversely, if a HiDPI external display is connected, the command will show its current display resolution. (**Update 2022-11:** As of macOS 10.13, the command shows for external HiDPI displays both the full hardware resolution beside "Resolution:" and the scaled display resolution beside "UI Looks like:". The script has been updated to fetch the latter value if present.) Until someone figures out how to pull the current display resolution for Retina displays, the script isn't able to calculate window positions for the built-in display.

- Your application windows will remember their positions on the display with the menu bar, so if your menu bar is set to occupy the external display when connected, this won't be as big of a problem; only the applications you prefer on the smaller built-in display will need to be manually moved back. The script can do this for you, but until a workaround for the issue above is found, it'll place them all near the top left of the built-in display.

- The script isn't very precise about where it places windows. I coded it with the assumption that an application's windows would take up a quarter of the external display, so when told to move a window to the bottom right, the script will move it halfway across and halfway down; similarly, middle center is located a quarter across and a third down. It also doesn't resize windows. (For now.)

- Setting a window's position is done using the coordinates fetched by the code `bounds of window of desktop`, which returns a list of four values. These represent the top left coordinate and the width and height of a square encompassing all connected displays, where the origin is the top left of the display containing the menu bar, as set in _System Preferences > Displays_. Checking if either of the first two values are negative indicates where the secondary display is relative to the primary, allowing the script to convert display-relative coordinates to desktop-relative coordinates accordingly.

- Performing the window moves is most reliably done by first bringing each application to the front, then having System Events count its windows and set their positions. System Events can only count an application's windows that are in the current Space, unless the application is hidden. (Asking the application directly returns an accurate count, but works only for AppleScript-aware applications.)

- The script would sometimes behave strangely when told to activate a non-AppleScript-aware application, e.g. `tell application "GitHub Desktop" to activate` would be sent to "GitHub Desktop Helper". This was avoided by activating applications using the line `set frontmost of process anApp to true` via System Events.
