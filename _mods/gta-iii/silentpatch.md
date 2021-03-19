---
title: SilentPatch
title-image: "assets/img/mods/silentpatch-gta.png"
order: -100
game-series: "gta-iii"
excerpt: "Fixes numerous issues in your favourite game."
date: 22-02-2020
---

These games are surely great, nonetheless they have some issues. This modification attempts to fix some of them.

Fixes marked with <i class="fas fa-cog"></i> can be configured/toggled via the INI file. These options are enabled by default, unless stated otherwise.

## Featured fixes:
* Mouse will not go beyond the game rect now, allowing to play the game on multimonitor setups without problems
* More precise frame limiter, reducing lag spikes a bit when playing with Frame Limiter on
* The game will not ask for a CD when all audio files are copied to the disk anymore
* In 1.0, armour cheat is now TORTOISE - like in 1.1 and Steam versions
* In 1.0, BOOOOORING cheat now works properly - like in 1.1 and Steam versions
* Mouse sensitivity is now properly saved - like in 1.1 and Steam versions
* Headlight coronas now display properly (as they do on PS2, XBOX and PC Steam versions)
* Purple Nines Glitch has been fixed
* Game now performs a bit better on high FPS. It doesn't freeze on fadeouts anymore, although it still has issues with car physics, gravity and sounds. Therefore it's still recommended to play with Frame Limiter set to ON
* Rhino spawned via a cheatcode doesn't stay on map forever anymore
* Blista now can be lifted by a car crusher crane - instead, now it cannot lift Coach
* Shooting from M16 in 1st person mode now increments bullets fired stat properly (so you can't score Accuracy to be more than 100%)
* Wet road reflections render properly again (just like with Road Reflections Fix)
* Reintroduced light glows under weapon/health/armour pickups, bribes, hidden packages and money pickups - they showed only on PS2 due to a bug in all PC versions
* Trace (Destination) blip is now scaling to the resolution properly
* Garages-related and rampages-related texts now scale to the resolution properly
* All texts now have proper shadows (depending on chosen resolution, without the fix they'd appear thinner etc.)
* Free resprays will not carry on a New Game now
* Fixed ambulance and firetruck dispatch timers - they reset on New Game now
* The game will not create more than 32 blips at once now - while this never happened in an unmodded game, it could have happened due to exploits and potentially corrupted the save
* FILE_FLAG_NO_BUFFERING flag has been removed from IMG reading functions - speeding up streaming
* Alt+F4 now works properly
* Some car panels now are detached after car's explosion (like they were meant to be but the code forcibly fixed them immediately after damaging)
* Metric-to-imperial conversion constants have been replaced with more accurate ones
* Pathfinding for cars chasing the player has been improved (most notably, it may result in 'Bait' being much more playable)
* All censorships from German and French versions of the game have been removed
* Bombs in cars stored in garages now save properly
* Fixed an issue which would cause games to freeze if III/VC/SA were running at the same time
* Car generator counters now work properly for generators with fixed amount of spawns
* Keyboard input latency decreased by one frame
* Fixed a crash after playing the game for a short amount of time without a sound card
* <i class="fas fa-cog"></i> Made the game select metric/imperial units basing on system locale settings
* Fixed corona lines rendering on non-NVIDIA graphics cards
* Corrected FBI Car secondary siren sound
* <i class="fas fa-cog"></i> Fixed siren corona placements in Firetruck, Ambulance, Enforcer
* <i class="fas fa-cog"></i> Fixed taxi light corona placement for Taxi
* <i class="fas fa-cog"></i> Fixed police chopper's search light placement
* Enlarged the bounding box of Catalina’s chopper and the police chopper to prevent it from being cut off on screen edges
* Fixed cranes and night windows disappearing when viewed from up close
* Fixed a glitch allowing lightless taxis to spawn in traffic
* Car reflections are now displayed correctly on Steam version

{% include setup-instructions.html %}

<a href="https://silent.rockstarvision.com/uploads/SilentPatchIII.zip" class="button" role="button">{{ site.theme_settings.download_icon }} Download</a>
<a href="https://silent.rockstarvision.com/uploads/SilentPatchDDraw.zip" class="button" role="button">{{ site.theme_settings.download_icon }} Download DDraw Component</a> \\
<a href="https://gtaforums.com/topic/669045-silentpatch/" class="button forums" role="button">{{ site.theme_settings.gtaf_icon }} Discuss on GTAForums</a>