---
layout: post
title: "Analyzing the official Yakuza 3, 4, 5 patches & SilentPatch update"
excerpt: "Checking the official changes for fun, and adding support for Yakuza 5."
thumbnail: "assets/img/posts/spyrc-r2/bg2.jpg"
feature-img: "assets/img/posts/spyrc-r2/bg2.jpg"
image: "assets/img/posts/spyrc/silentpatch_yakuza.png"
date: 2021-02-27 22:00:00 +0100
tags: [Articles, Releases]
twitter: {card: "summary_large_image"}
---

*TL;DR - if you are not interested in an in-depth overview of what was wrong with the game and how it was fixed,
scroll down to the [**Download**](#download) section for a download link.*

***

Disclaimer: This post was almost ready a few days ago and was supposed to be an analysis of QLOC's patch together
with a SilentPatch update. However, just before releasing I was made aware of several issues with Yakuza 5 and looked into them.
This resulted in SilentPatch receiving support for Yakuza 5, which naturally delayed everything a few days.
Because of that, this post now consists of two parts, with Part 1 being the "original" patch analysis and Part 2 being a Yakuza 5 release post.

# Part 1 -- official patch analysis

It's been a bit over three weeks since
[I released a SilentPatch for Yakuza 3 and 4]({% post_url 2021-02-05-silentpatch-yakuza-remastered-collection %})
and around a week ago those games (together with Yakuza 5) received [official patches](https://store.steampowered.com/news/app/1088710/view/3041589319532558479).
The majority of the changelog is shared across three games, with only Yakuza 3 receiving a single "exclusive" fix.

Let's have a look at this shared changelog!

> * Proper dedicated GPU detection so that the game won’t try to launch on integrated GPUs.
> * Setting ‘Auto’ as default framerate settings.
> * High CPU usage thread – CPU usage has been cut down by ~30%.
> * Random crash when ending fights with Heat Move – we managed to fix a crash occurring occasionally after finishing battles with a Heat Action.

That changelog is... very similar to [what SilentPatch]({% link _games/yakuza/yakuza-3.md %}#silentpatch) brings!
This means that the fixes I bring need to be re-visited, as they might be obsolete. While I'm at it, let's also take a look at **how**
QLOC fixed those issues!

## High CPU usage

In the last blog post, I [identified several hot points]({% post_url 2021-02-05-silentpatch-yakuza-remastered-collection %}#burning-cpu-cycles-for-fun)
causing the needlessly high CPU usage and fixed some of them. My approach had some side effects, though:
* Loading times got significantly longer.
* I was not able to easily patch the biggest offender, `pxd::server_job`. Throttling that thread affected performance negatively, and a proper fix was invasive.

A quick test run with [Special K](https://special-k.info/) (and without CPU usage fixes from SilentPatch) shows that the CPU usage is indeed significantly lower:

<p align="center">
<img src="{% link assets/img/posts/spyrc-r2/y3-cpu-usage-before.png %}">
</p>

`pxd::server_job` thread is now reasonable with its CPU usage! A quick dive into the disassembly reveals that QLOC made this thread wait for jobs on a **condition variable**,
whereas a pre-patch version periodically checked if it has any jobs to serve and slept otherwise. Great job (pun unintended)! That's exactly how I would have solved this problem
if I could reasonably hook up such invasive changes via SP... Well OK, I could have done it regardless -- but since it would have required a lot of effort,
I wanted to wait and see if it was going to be fixed officially. Turns out it was the right call.

With this change done officially, I was able to remove most of the CPU usage fixes SilentPatch introduced -- this means that with an updated SilentPatch,
**loading times will not be longer anymore!**

Although...

I did say I removed *most* fixes because QLOC did not identify one of the CPU hogs I found. As you can see on the screenshot above,
`message_pump_thread` is still spinning even after the official patch. My fix still works as-is and relieves the CPU further in both Yakuza 3 and 4.
Now the CPU usage looks exactly like I envisioned it to be when I started the first version of the patch!

<p align="center">
<img src="{% link assets/img/posts/spyrc-r2/y3-cpu-usage-after.jpg %}">
</p>

FWIW the fact QLOC did not include this particular fix also makes me confident that they did **not** investigate my patch when working on their fixes.
While the code for all the other fixes is largely useless for developers with source access, this one fix could have been easily referenced in source code.
It would make no sense for QLOC to reference the more "complex" fixes while leaving the lowest hanging fruit behind, so I am convinced they (sadly) are
not aware of what I did.

## Random crash when ending fights with Heat Move

When in the first release of SilentPatch I worked around the crash after finishing fights with a Heat Move/Head Action, I said:

> ...as a workaround I added additional parameter validation to the function, and... it seems to have helped!
> [...]
> As for an official fix, **I would hope QLOC performs an in-depth investigation of this bug**

Additionally, I elaborated on that in the code comments:
```cpp
// Post-battle race condition crash workaround
// HACK! A real fix is probably realistically not possible to do without
// the source access.
// The game crashes at
// movsx eax, word ptr [rdx+rcx*8]
// so add an early out from the job if rdx is 0
```

Much to my disappointment, when I dissected a patched Yakuza 3 and checked QLOC's changes to this particular function,
they... seemed to have applied an identical workaround!

<p align="center">
<img src="{% link assets/img/posts/spyrc-r2/y3-crash-hack.jpg %}"><br>
<em>Coincidentally, it matches down to the same register usage.</em>
</p>

Note -- I did not verify if QLOC did anything else to fix this bug (admittedly, it would be hard to prove),
but in my opinion the part I am aware of a rushed way to fix this issue. The real fix would not have touched this function in the first place,
and instead, it would have fixed the bug leading to incorrect data being passed to that function. Or in other words,
the patched function is merely a "victim" of some other place in code sending the wrong data.

Even worse, some people report those crashes have not been fully solved, so it's possible that there is something else to that crash
both me and QLOC missed.

## Other

In the first post, I outlined some of the issues I could not realistically fix. Sadly, out of all those only **one** thing got addressed officially.
The rest still applies, so I'll list it in hopes of bringing more visibility to those issues:

> * Yakuza 3, 4, and 5 all require SSE4.2 and AVX. It seems like the games were compiled with `/arch:AVX` and so it's virtually impossible to remove that requirement
  via an ASI file. I could patch out the checks trivially, but I'm sure the game would just crash right away on startup.
> * Some minigames appear to be affected by FPS cap changes. I have not looked into them yet, but I would expect them to be fixed officially by QLOC.
  I might pick these issues up at a later point if they remain unfixed.
> * Something may be wrong with the camera sensitivity when using the controller. Reportedly it is much faster than on PS4, despite the fact both versions run at 60 FPS.
> * Reportedly, Yakuza 3 often locks up in infinite loading when entering hostess clubs.
> * ~~On laptops with an integrated and discrete GPU, all 3 games default to an integrated graphics card as the game executables don't~~
  ~~export [NvOptimusEnablement](https://docs.nvidia.com/gameworks/content/technologies/desktop/optimus.htm) and~~
  ~~[AmdPowerXpressRequestHighPerformance](https://gpuopen.com/learn/amdpowerxpressrequesthighperformance/) variables.~~ **-- FIXED!**

# Part 2 -- SilentPatch for Yakuza 5

Just as I was wrapping this blog post, I was made aware of a bug seemingly only present in Yakuza 5:

<div align="center">
<blockquote class="twitter-tweet"><p lang="en" dir="ltr">Yakuza 5 enters the list of &quot;games which cannot handle non-ASCII user names&quot; with it&#39;s photo booth crash 😐 <a href="https://t.co/87JmrvXJe1">https://t.co/87JmrvXJe1</a><a href="https://twitter.com/qlocsa?ref_src=twsrc%5Etfw">@qlocsa</a> śhóułd hąvę ręąłły ćhęćkęd ćąśęś łikę thiś, it&#39;ś vęry ęąśy with Półiśh diąćritićś...<br>For now, I guess that means Yakuza 5 <a href="https://twitter.com/hashtag/SilentPatchIt?src=hash&amp;ref_src=twsrc%5Etfw">#SilentPatchIt</a></p>&mdash; Silent (@__silent_) <a href="https://twitter.com/__silent_/status/1364503326126198784?ref_src=twsrc%5Etfw">February 24, 2021</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>
</div>

Sounds familiar? Indeed, it's a topic I've been covering more times than I'd want by now, yet again an example of a game not handling Unicode paths correctly.
Thankfully, in the case of Yakuza 5, it's not completely broken -- only the photo booth functionality seems to be affected, as it saves photos to a separate user directory!

As always with issues like this, first I modified the game's code to use fake paths to user directories, then used
[Process Monitor](https://docs.microsoft.com/en-us/sysinternals/downloads/procmon) to log all IO calls, and sure enough, there it is -- not all paths passed to the API are well-formed:

<p align="center">
<img src="{% link assets/img/posts/spyrc-r2/40180893.780000016_image.png %}">
</p>

<p align="center">
<img src="{% link assets/img/posts/spyrc-r2/40209212.1_image.png %}"><br>
<em markdown="1">Process Monitor can even show what was the specific API call used and where did it come from. Insanely useful! In this case, I see the game made a call to `GetFileAttributesA`,
erroneously passing a UTF-8 path.</em>
</p>

Thankfully, the usual set of fixes works fine:
* I assumed all ANSI functions used in the game are given UTF-8 paths and converted them properly before passing to the wide char equivalents.
* I stopped the game from ever converting wide char strings **to** ANSI strings. Now UTF-8 is used across the board.

Those fixes may have been a bit more overreaching than needed, but they work fine! Photo booths now work properly even with my weird debug user paths 🍪⟑η∏☉ⴤℹ︎∩₲ ₱⟑♰⫳🐱.

Is this everything? Obviously not! Just like Yakuza 3 and 4, Yakuza 5 likes hogging CPU threads a bit more than it should, although it does so in a different way than the first two games:

<p align="center">
<img src="{% link assets/img/posts/spyrc-r2/Yakuza5.exe.unpacked_IoQoR9dYjJ.jpg %}">
</p>

This time, `rt_resize_thread` is a culprit. What does it do when spinning? Not much -- in fact, most of the time it's endlessly looping on this tiny chunk of code!

<p align="center">
<img src="{% link assets/img/posts/spyrc-r2/thread-loop.png %}">
</p>

Later on, I identified this thread as the one responsible for controlling the dynamic resolution feature. This makes the entire issue awkward -- a feature which is intended
to help maintain performance in GPU bound scenarios ends up contributing towards CPU bottlenecks by spinning an entire CPU core endlessly 😅
To make this even worse, even when the dynamic resolution option is disabled in settings, this thread still exists and still takes up CPU time!

Thankfully, I got lucky there. Based on my findings, I think this thread performs meaningful work only in two scenarios:
* Once on game start.
* Once a boolean variable indicating the resolution needs to change is set.

With this knowledge, I inserted an additional event wait to this hot loop and I signal it once on game startup and once when the boolean variable gets set.
The result is **greatly** reduced CPU usage and dynamic resolution still working fine -- in fact, `rt_resize_thread` now uses no resources and has vanished
from Special K's performance metrics!

<p align="center">
<img src="{% link assets/img/posts/spyrc-r2/Yakuza5.exe.unpacked_0ihiUdWxgs.jpg %}">
</p>

QLOC could (and **should**) have done the same. At this point I'm not convinced it will be patched officially, but... who knows?

# Download

The modification can be downloaded in *Mods & Patches*. Click here to head to the game's page directly (the same download works for **Yakuza 3, Yakuza 4, and Yakuza 5**):

<a href="{% link _games/yakuza/yakuza-3.md %}#silentpatch" class="button" role="button" target="_blank">{{ site.theme_settings.download_icon }} Download SilentPatch for Yakuza 3/Yakuza 4/Yakuza 5</a> \\
After downloading, all you need to do is to extract the archive to the game's directory, and that's it! Not sure how to proceed? Check the [Setup Instructions]({% link pages/setup-instructions.md %}).
**Those patches only work with the Steam version of the games!**

***

For those interested,
full source code of the mod has been published on GitHub, so it can be freely used as a point of reference: \\
<a href="https://github.com/CookiePLMonster/SilentPatchYRC" class="button github" role="button" target="_blank">{{ site.theme_settings.github_icon }} See source on GitHub</a>
