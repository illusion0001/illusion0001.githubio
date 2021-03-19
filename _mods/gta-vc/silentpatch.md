---
title: SilentPatch
title-image: "assets/img/mods/silentpatch-gta.png"
order: -100
game-series: "gta-vc"
excerpt: "Fixes numerous issues in your favourite game."
date: 28-12-2019
---

These games are surely great, nonetheless they have some issues. This modification attempts to fix some of them.

Fixes marked with <i class="fas fa-cog"></i> can be configured/toggled via the INI file. These options are enabled by default, unless stated otherwise.

## Featured fixes:
* Fixed an issue where installing the game on A or B drive made the game ask for the CD
* Mouse should not lock up randomly when exiting the menu on newer systems anymore
* Mouse will not go beyond the game rect now, allowing to play the game on multimonitor setups without problems
* Mouse vertical axis sensitivity now matches horizontal axis sensitivity
* Mouse vertical axis does not lock during camera fadeins now
* More precise frame limiter, reducing lag spikes a bit when playing with Frame Limiter on
* Rosenberg's lines which play when player is Busted work correctly now (just like with Rosie's Audio Fix)
* Game now performs a bit better on high FPS. It doesn't freeze on fadeouts anymore, although it still has issues with car physics, gravity and sounds. Therefore it's still recommended to play with Frame Limiter set to ON
* Wet road reflections render properly again (just like with Road Reflections Fix)
* Reintroduced light glows under weapon/health/armour pickups, bribes, hidden packages and money pickups - they showed only on PS2 due to a bug in all PC versions
* All texts now have proper shadows (depending on chosen resolution, without the fix they'd appear thinner etc.
* Free resprays will not carry on a New Game now
* Fixed ambulance and firetruck dispatch timers - they reset on New Game now
* Corrected crime codes for police dispatch audio - police dispatch now refers to player crimes correctly
* Fixed a bug causing cheat-spawned melee weapons to be forcibly replaced by other melee weapons upon walking into a pickup
* FILE_FLAG_NO_BUFFERING flag has been removed from IMG reading functions - speeding up streaming
* Alt+F4 now works properly
* Some car panels now are detached after car's explosion (like they were meant to be but the code forcibly fixed them immediately after damaging)
* Metric-to-imperial conversion constants have been replaced with more accurate ones
* Pathfinding for cars chasing the player has been improved
* All censorships from German and French versions of the game have been removed
* Bombs in cars stored in garages now save properly
* Fixed an issue which would cause games to freeze if III/VC/SA were running at the same time
* Car generator counters now work properly for generators with fixed amount of spawns
* Extras on bikes now behave correctly, following bike lean and not floating in air
* Keyboard input latency decreased by one frame
* <i class="fas fa-cog"></i> Made the game select metric/imperial units basing on system locale settings
* Fixed corona lines rendering on non-NVIDIA graphics cards
* Corrected FBI Washington siren sound
* <i class="fas fa-cog"></i> Fixed siren corona placements in Police, Firetruck, Ambulance, Enforcer, Vice Cheetah, FBI Washington
* <i class="fas fa-cog"></i> Added siren corona to FBI Washington
* <i class="fas fa-cog"></i> Fixed taxi light corona placement for Taxi
* <i class="fas fa-cog"></i> Fixed police chopper's search light and red tail light placement
* Fixed a glitch allowing lightless taxis to spawn in traffic
* Allowed extra6 part to be picked when a random extra is to be picked
* Made Drive-By use correct sounds based on what machine gun is used
* Some props in Malibu Club, Ocean View Hotel and Pole Position Club have been restored; more environment shows outside when player is in interior too (just like on PS2)

{% include setup-instructions.html %}

<a href="https://silent.rockstarvision.com/uploads/SilentPatchVC.zip" class="button" role="button">{{ site.theme_settings.download_icon }} Download</a>
<a href="https://silent.rockstarvision.com/uploads/SilentPatchDDraw.zip" class="button" role="button">{{ site.theme_settings.download_icon }} Download DDraw Component</a> \\
<a href="https://gtaforums.com/topic/669045-silentpatch/" class="button forums" role="button">{{ site.theme_settings.gtaf_icon }} Discuss on GTAForums</a>