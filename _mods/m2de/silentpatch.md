---
title: SilentPatch
game-series: "mafia2-de"
excerpt: "Quick fixes for saving issues."
date: 18-06-2020
---

**DISCLAIMER: The following patch has been made with the launch game version in mind, then updated to work with the 18.06 patch.
Bugs fixed by SilentPatch may be addressed in an official patch in the nearby future. If this happens, this SilentPatch
will be updated not to include fixes for those and/or completely deprecated. If SilentPatch detects an unknown executable version,
it will ask the user if they want to continue using it or not.**

**You have been warned.**

A "quick" patch for Mafia 2: Definitive Edition, resolving one of the high-priority issues with saving for users with non-ASCII user names.
A bug exclusive to the Definitive Edition makes it impossible to save without making changes to the OS configuration -- this SilentPatch release
resolves the issue in the same way it will (hopefully) be resolved in an official patch, thus making the changes future-proof.

## Featured fixes:
* Fixed a number of saving-related issues when the path to Documents contains any non-ASCII characters.
* ~~Fixed a bug where copying saves from the original Mafia 2 would fail quietly.~~ **- fixed in an official patch.**
* Introduced a temporary 'emergency' save migration as an attempt to move the saves from a wrong save directory, if any were created. This could have happened if at any point the user ran the game as an Administrator in an attempt to 'fix' the saving issue.
* Allowed Mafia II Definitive Edition and Mafia II Classic to run at the same time.

{% include setup-instructions.html %}

<a href="https://github.com/CookiePLMonster/SilentPatchM2DE/releases/latest/download/SilentPatchM2DE.zip" class="button" role="button">{{ site.theme_settings.download_icon }} Download</a> \\
<a href="https://github.com/CookiePLMonster/SilentPatchM2DE" class="button github" role="button" target="_blank">{{ site.theme_settings.github_icon }} See source on GitHub</a>