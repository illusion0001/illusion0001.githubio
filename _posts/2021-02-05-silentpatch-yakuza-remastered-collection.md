---
layout: post
title: "SilentPatch for Yakuza 3 & Yakuza 4 Remastered"
excerpt: "Reducing CPU usage, working around a crash."
thumbnail: "assets/img/games/bg/yakuza.jpg"
feature-img: "assets/img/games/bg/yakuza.jpg"
image: "assets/img/posts/spyrc/silentpatch_yakuza.png"
date: 2021-02-05 20:20:00 +0100
tags: [Articles, Releases]
twitter: {card: "summary_large_image"}
---

*TL;DR - if you are not interested in an in-depth overview of what was wrong with the game and how it was fixed,
scroll down to [**Download**](#download) section for a download link.*

***

It's been a week since Yakuza Remastered Collection has released on PC and Xbox One. I've been looking forward to these games,
and now with Yakuza 3, 4, and 5 released, we are very close to seeing a complete mainline Yakuza saga on PC!

Like with Yakuza Kiwami 2 (but unlike Yakuza: Like a Dragon!), these ports have been made by [QLOC](https://q-loc.com/).
Yakuza Kiwami 2 [got a SilentPatch several months after the release]({% post_url 2020-01-18-silentpatch-yakuza-kiwami-2 %}),
therefore with the Remastered Collection, I knew I'll be looking into these games
[shortly after launch](https://twitter.com/__silent_/status/1354847807941849091).
As always in cases like this, it didn't take long before something worth looking into showed up.

# Burning CPU cycles for... fun?

**DISCLAIMER: All screenshots shown in this article come from the early stages of Yakuza 3 and 4.
You can reach those segments within one or two hours of playing, so I don't think they can be considered major spoilers.**

I spotted the first oddity even before launching any of the games. For some reason, Yakuza 3 and Yakuza 4 ship with... compatibility options!
The included script file sets the games to use Windows 8 compatibility and overrides the DPI scaling settings.

<p align="center">
<img src="{% link assets/img/posts/spyrc/win8-compat.png %}">
</p>

I can't think of **any** reasons where those would be required in a modern game. Although I can't say for sure what went down,
Windows 8 compatibility is highly likely a placebo fix for something -- I find it unlikely it was the only viable option.
DPI scaling settings, on the other hand, should have been set in the executable's manifest instead of being forced through compatibility settings!

The next issue may not be obvious, but something is definitely wrong -- both Yakuza 3 and Yakuza 4 are insanely CPU intensive,
[much more than Yakuza 0 and even Yakuza Kiwami 2](https://twitter.com/__silent_/status/1355595366327066626)!
My 4-core, 8-thread CPU regularly saw 50-60% overall CPU load, with some cutscenes going as far as 90%.

<p align="center">
<img src="{% link assets/img/posts/spyrc/y3-cpu-usage.jfif %}">
<em>A simple cutscene using 90% CPU time? Hmmm.</em>
</p>

A quick look at Steam forums reveals it's a widespread issue. Even worse, for some people, it seems to have caused performance issues
(it is possible if the CPU heats up and throttles or if threads are starving each other for CPU time).

This time for my investigations I used [Special K](https://special-k.info/). Most people use it as a general-purpose framework
to address the most common issues in various games and/or to retrofit HDR, but I instead used its extensive resource monitoring capabilities.
With proper setup, it can act almost as a profiler!

<p align="center">
<img src="{% link assets/img/posts/spyrc/Yakuza3_yVQSYbuTtT.jpg %}">
</p>

Special K makes the worst offenders instantly noticeable, so I could come forward with several fixes. I'll spare the implementation details
because they aren't useful -- instead, I'll briefly explain the changes I made. In the best case, the information presented this way could
even come in handy for QLOC themselves.

* The message pump thread processing Windows messages was spin locking using `PeekMessage`, and therefore hogging the entire CPU core even if there
  was nothing to process. I modified the message loop to use `GetMessage` to **wait** for messages and free CPU time.
  This change made the message pump thread disappear from the profiling results and already relieved the CPU noticeably.
    <p align="center">
    <img src="{% link assets/img/posts/spyrc/Yakuza3.exe.unpacked_Rw9YqDYIaG.jpg %}">
    </p>
* Thread scheduling appears not to give away enough CPU time from idle threads. Both Yakuza 3 and 4 widely use `Sleep(0)` to idle threads,
  but that ends up not giving away CPU time to other threads if there is no other thread (at the same or higher priority level) ready,
  and thus in practice threads end up spinning for no apparent reason. In this case, I settled for a mixed solution -- non-time-critical threads
  regain CPU time properly by using `Sleep(1)` instead, while the frame limiter loop now uses `SwitchToThread()` to yield only when it is possible.
  This keeps the frame limiter loop spinning the CPU, but the other threads don't. Most notably, the current worst offender, `pxd::server_job`
  stopped hogging the CPU in Yakuza 3.
    <p align="center">
    <img src="{% link assets/img/posts/spyrc/Yakuza3.exe.unpacked_gHukXwkcly.jpg %}">
    </p>
* Even though Yakuza 4 appears to be extremely similar to 3 from the technical side, the situation there was slightly more complicated.
  In this case, throttling `pxd::server_job` destroyed the game's performance, so I had to give up on that particular change.
  Therefore, Yakuza 4's CPU usage is still lower than without SilentPatch, but the difference is not as drastic as it is in Yakuza 3.
    <p align="center">
    <img src="{% link assets/img/posts/spyrc/Yakuza4.exe.unpacked_R9ffXIsicf.jpg %}">
    </p>

# Other issues and things I can't fix

Shortly after I sent out the first patch version for testing, I've been notified it causes crashes when finishing fights with a Heat Action.
What we originally thought was the patch fault seems to be an [original game bug occurring randomly](https://twitter.com/__silent_/status/1356311638509543427),
but SilentPatch made this bug happen consistently!

I would like to emphasize here that I don't know exactly what causes this crash. It seems to be a race condition between a scheduled asynchronous
job and something else, and the job does not have its input data prepared correctly, and thus crashes. Thankfully, said job already has some failure handling in place,
so as a workaround I added additional parameter validation to the function, and... it seems to have helped!
<p align="center">
<img src="{% link assets/img/posts/spyrc/Yakuza3.exe.unpacked_iZtebt5zR0.jpg %}">
<em>At last, no crashes.</em>
</p>

As for an official fix, I would hope QLOC performs an in-depth investigation of this bug, as with source access it should be much easier
than when operating on a compiled executable. Scheduling changes I mentioned above make this bug 100% reproducible,
which is IMO the most important aspect for getting the bug fixed quickly. Fingers crossed!

***

On a closing note, there are also some issues I was not able to address, and some of them are realistically not fixable by anyone except for QLOC:
* Yakuza 3, 4, and 5 all require SSE4.2 and AVX. It seems like the games were compiled with `/arch:AVX` and so it's virtually impossible to remove that requirement
  via an ASI file. I could patch out the checks trivially, but I'm sure the game would just crash right away on startup.
* Some minigames appear to be affected by FPS cap changes. I have not looked into them yet, but I would expect them to be fixed officially by QLOC.
  I might pick these issues up at a later point if they remain unfixed.
* Something may be wrong with the camera sensitivity when using the controller. Reportedly it is much faster than on PS4, despite the fact both versions run at 60 FPS.
* Reportedly, Yakuza 3 often locks up in infinite loading when entering hostess clubs.
* On laptops with an integrated and discrete GPU, all 3 games default to an integrated graphics card as the game executables don't
export [NvOptimusEnablement](https://docs.nvidia.com/gameworks/content/technologies/desktop/optimus.htm) and
[AmdPowerXpressRequestHighPerformance](https://gpuopen.com/learn/amdpowerxpressrequesthighperformance/) variables.

# Download

The modification can be downloaded in *Mods & Patches*. Click here to head to the game's page directly (the same download works for **both Yakuza 3 and Yakuza 4**):

<a href="{% link _games/yakuza/yakuza-3.md %}#silentpatch" class="button" role="button" target="_blank">{{ site.theme_settings.download_icon }} Download SilentPatch for Yakuza 3/Yakuza 4</a> \\
After downloading, all you need to do is to extract the archive to the game's directory, and that's it! Not sure how to proceed? Check the [Setup Instructions]({% link pages/setup-instructions.md %}).
**Those patches only work with the Steam version of the games!**

***

For those interested,
full source code of the mod has been published on GitHub, so it can be freely used as a point of reference: \\
<a href="https://github.com/CookiePLMonster/SilentPatchYRC" class="button github" role="button" target="_blank">{{ site.theme_settings.github_icon }} See source on GitHub</a>
