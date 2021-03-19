---
title: SilentPatch
game-series: "w101"
excerpt: "Improved frame pacing and standardized hotkeys."
date: 17-07-2020
---

At the moment of writing this text, The Wonderful 101: Remastered is not released yet, but it was given early to Kickstarter backers.
Since time is of an essence, this SilentPatch aims to fix a few issues the community has noticed during those few days.
Most notably, SilentPatch drastically improves the game's frame pacing, so it's now locked to stable 60 FPS.

Additionally, SilentPatch now brings a much requested feature absent from the game (courtesy of Banz99) -- an option to switch between
the original and remastered soundtracks!

Notice: This patch has only been tested with version 1.0.0. Future official patches (if applicable) may have fixed some of those issues,
in which case SilentPatch will skip them.

Fixes marked with <i class="fas fa-cog"></i> can be configured/toggled via the INI file.

## Featured fixes:
* The game's frame limiter has been rewritten for greater accuracy. Previously it would lock the game at 59 FPS and calculations would drift
  over time, effectively making the game's frame rate more unstable over time. It's now been fixed to be rock solid 60 FPS with consistent frametime, regardless of playtime.
* Escape key used to put the game in the windowed mode has been remapped to Alt+Enter. Additionally, that hotkey can now cycle between windowed, fullscreen, and borderless
  instead of only putting the game in the windowed mode.
* <i class="fas fa-cog"></i> 60 FPS cap can now optionally be disabled from the INI file. **WARNING: The game does not support high frame rates and it will speed up,**
  **so use this option ONLY if you use VSync or external frame limiters (like RTSS).**
* Error messages now display properly on PC's with a non-Japanese locale.
* <i class="fas fa-cog"></i> Original OST from the Wii U version of the game can now be enabled from the INI file.

## Credits
* [Banz99](https://github.com/Banz99) - OriginalOST feature

{% include setup-instructions.html %}

<a href="https://github.com/CookiePLMonster/SilentPatchW101/releases/latest/download/SilentPatchW101.zip" class="button" role="button">{{ site.theme_settings.download_icon }} Download</a> \\
<a href="https://github.com/CookiePLMonster/SilentPatchW101" class="button github" role="button" target="_blank">{{ site.theme_settings.github_icon }} See source on GitHub</a>