---
layout: post
title: "30th Release of SilentPatch for San Andreas!"
feature-img: "assets/img/silentpatch_banner.png"
thumbnail: "assets/img/silentpatch_banner.png"
image: "assets/img/silentpatch_banner.png"
excerpt: "Now officially compliant with LS-RP rules, so you can roleplay with it without the risk of getting banned."
date: 2019-03-09 21:00:00 +0100
tags: [Releases]
---
Hey there! It's time for another release of SilentPatch for San Andreas!
Almost 5 years after the initial announcement, SilentPatchSA has reached 30 official releases!

Here are all the changes introduced in this release:

GTA San Andreas
----------

* Rhino does not gain extra wheels after being fixed anymore
* Firetruck (firela variant) now has a functional ladder - it can be raised by moving right analog stick down/pressing Num2
* artict3 trailers now can be chained (as it was most likely intended, since the model has a hook dummy which was not functional until now)
* Tug now has a functional tow bar (model has a hook dummy which was not functional until now)
* DFT-30 left middle wheel now displays properly (game now accepts a typo present in its naming)
* Pushing pedestrians against the wall with a vehicle will not trigger passenger's voice lines anymore - instead, now they are triggered when player runs over pedestrians
* Several stat counters now reset on New Game - so the player will not level up quicker after starting New Game from a save
* "To stop Carl..." message now resets properly on New Game
* Pay 'n Spray will not clean the car BEFORE garage doors close anymore - now it cleans them while the car is hidden behind the garage door
* "True Invicibility" option has been added - with the option enabled, police helicopter will not hurt the player when they have an Invicibility cheat enabled (can be toggled on/off; DISABLED by default)

***

While this release does not bring an insane amount of new fixes, it's quite important due to its internal changes:

- Lightbeam fix now contains an INI exceptions list, as described in [this blog post]({% post_url 2019-02-03-clever-bug-exploitation-backface-culling %}).
- This release marks the first time SilentPatch has a special **LS-RP Mode**. For a long time, SilentPatch has been banned from usage on [Los Santos Role Play](https://ls-rp.io/) SA-MP server
due to the fact it [makes weapons visible through car windows](https://cdn.discordapp.com/attachments/360065524681539585/552959835587739718/unknown.png).
This was decided to be advantageous for players who have the patch, as they had access to information stock game did not provide.
From Build 30, SilentPatch detects when the player is playing on LS-RP and **disables** this one fix there. No other SA-MP server is affected!

While it has **not** been announced officially yet, it is more than likely that this and future builds of SilentPatch will be officially allowed to use without the risk of any punishment.
When new information arrives, I will modify this post with up to date changes!

{{ "2019-03-18" | date: page.date-format }} UPDATE: SilentPatch Build 30 is now **officially** allowed for use on LS-RP!

***

The newest build can be downloaded from *Mods & Patches*: \\
<a href="{% link _games/gta/gta-sa.md %}#silentpatch" class="button" role="button">{{ site.theme_settings.download_icon }} Download</a>

As always, enjoy! I hope this release enables even more people to enjoy improvements brought by the 'patch.