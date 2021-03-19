---
layout: post
title: "New SilentPatch releases with support for Rockstar Games Launcher version!"
feature-img: "assets/img/posts/rgl.jpg"
thumbnail: "assets/img/posts/rgl.jpg"
image: "assets/img/posts/rgl.jpg"
excerpt: "And also some new fixes for all three games, of course."
date: 2019-09-22 16:25:00 +0200
twitter: {card: "summary_large_image"}
tags: [Releases]
---
Hello! Rockstar surprised everyone by [releasing Rockstar Games Launcher](https://socialclub.rockstargames.com/rockstar-games-launcher) earlier this week,
and also offered a **free copy of San Andreas** for everyone who installs it!
Curiously, this seems to be a brand new version, and SilentPatch sadly did not support it -- until now!

All three GTA's receive new fixes today, but do note that only San Andreas requires an upgrade in order to work with
the Rockstar Games Launcher (called RGL later in this post) version!

Without further ado, let's see all the new changes:

Rockstar Games Launcher support
===============================

As RGL version of San Andreas is based upon a new Steam version, supporting it was relatively easy.
Additionally, starting from this release SilentPatch for San Andreas is going to behave differently upon encountering an "unsupported"
executable -- previous versions would just abort any attempts to patch, but from now on SP will assume this is a "future" executable
and attempt to patch in all Steam/RGL fixes!
For now, if **any** fix fails to be patched, SilentPatch will unload -- for upcoming versions I plan to improve that further
and make SP only skip that specific fix.

**Important note:** Rockstar Games Launcher forcibly overwrites any modified files even if you disable updates for the game!
Because of that, my ASI Loader is nearly unusable, as it's removed every time you restart RGL. Because of this,
I recommend using [Ultimate ASI Loader](https://github.com/ThirteenAG/Ultimate-ASI-Loader/releases/download/v4.52/Ultimate-ASI-Loader.zip), because it does not overwrite any files.

That said, do note that RGL version receives the same level of support as a Steam release,
that is only **some** fixes are currently supported. Those are:

* 14ms frame delay has been removed. As a result, game now locks properly on 30 FPS instead of 25 FPS
* More precise frame limiter, reducing lag spikes a bit when playing with Frame Limiter on
* Mouse should not lock up randomly when exiting the menu on newer systems anymore
* Mouse vertical axis sensitivity now matches horizontal axis sensitivity
* Mouse vertical axis does not lock during camera fadeins now
* If the settings file is absent, the game will now default to your desktop resolution instead of 800x600x32
* DirectPlay dependency has been removed - this should improve compatibility with Windows 8 and newer
* Path to the GTA San Andreas User Files directory is now obtained differently, increasing compatibility and future-proofing the games more
* Fixed sun lens flare effect not appearing with AMD/Intel graphics cards
* Fixed an issue introducing graphical artifacts from ped animations with high RAM usage - so called "streaming memory bug"
* Fixed a bug causing cheat-spawned melee weapons to be forcibly replaced by other melee weapons upon walking into a pickup
* Some car panels now swing after car's explosion (like they were meant to be but the code forcibly fixed them immediately after damaging)
* When playing on Visual FX Quality higher than low, the game will now cast up to 6 lights on each model both indoors and outdoors (on Low details, game's stock behaviour has been kept - allowing up to 4 lights per model outdoors and 6 indoors)
* Muzzle flash will now show up when firing the last bullet from the clip
* Script sprites now have bilinear filtering applied
* Car generator counters now work properly for generators with fixed amount of spawns
* Randomizer error causing peds not to spawn in some areas has been fixed
* Randomizer error causing prostitutes to be quiet during solicit has been fixed
* Text boxes now can show together with a Mission Passed text
* Fixed an occasional crash when Alt+Tabbing back to the game while standing next to a mirror
* Mirror reflection doesn't break with Anti-Aliasing enabled anymore
* With Visual FX Quality set to Very High, mirror reflection quality has been bumped
* Anti-Aliasing option has been altered - instead of listing 1, 2, 3 options (which in fact are 2x/2x/4x MSAA), the game will now show proper MSAA values from 2x up to 16x (depending on max MSAA level supported by the graphics card)
* Colliding with another car will now damage proper parts on both cars - previously, both cars got damaged the same way
* Fixed a crash on car explosions - most likely to happen when playing with a multimonitor setup
* Fixed a crash when entering advanced display options on a dual monitor machine after: starting game on primary monitor in maximum resolution, exiting, starting again in maximum resolution on secondary monitor. Secondary monitor maximum resolution had to be greater than maximum resolution of primary monitor.
* Fixed an occasional crash occuring when standing next to escalators
* Slightly reduced stencil shadows memory overhead
* Fixed an AI issue where enemies became too accurate after the player has been in the car earlier
* Alt+F4 now works properly
* Fixed a crash occuring when the vending machine was continuously used for an extended period of time
* FILE_FLAG_NO_BUFFERING flag has been removed from IMG reading functions - speeding up streaming
* Fixed a streaming related deadlock, which could occasionally result in game being stuck on black screen when entering or exiting interiors (this is the issue people used to fix by setting CPU affinity to one core)
* Metric-to-imperial conversion constants have been replaced with more accurate ones
* Fixed a glitch where random cars would end up being impounded to garage, replacing player's vehicles
* Dancing minigame timings have been improved, now they do not lose accuracy over time depending on PC's uptime
* Car generators placed in interiors are now placed correctly - this 'unhides' two vehicles in Madd Dogg's mansion, which were always there but they were not visible
* Bombs in cars stored in garages now save properly
* Fixed an issue which would cause games to freeze if III/VC/SA were running at the same time
* Streaming has been greatly improved during Supply Lines mission (or more general, any time when using an RC vehicle) - it now behaves as expected, as opposed to displaying LODs way too quickly
* Health triangle displaying when aiming at peds is now properly orientated (it's now upside down) for peds player can recruit
* Setting a BMX on fire will not set CJ on fire anymore
* Keyboard input latency decreased by one frame
* Made the game select metric/imperial units basing on system locale settings. This can be overridden from the INI file
* Steam and RGL versions have proper aspect ratios now
* Steam/RGL version of the game will not reject 1.0/1.01 saves anymore (still, a compatible SCM is needed for the save to work)
* Censorships from Steam and RGL versions for German players have been removed
* Steam/RGL versions will now default Steer with Mouse option to disabled, like in 1.0/1.01

I never bothered backporting more fixes to a Steam version, because most people end up downgrading -- however,
this might change with the arrival of RGL version. Let me know in the comments what do you think about it!

Some notes about the RGL version
--------------------------------

For those curious, when working on the RGL version I figured out some things about this version:
* It's based on a Steam version
* For German players, it's **less** censored than the Steam version -- you can kick people who are down.
  You can also circumvent censorship by changing system locale settings, which will not work in a Steam version.
  Do note SilentPatch removes those censorships!
* It is supposed to accept 1.0 EU saves, but those will crash due to SCM differences between 1.0 and later versions!
* Rockstar **attempted** to fix mouse issues, as I mentioned on Twitter!

<div align="center">
<blockquote class="twitter-tweet"><p lang="en" dir="ltr">Gotta give <a href="https://twitter.com/RockstarGames?ref_src=twsrc%5Etfw">@RockstarGames</a> some credit for San Andreas on RGL, it seems like they moved away from DirectInput based mouse in favour of Windows messages based mouse!<br>What this means for users - mouse should not be breaking anymore!? <a href="https://t.co/Wa4gB6vATs">pic.twitter.com/Wa4gB6vATs</a></p>&mdash; Silent (@__silent_) <a href="https://twitter.com/__silent_/status/1175106767325487104?ref_src=twsrc%5Etfw">September 20, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>
</div>

Sadly, this fix is incomplete and mouse still breaks for most users. Still, have to give Rockstar credit where it's due.

***

Other versions
==============

Not only RGL version of San Andreas got attention. Other versions also received a few new features, and those are:

GTA San Andreas
---------------

* Made the game select metric/imperial units basing on system locale settings. This can be overridden from the INI file
* Fixed a bug where paintjobs would vanish from cars stored in garage if they were stored without looking at them

Additionally, Steam support in San Andreas had a few bugs which remained unnoticed.
When adding support for the RGL version, I ended up identifying and fixing them -- so if you had crashes with SP on a Steam version,
give it another try!

GTA Vice City
-------------

* Made the game select metric/imperial units basing on system locale settings. This can be overridden from the INI file

GTA III
-------

* Fixed a crash after playing the game for a short amount of time without a sound card
* Made the game select metric/imperial units basing on system locale settings. This can be overridden from the INI file

***

The newest builds can be downloaded from *Mods & Patches* section here: \\
<a href="{% link _games/gta/gta-iii.md %}#silentpatch" class="button" role="button">{{ site.theme_settings.download_icon }} Download for GTA III</a>
<a href="{% link _games/gta/gta-vc.md %}#silentpatch" class="button" role="button">{{ site.theme_settings.download_icon }} Download for GTA Vice City</a>
<a href="{% link _games/gta/gta-sa.md %}#silentpatch" class="button" role="button">{{ site.theme_settings.download_icon }} Download for GTA San Andreas</a>

As always, enjoy! Rockstar Games Launcher support should allow SilentPatch to be introduced to more people who are new to the game,
and I hope it will benefit them greatly.