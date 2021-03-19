---
title: SilentPatch
title-image: "assets/img/mods/silentpatch-gta.png"
order: -100
game-series: "gta-sa"
excerpt: "Fixes numerous issues in your favourite game."
date: 22-02-2020
---

These games are surely great, nonetheless they have some issues. This modification attempts to fix some of them.

Fixes marked with <i class="fab fa-steam-symbol"></i> are supported in 1.0, Steam and Rockstar Games Launcher versions. Fixes without that symbol require 1.0.\\
Fixes marked with <i class="fas fa-cog"></i> can be configured/toggled via the INI file. These options are enabled by default, unless stated otherwise.

**Featured fixes:**
* <i class="fab fa-steam-symbol"></i> 14ms frame delay has been removed. As a result, game now locks properly on 30 FPS instead of 25 FPS
* <i class="fab fa-steam-symbol"></i> More precise frame limiter, reducing lag spikes a bit when playing with Frame Limiter on
* Game timers now tick in a more accurate manner, making them not freeze if framerate exceeds 1000 frames per second; in other words, this fixes occasional freezes on fadeouts if playing with Frame Limiter off
* <i class="fab fa-steam-symbol"></i> Mouse should not lock up randomly when exiting the menu on newer systems anymore
* <i class="fab fa-steam-symbol"></i> Mouse vertical axis sensitivity now matches horizontal axis sensitivity
* <i class="fab fa-steam-symbol"></i> Mouse vertical axis does not lock during camera fadeins now
* NUM5 is now bindable (like in 1.01)
* 16:9 resolutions are now selectable (like in 1.01)
* <i class="fab fa-steam-symbol"></i> If the settings file is absent, the game will now default to your desktop resolution instead of 800x600x32
* <i class="fab fa-steam-symbol"></i> DirectPlay dependency has been removed - this should improve compatibility with Windows 8 and newer
* <i class="fab fa-steam-symbol"></i> Path to the GTA San Andreas User Files directory is now obtained differently, increasing compatibility and future-proofing the games more
* A heap corruption in one place is now fixed (did not affect gameplay but could potentially make the game crash)
* <i class="fas fa-cog"></i> EAX/NVIDIA splashes are now removed
* <i class="fas fa-cog"></i> Subtitle and Radio text sizes can now be toggled between the original release and updated Steam version
* <i class="fas fa-cog"></i> Area name colour now matches the gang colour of the gang that owns that territory (off by default)
* Wet road reflections render properly again (just like with Road Reflections Fix)
* <i class="fab fa-steam-symbol"></i> Fixed sun lens flare effect not appearing with AMD/Intel graphics cards
* <i class="fab fa-steam-symbol"></i> Fixed an issue introducing graphical artifacts from ped animations with high RAM usage - so called "streaming memory bug"
* <i class="fab fa-steam-symbol"></i> Fixed a bug causing cheat-spawned melee weapons to be forcibly replaced by other melee weapons upon walking into a pickup
* <i class="fas fa-cog"></i> Helicopter rotors and plane propellers now work correctly. They now have a blurring effect present in VC and PS2 version of SA
* Hunter interior does not dissapear when viewed through the glass door panel
* <i class="fas fa-cog"></i> Dual rear wheels now show up properly (Yosemite, Tanker etc.)
* Weapons are now visible when viewed through a vehicle window
* Holding a weapon will not cause some objects to be incorrectly lit anymore
* Blown up vehicles are now correctly coloured and no longer shine (like in 1.01 and Steam versions)
* Dirty cars are now able to get clean (like in 1.01)
* Each car has an unique numberplate now
* Custom numberplates now show up correctly in all cases
* Custom numberplates are now also allowed on bikes
* Numberplates are now bilinear filtered, resulting in a smoother look
* Vehicle lights do not get dark after being being initially lit anymore (like on PS2)
* Detached vehicle parts will now remain the same colour as the vehicle they came from
* Detached vehicle parts now render from both sides
* <i class="fab fa-steam-symbol"></i> Some car panels now swing after car's explosion (like they were meant to be but the code forcibly fixed them immediately after damaging)
* Moonphases now show up correctly, like on PS2 version (only when playing in 32-bit colour mode)
* Toggling car lights on does not make windows invisible when viewed from inside anymore
* Illumination value from timecyc.dat now accepts any float value in 0.0-2.0 ranges, not just 0.0, 1.0 and 2.0
* In addition, if illumination value is absent from the timecycle (like on a stock PC timecycle), the game will now default to 1.0
* Lights now get casted on vehicles and peds properly - previously, they'd dissapear under some conditions
* <i class="fab fa-steam-symbol"></i> In addition, when playing on Visual FX Quality higher than low, the game will now cast up to 6 lights on each model both indoors and outdoors (on Low details, game's stock behaviour has been kept - allowing up to 4 lights per model outdoors and 6 indoors)
* Muzzle flash looks better now
* <i class="fab fa-steam-symbol"></i> Muzzle flash will now show up when firing the last bullet from the clip
* Coronas' don't have a Z test forced all the time now - as a result, sunglare now matches original PS2 version
* With User Tracks automatic scan enabled, MP3 playback will now work properly if QuickTime is not installed
* User Tracks now supports the FLAC codec (Only 8/16/24bits, Mono/Stereo and up to 48Khz)
* PCM WAVE has been expanded to also accept additional profiles (Now 8/16/24bits, Mono/Stereo and up to 48Khz)
* PCM WAVE files with an ID3-TAG will now also work with the game
* Temple and Queens are now correctly called on the police scanner
* Travelling far away from the map will no longer trigger the extra gang territories glitch, nor will it corrupt the Taxi Driver submission
* <i class="fas fa-cog"></i> Gym glitch ("You have worked out enough..." showing infinitely) has been fixed
* <i class="fas fa-cog"></i> Saving in Madd Dogg's mansion will no longer trigger the missing basketball glitch
* <i class="fas fa-cog"></i> Fixed an occasional softlock in Mountain Cloud Boys - the player will not freeze after arriving to the meeting anymore
* <i class="fas fa-cog"></i> Possible softlock in Sweet's Girl initial cutscene fixed
* <i class="fas fa-cog"></i> Quadruple Stunt Bonus now works correctly
* <i class="fab fa-steam-symbol"></i> Script sprites now have bilinear filtering applied
* <i class="fab fa-steam-symbol"></i> Car generator counters now work properly for generators with fixed amount of spawns
* Impound garages now function correctly, allowing the player to recover his last vehicle after it had vanished after a mission start
* In addition, impound garages will now store player's car when he's busted
* Streamed entity list has been expanded a bit, so now the game world shouldn't flash when looking down with high Draw Distance settings anymore
* Mouse rotates an airborne car only with Steer with Mouse option enabled
* Towtruck tow hitch does not get bugged after it has been fixed anymore
* Plane doors don't corrupt after the plane has been fixed anymore
* Fixing a plane will now reset its moving props to an undamaged state
* Several vehicle components (most notably, Rumpo's front bumper and Bandito's moving prop) will not get glitched after the vehicle has been fixed anymore
* Weapons and a jetpack now cast proper shadows
* Crosshair doesn't mess up weapon icon when on a jetpack anymore
* Free resprays will not carry on a New Game now
* Fixed ambulance and firetruck dispatch timers - they reset on New Game now
* Several stat counters now reset on New Game - so the player will not level up quicker after starting New Game from a save
* "To stop Carl..." message now resets properly on New Game
* Previously present only on PS2, 'Cars drive on water' cheat is now toggleable - its string is SEAROADER
* <i class="fab fa-steam-symbol"></i> Randomizer error causing peds not to spawn in some areas has been fixed
* <i class="fab fa-steam-symbol"></i> Randomizer error causing prostitutes to be quiet during solicit has been fixed
* <i class="fab fa-steam-symbol"></i> Text boxes now can show together with a Mission Passed text
* Fixed a 1.01 only tiny memory leak which occured every time the player switched a radio station
* <i class="fab fa-steam-symbol"></i> Fixed an occasional crash when Alt+Tabbing back to the game while standing next to a mirror
* <i class="fab fa-steam-symbol"></i> Mirror reflection doesn't break with Anti-Aliasing enabled anymore
* <i class="fab fa-steam-symbol"></i> With Visual FX Quality set to Very High, mirror reflection quality has been bumped
* <i class="fab fa-steam-symbol"></i> Anti-Aliasing option has been altered - instead of listing 1, 2, 3 options (which in fact are 2x/2x/4x MSAA), the game will now show proper MSAA values from 2x up to 16x (depending on max MSAA level supported by the graphics card)
* <i class="fab fa-steam-symbol"></i> Colliding with another car will now damage proper parts on both cars - previously, both cars got damaged the same way
* <i class="fab fa-steam-symbol"></i> Fixed a crash on car explosions - most likely to happen when playing with a multimonitor setup
* <i class="fab fa-steam-symbol"></i> Fixed a crash when entering advanced display options on a dual monitor machine after: starting game on primary monitor in maximum resolution, exiting, starting again in maximum resolution on secondary monitor. Secondary monitor maximum resolution had to be greater than maximum resolution of primary monitor.
* <i class="fab fa-steam-symbol"></i> Fixed an occasional crash occuring when standing next to escalators
* <i class="fab fa-steam-symbol"></i> Slightly reduced stencil shadows memory overhead
* <i class="fab fa-steam-symbol"></i> Fixed an AI issue where enemies became too accurate after the player has been in the car earlier
* IMGs bigger than 4GB are now handled properly
* <i class="fab fa-steam-symbol"></i> Alt+F4 now works properly
* Several vehicles now have extra animated components: Phoenix hood scoops, Sweeper brushes, Newsvan antenna, radars on several boats, extra flaps on Stuntplane and Beagle
* Animated engine components on Bandito, BF Injection and Hotknife will not animate if the engine is off
* <i class="fab fa-steam-symbol"></i> Fixed a crash occuring when the vending machine was continuously used for an extended period of time
* <i class="fab fa-steam-symbol"></i> FILE_FLAG_NO_BUFFERING flag has been removed from IMG reading functions - speeding up streaming
* <i class="fab fa-steam-symbol"></i> Fixed a streaming related deadlock, which could occasionally result in game being stuck on black screen when entering or exiting interiors (this is the issue people used to fix by setting CPU affinity to one core)
* <i class="fab fa-steam-symbol"></i> Metric-to-imperial conversion constants have been replaced with more accurate ones
* <i class="fab fa-steam-symbol"></i> Fixed a glitch where random cars would end up being impounded to garage, replacing player's vehicles
* Very long loading times will now loop loading screens, as opposed to fading to white
* <i class="fas fa-cog"></i> Sun reflections on peds and vehicles now change direction depending on time of day, like in III and VC (off by default)
* <i class="fab fa-steam-symbol"></i> Dancing minigame timings have been improved, now they do not lose accuracy over time depending on PC's uptime
* <i class="fab fa-steam-symbol"></i> Car generators placed in interiors are now placed correctly - this 'unhides' two vehicles in Madd Dogg's mansion, which were always there but they were not visible
* <i class="fab fa-steam-symbol"></i> Bombs in cars stored in garages now save properly
* <i class="fab fa-steam-symbol"></i> Fixed an issue which would cause games to freeze if III/VC/SA were running at the same time
* <i class="fab fa-steam-symbol"></i> Streaming has been greatly improved during Supply Lines mission (or more general, any time when using an RC vehicle) - it now behaves as expected, as opposed to displaying LODs way too quickly
* <i class="fab fa-steam-symbol"></i> Health triangle displaying when aiming at peds is now properly orientated (it's now upside down) for peds player can recruit
* <i class="fab fa-steam-symbol"></i> Setting a BMX on fire will not set CJ on fire anymore
* <i class="fab fa-steam-symbol"></i> Keyboard input latency decreased by one frame
* Rhino does not gain extra wheels after being fixed anymore
* Firetruck (firela variant) now has a functional ladder - it can be raised by moving right analog stick down/pressing Num2
* artict3 trailers now can be chained (as it was most likely intended, since the model has a hook dummy which was not functional until now)
* Tug now has a functional tow bar (model has a hook dummy which was not functional until now)
* DFT-30 left middle wheel now displays properly (game now accepts a typo present in its naming)
* Pushing pedestrians against the wall with a vehicle will not trigger passenger's voice lines anymore - instead, now they are triggered when player runs over pedestrians
* Pay 'n Spray will not clean the car BEFORE garage doors close anymore - now it cleans them while the car is hidden behind the garage door
* <i class="fas fa-cog"></i> "True Invicibility" option has been added - with the option enabled, police helicopter will not hurt the player when they have an Invicibility cheat enabled (off by default)
* <i class="fas fa-cog"></i> Made the game select metric/imperial units basing on system locale settings
* Fixed a bug where paintjobs would vanish from cars stored in garage if they were stored without looking at them
* <i class="fab fa-steam-symbol"></i> Coronas now properly rotate as camera is getting closer to them, like on PS2
* <i class="fab fa-steam-symbol"></i> Light shadows from fire now show up properly
* <i class="fab fa-steam-symbol"></i> Fixed parachute animations
* <i class="fab fa-steam-symbol"></i> "Keep weapons after wasted" and "keep weapons after busted" now reset on New Game
* <i class="fab fa-steam-symbol"></i> Fixed a glitch allowing bikes without engines to spawn
* <i class="fab fa-steam-symbol"></i> Allowed extra6 part to be picked when a random extra is to be picked
* <i class="fab fa-steam-symbol"></i> Fixed in-car camera mouse behaviour when looking left/right/behind
* <i class="fab fa-steam-symbol"></i> Steam and RGL versions have proper aspect ratios now
* <i class="fab fa-steam-symbol"></i> Steam/RGL version of the game will not reject 1.0/1.01 saves anymore (still, a compatible SCM is needed for the save to work)
* <i class="fab fa-steam-symbol"></i> Censorships from Steam and RGL versions for German players have been removed
* <i class="fab fa-steam-symbol"></i> Steam/RGL versions will now default Steer with Mouse option to disabled, like in 1.0/1.01

{% include setup-instructions.html %}

<a href="https://silent.rockstarvision.com/uploads/SilentPatchSA.zip" class="button" role="button">{{ site.theme_settings.download_icon }} Download</a> \\
<a href="https://gtaforums.com/topic/669045-silentpatch/" class="button forums" role="button">{{ site.theme_settings.gtaf_icon }} Discuss on GTAForums</a>

Requires <a href="#asiloader">ASI Loader<a> (or [Ultimate ASI Loader](https://github.com/ThirteenAG/Ultimate-ASI-Loader/releases/download/v4.52/Ultimate-ASI-Loader.zip) for Rockstar Games Launcher version).