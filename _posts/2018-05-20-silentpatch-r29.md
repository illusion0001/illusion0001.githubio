---
layout: post
title: "New SilentPatch builds for Grand Theft Auto!"
feature-img: "assets/img/silentpatch_banner.png"
thumbnail: "assets/img/silentpatch_banner.png"
image: "assets/img/silentpatch_banner.png"
excerpt: "New builds for GTA III, GTA VC and GTA SA!"
date: 2018-05-20 11:10:00 +0200
redirect_from: "/2018/05/20/silentpatch-r29.html"
tags: [Releases]
---
What's up? It's been over a year since a patch for GTA San Andreas and nearly **two years** since a patch for III and Vice City were updated!
However, today is the day where all three of them receive an update at the same time! These releases also celebrate the 4th anniversary of SilentPatchSA, which was first announced on 27.04.2014.

Without further ado, here are all the changes introduced in new releases:

GTA III
----------
* `FILE_FLAG_NO_BUFFERING` flag has been removed from IMG reading functions - speeding up streaming
* Alt+F4 now works properly
* Some car panels now are detached after car's explosion (like they were meant to be but the code forcibly fixed them immediately after damaging)
* Metric-to-imperial conversion constants have been replaced with more accurate ones
* Pathfinding for cars chasing the player has been improved (most notably, it may result in 'Bait' being much more playable)
* All censorships from German and French versions of the game have been removed
* Bombs in cars stored in garages now save properly
* Fixed an issue which would cause games to freeze if III/VC/SA were running at the same time
* Car generator counters now work properly for generators with fixed amount of spawns
* Keyboard input latency decreased by one frame

GTA Vice City
----------
* `FILE_FLAG_NO_BUFFERING` flag has been removed from IMG reading functions - speeding up streaming
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

GTA San Andreas
----------
Aside from fixes listed below, it's worth to mention that this release removes several features -- namely, *two pass rendering* and *NVC shader* are both gone.
If you still want to continue using those, try [SkyGfx](https://gtaforums.com/topic/750681-skygfx-ps2-and-xbox-graphics-for-pc) -- it provides both features and they are far superior to my implementations.

As for fixes:
* Several vehicles now have extra animated components: Phoenix hood scoops, Sweeper brushes, Newsvan antenna, radars on several boats, extra flaps on Stuntplane and Beagle
* Animated engine components on Bandito, BF Injection and Hotknife will not animate if the engine is off
* Fixed a crash occuring when the vending machine was continuously used for an extended period of time
* `FILE_FLAG_NO_BUFFERING` flag has been removed from IMG reading functions - speeding up streaming
* Fixed a streaming related deadlock, which could occasionally result in game being stuck on black screen when entering or exiting interiors (this is the issue people used to fix by setting CPU affinity to one core)
* Metric-to-imperial conversion constants have been replaced with more accurate ones
* Fixed a glitch where random cars would end up being impounded to garage, replacing player's vehicles
* Very long loading times will now loop loading screens, as opposed to fading to white
* Sun reflections on peds and vehicles now change direction depending on time of day, like in III and VC (can be toggled on/off; DISABLED by default)
* Dancing minigame timings have been improved, now they do not lose accuracy over time depending on PC's uptime
* Car generators placed in interiors are now placed correctly - this 'unhides' two vehicles in Madd Dogg's mansion, which were always there but they were not visible
* Bombs in cars stored in garages now save properly
* Fixed an issue which would cause games to freeze if III/VC/SA were running at the same time
* Streaming has been greatly improved during Supply Lines mission (or more general, any time when using an RC vehicle) - it now behaves as expected, as opposed to displaying LODs way too quickly
* Health triangle displaying when aiming at peds is now properly orientated (it's now upside down) for peds player can recruit
* Setting a BMX on fire will not set CJ on fire anymore
* Keyboard input latency decreased by one frame

The newest builds can be downloaded from *Mods & Patches* section here: \\
<a href="{% link _games/gta/gta-iii.md %}#silentpatch" class="button" role="button">{{ site.theme_settings.download_icon }} Download for GTA III</a>
<a href="{% link _games/gta/gta-vc.md %}#silentpatch" class="button" role="button">{{ site.theme_settings.download_icon }} Download for GTA Vice City</a>
<a href="{% link _games/gta/gta-sa.md %}#silentpatch" class="button" role="button">{{ site.theme_settings.download_icon }} Download for GTA San Andreas</a>

However, expect the 'patches (and other mods) to appear here permanently sometime soon... That's what this website should be for, after all.

EDIT: \\
What happened? Since this release, **I am not hosting SilentPatch on GTAGarage** anymore. Due to a bug on their website (which remains ignored for over a year), every new submission is marked as malware.
This was the final straw for me, so now releases are self-hosted.

EDIT2: \\
Attention, Steam SA users! r29 used to crash on a Steam version, now it doesn't anymore! If you encountered them, please redownload.