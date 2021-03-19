---
title: SilentPatch
title-image: "assets/img/mods/silentpatch-farcry.png"
game-series: "far-cry"
excerpt: "Water reflections shall work properly again."
date: 06-07-2018
---

Far Cry, once a game considered an example of visual fidelity and de facto a benchmark of then-modern PCs, turns out not to be free of issues.

The main reason for creating this patch were broken water reflections - landmass would not reflect on water if the game is played on anything newer than Windows XP. This patch aims to fix this issue without a need for any D3D wrappers.

## Featured fixes:
### Crash and bug fixes
* Water reflections now work as expected on modern Windows versions
* Vertical Sync option now works as expected - it used to be ignored by the game completely, defaulting to VSync off
* Fixed a crash occuring when using a mouse scroll wheel during loading screens
* Fixed a crash/freeze on exit when using a 64-bit executable
 
### Quality of life improvements
* **-64bit** commandline option has been added - when it's used, the game will always attempt to launch using a 64-bit executable. This is useful for getting a 64-bit version of the game to work via Steam, as previously it was required to swap files around.

<p class="mod-screenshot" align="center">
<a href="https://i.imgur.com/hizKXrW.jpg"><img src="https://i.imgur.com/hizKXrWl.jpg"></a>
</p>

{% include setup-instructions.html %}

<a href="https://github.com/CookiePLMonster/SilentPatchFarCry/releases/latest/download/SilentPatchFarCry.zip" class="button" role="button">{{ site.theme_settings.download_icon }} Download</a> \\
<a href="https://github.com/CookiePLMonster/SilentPatchFarCry" class="button github" role="button" target="_blank">{{ site.theme_settings.github_icon }} See source on GitHub</a>