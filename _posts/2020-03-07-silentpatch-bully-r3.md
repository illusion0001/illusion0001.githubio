---
layout: post
title: "Third public beta release of SilentPatch for Bully: Scholarship Edition!"
excerpt: "Numerous new crash fixes and removed sketchy workarounds."
date: 2020-03-07 18:45:00 +0100
tags: [Releases]
---

Today marks **exactly** 2 years since
[the first build of SilentPatch for Bully: Scholarship Edition was released](https://twitter.com/__silent_/status/971434668129218566).
Believe it or not, but it's a total coincidence that today I am releasing Build 3 of the patch,
which may also be **the final beta release**! Featuring numerous fixes to crashes plaguing the game and **removal**
of workarounds introduced in the first build should make for the most significant update released so far.

If all goes well, the game may very well be now free of crashes.
If that proves to be true, this release will be promoted to a non-beta release in a few weeks. We'll see!

This release has also been co-developed by [P3ti](https://github.com/P3ti) -- hats off to him for figuring most
of the audio related fixes introduced in this build!

# Changes since the previous release
* Fixed numerous instances of memory corruption on game exit
* Fixed an use-after-free in sound streaming code, causing a rare crash when talking to people
* Fixed handle leaks in audio code, preventing handles from accumulating during the game
* Fixed several memory leaks in audio code, preventing out of memory crashes during extended play sessions
* Made memory manager workarounds toggleable via the INI file -- disabled by default, to be removed in the future

Curiously, all three audio related bug fixes relate to code using WinAPI, which was also used in the X360 port --
so if you're playing on the console and your game crashes (especially during long playthroughs), it may be due to one of those =)

I will also publish a detailed write-up about these issues at a later point, since they are fairly amusing.
Use-after-free issue actually relies on something which I am willing to believe **could be the cause why only Windows 10 users
encountered these crashes often**
-- naturally, the code was glitched since day one, but now I can see where Windows 10 could have possibly changed its
behaviour enough to expose the game bug 😉

Naturally, all third party dependencies (Ultimate ASI Loader, Mod Updater) have been updated to the newest available versions.

# Removed workarounds?
As mentioned above, this release of the patch removes (or to be precise, hides under an INI option which is disabled by default)
workarounds introduced in the first build. Back in 2018, we did not have enough knowledge about the real cause of most crashes
in the game, other than people claiming _it crashes only on Windows 10_. The first SP for Bully attempted to address those
by bringing parity between Bully and GTA memory allocators, and surprisingly enough -- it helped a lot of people!

However, now we know that this was only a happy coincidence and it only hid the real issue -- and to make things worse,
streaming reportedly regressed, making the game not release resources related to unloaded areas properly.
This is theoretized to be the reason behind the remaining crashes people had with SP installed,
especially since they did seem to be "out of memory" crashes.

As of Build 3, changes related to memory allocators have been hidden under a `CustomMemoryMgr=0` INI option.
It is **NOT** recommended to touch it, unless you still are encountering crashes and want to check if this changes anything.
If that is the case, please report your crash **before** you enable that option!

Happy playing! I hope you'll now be able to enjoy this game the way it should've been from the very beginning 🙂

***

The newest build can be downloaded from *Mods & Patches* section here: \\
<a href="{% link _games/bully.md %}#silentpatch" class="button" role="button" target="_blank">{{ site.theme_settings.download_icon }} Download</a>
