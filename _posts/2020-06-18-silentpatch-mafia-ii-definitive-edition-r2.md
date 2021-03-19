---
layout: post
title: "Second release of SilentPatch for Mafia II: Definitive Edition"
thumbnail: "assets/img/games/bg/m2de.jpg"
feature-img: "assets/img/games/bg/m2de.jpg"
image: "assets/img/games/bg/m2de.jpg"
excerpt: "Patching the patch."
date: 2020-06-18 22:45:00 +0200
tags: [Releases]
---

A small note before I start -- the website now has a dark/light mode switch! Click a <i class="fas theme-icon"></i> button in the top right corner (might have to expand the menu with a <i class="fas fa-bars"></i> button on mobile devices first) to switch the theme!

***

So... Mafia II: Definitive Edition has been updated. Here's an official changelog from the official website:

> Addressed a reported concern that the 2K Account icon was sometimes getting stuck on the screen \\
> **[PC] Addressed a reported concern that legacy saves were sometimes not working on Steam** \\
> [PS4] Improved game performance on PlayStation®4 consoles \\
> [PS4] Addressed a reported concern that the "Enforcer" trophy would sometimes not unlock on PS4 \\
> Addressed reported concerns related to game audio \\
> Addressed reported concerns related to refining gameplay

The **bold** entry has been reported by me! Very nice.

Does it mean SilentPatch can be deprecated? Sadly not. You might notice that there is no mention of
a fix for saving issues for users with non-ASCII user names. Sure enough,
based on my tests I can confidently say that **this has not been fixed**.
I'm disappointed because the legacy saves issue was fixed despite being a tiny annoyance,
while an actual major (breaking for some) bug regarding saving has not been addressed. Better luck next time?

Because of this, SilentPatch has been updated to work with the 18.06 patch. Since the legacy saves issue
is fixed officially, my fix for that has been removed. Everything else remains as is.

This release also introduces a new tiny change -- Mafia II Classic and Mafia II Definitive Edition
can now run together at the same time. While for regular players this sounds useless, it might be useful
for modders.

# Download

The modification can be downloaded in *Mods & Patches*. Click here to head to the game's page directly:

<a href="{% link _games/mafia2-de.md %}#silentpatch" class="button" role="button" target="_blank">{{ site.theme_settings.download_icon }} Download SilentPatch for Mafia II: Definitive Edition</a> \\
After downloading, all you need to do is to extract the archive to the game's directory and that's it! Not sure how to proceed? Check the [Setup Instructions]({% link pages/setup-instructions.md %}).

**DISCLAIMER: The following patch has been made with the 18.06 patch version in mind.
Bugs fixed by SilentPatch may be addressed in an official patch in the nearby future. If this happens, this SilentPatch
will be updated not to include fixes for those and/or completely deprecated. If SilentPatch detects an unknown executable version,
it will ask the user if they want to continue using it or not.**

***

For those interested,
full source code of the mod has been published on GitHub, so it can be freely used as a point of reference: \\
<a href="https://github.com/CookiePLMonster/SilentPatchM2DE" class="button github" role="button" target="_blank">{{ site.theme_settings.github_icon }} See source on GitHub</a>