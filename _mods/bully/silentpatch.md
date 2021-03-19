---
title: SilentPatch
title-image: "assets/img/mods/silentpatch-bully.png"
game-series: "bully"
excerpt: "If you are on Windows 10, now you should be able to play without crashes."
date: 07-03-2020
beta-caption: Patch may not fix all crashes yet.
---

*Co-developed with [P3ti](https://github.com/P3ti)*

This game, which shares a lot of the internals with GTA games, performs fairly well in its PC incarnation as is.
However, it's more than likely that you have at some point spotted the amount of complaints Windows 10
users have about the game, or maybe you have encountered crashes yourself.

SilentPatch attempts to fix Bully memory management completely, so it behaves in the same way independent
of Windows version. This is not the only fix included, however - most notably, it attempts to improve
gameplay experience by improving frame pacing, as well as fixing a few other issues.

Fixes marked with <i class="fas fa-cog"></i> can be configured/toggled via the INI file.

## Featured fixes:
### Crash and bug fixes
* Collision loading code has been improved, fixing occasional crashes on initial game load
* Fixed game's objects pool usage, fixing possible crashes
* Fixed an occasional crash when starting Nutcrackin' or Music Class
* Fixed numerous instances of memory corruption on game exit
* Fixed an use-after-free in sound streaming code, causing a rare crash when talking to people
* Fixed handle leaks in audio code, preventing handles from accumulating during the game
* Fixed several memory leaks in audio code, preventing out of memory crashes during extended play sessions
* <i class="fas fa-cog"></i> Made memory manager workarounds toggleable via the INI file -- disabled by default, to be removed in the future
* Frame Limiter has been made much more precise, so the game should lock at exactly 30FPS now
  (as opposed to stock limiter being prone to dropping frames a lot)
* Fixed an issue where game would use more CPU than required when minimized

### Quality of life improvements
* <i class="fas fa-cog"></i> An option to change FPS cap has been added to SilentPatchBully.ini file (game defaults to 30 FPS)
* **FILE_FLAG_NO_BUFFERING** flag has been removed from IMG reading functions - potentially speeding up streaming


## Submitting feedback
Since this is a public beta release, you may encounter crashes. Because of this, MiniDumper utility has
been shipped together with SilentPatch. In case of a crash, a .dmp file will be created in your game directory.

If you want to report it as a bug (any feedback is very much appreciated), first **ENSURE YOU HAVE AN UNMODDED GAME**
(texture mods are fine, scripts - not so much). You can report a bug (.dmp file + a brief explanation on what
you were doing when the game crashes) on the GitHub Issue page.

{% include setup-instructions.html %}

<a href="https://github.com/CookiePLMonster/SilentPatchBully/releases/latest/download/SilentPatchBully.zip" class="button" role="button">{{ site.theme_settings.download_icon }} Download</a> \\
<a href="https://github.com/CookiePLMonster/SilentPatchBully" class="button github" role="button">{{ site.theme_settings.github_icon }} See source on GitHub</a>