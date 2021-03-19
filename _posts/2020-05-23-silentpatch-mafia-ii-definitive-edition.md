---
layout: post
title: "SilentPatch for Mafia II: Definitive Edition, fixing (some) saving issues"
thumbnail: "assets/img/games/bg/m2de.jpg"
feature-img: "assets/img/games/bg/m2de.jpg"
image: "assets/img/games/bg/m2de.jpg"
excerpt: "Non-English characters in save path? Got you covered."
date: 2020-05-23 16:20:00 +0200
tags: [Releases]
---

For those expecting a lengthy post, detailing all the gritty details behind the bugs -- sorry, not this time!

On the 20th of May 2020, a remaster of Mafia II released on Steam. A free upgrade for existing Mafia II owners had its ups and downs,
but without a doubt it's a welcome release, bringing this classic game to modern consoles and refreshing it with newer technology.

Sadly, it has issues. Some people report performance issues, crashes, whereas Windows 7 users found that you can't run the game on this OS
without moving a few system files around. It's safe to assume that the developers are aware of the issues and are working to fix them,
and besides, they are not the kind of issues that are feasible to solve without the source access.

However, there are a few exotic issues that could be fixed. Namely, you may have noticed posts like these on the Steam Community boards:
> If you started to play the game, went to the main menu and found that you did not save anything, then you have a problem.
> It consists in the fact that in the path to the location of the Documents folder you have Cyrillic or some other characters (not Latin).
> The problem is in mafia 2 since 2010. But in the normal version, the saving is stored in the Appdata folder, and in the new one in the "Documents" folder.

This issue is a classic case of misinterpreting Unicode paths, same as the ones I've previously analyzed e.g. in Yakuza 0 or Metal Gear Rising.
Granted, workarounds exist, but why not fix them?

SilentPatch fixes paths handling, so the game handles them the way it should've been since day one and will (hopefully) be soon patched officially. \\
What's more, I came up with an attempt to "rescue" saves placed in a wrong directory! If the user has at any point attempted to 'fix' saving
by running the game as an Admin, their saves will be **gone** once the issue is officially patched. SilentPatch tries its best to detect this
event and offers a fix:

<p align="center">
<img src="https://cdn.discordapp.com/attachments/360065524681539585/713480937169748078/unknown.png">
</p>

If the user agrees to have their saves moved, it essentially future-proofs saves against a future official patch.

One more issue I handled relates to saves from the original Mafia II. Did you know that the game is supposed to copy saves from the original game?
You might not be aware of this feature, because it's not mentioned anywhere in-game, and the code responsible for that was bugged.
Alas, now it works but it might be a bit too late if the users have already started playing the game from the beginning :(

# Download

Without further ado, the modification can be downloaded in *Mods & Patches*. Click here to head to the game's page directly:

<a href="{% link _games/mafia2-de.md %}#silentpatch" class="button" role="button" target="_blank">{{ site.theme_settings.download_icon }} Download SilentPatch for Mafia II: Definitive Edition</a> \\
After downloading, all you need to do is to extract the archive to the game's directory and that's it! Not sure how to proceed? Check the [Setup Instructions]({% link pages/setup-instructions.md %}).

**DISCLAIMER: The following patch has been made with the launch game version in mind.
Bugs fixed by SilentPatch may be addressed in an official patch in the nearby future. If this happens, this SilentPatch
will be updated not to include fixes for those and/or completely deprecated. If SilentPatch detects an unknown executable version,
it will ask the user if they want to continue using it or not.**

# What's next?

Since it's been just a few days since the game launched, all issues covered there might be fixed. I did what I could to bring those issues to the attention of the developers,
so we might see those resolved officially soon.

***

For those interested,
full source code of the mod has been published on GitHub, so it can be freely used as a point of reference: \\
<a href="https://github.com/CookiePLMonster/SilentPatchM2DE" class="button github" role="button" target="_blank">{{ site.theme_settings.github_icon }} See source on GitHub</a>