---
layout: post
title: "SilentPatch for Yakuza Kiwami 2 releases -- fixes Toylets and more"
thumbnail: "assets/img/games/bg/yk2.jpg"
feature-img: "assets/img/games/bg/yk2.jpg"
image: "assets/img/games/bg/yk2.jpg"
excerpt: "Peeing minigame in 30 FPS - but also more FPS related fixes."
date: 2020-01-18 14:30:00 +0100
twitter: {card: "summary_large_image"}
tags: [Releases, Articles]
---

TL;DR - if you are not interested in an in-depth overview of what was wrong with the game and how it was fixed, just follow the link to check out a concise changelog and grab **SilentPatch for Yakuza Kiwami 2**: \\
<a href="{% link _games/yakuza/yakuza-kiwami-2.md %}#silentpatch" class="button" role="button" target="_blank">{{ site.theme_settings.download_icon }} Download SilentPatch for Yakuza Kiwami 2</a> \\
Upon downloading, all you need to do is to extract the archive to game’s directory and that’s it!

***

Yakuza Kiwami 2 is IMO a fun game. Remake of Yakuza 2, released on PS4 in December 2017 and ported to PC by [QLOC](https://q-loc.com/) in May 2019 was met with overwhelmingly positive reception
and is the first Yakuza game running on a new Dragon Engine which was ported to PC. It is speculated that this is one of the reasons this port was not outsourced to [Lab42](https://www.lab42.games/),
unlike the PC ports of Yakuza 0 and Yakuza Kiwami.

I have [covered my adventures with Yakuza 0 lighting]({% post_url 2019-02-24-yakuza-0-fixing-attempts %}) more or less a year ago. Both Yakuza Kiwami and Kiwami 2
launched without issues as jarring as this, so at the first glance it looked like I could play this game without thinking about bugs and theoretizing fixes (for once).
However, once again as soon as I say something like this, [karma](https://twitter.com/__silent_/status/1211343094253342720?s=20) makes sure to prove me wrong.

# Creating a patch

I obviously would not be writing this post if the port was free of issues. It does have some, most notably performance issues on relatively recent hardware and several not-so-annoying
FPS issues. While fixing the former (especially without source code access) is likely an extremely challenging task, some of the FPS cap issues could be addressed relatively easily.
That said, known issues like exaggerated physics presumably fall into the same bucket of difficulty as performance optimizations -- but thankfully not all problems are this complex.

All games have bugs, however, and I *usually* don't run in circles trying to fix them all. For this game, my "tipping point" was a thing so absurd so it would be a crime not to
mention it. That thing was a... **peeing minigame called Toylets**[^1]. 😑

When Yakuza Kiwami 2 launched on Steam, one of the highlights of this release were uncapped framerates, compared to the PS4 version which runs at 30 FPS (with the exception of arcade games
which run at 60 FPS). This was *mostly* fine, however it was quickly identified that playing at over 60 FPS (either due to disabled vertical sync or due to a high refresh rate monitor)
alters physics (in a non-breaking way, so it doesn't cause any major side effects other than the fact it looks hilarious during fights) and breaks several minigames -- UFO Catcher
was completely unplayable (same as in Yakuza 0 and Yakuza Kiwami), and so were the arcade games.

Patch quickly followed, in which QLOC changed the following to improve the situation:
- "Framelimit" option has been added to Graphics Options, allowing users to select between 30/60/120/Unlimited.
- Most minigames have been forcibly limited to 60 FPS. However, if user opted to limit the game to 30 FPS, all minigames will be limited to 30 FPS too.

Good changes? Technically yes, but in practice *not quite*. While a forced cap is a good solution to this problem (it is imperative to know that arcade games and Toylets minigame
are actually **using emulated and/or original code**, so limiting them instead of "fixing them" to handle arbitrary framerates is preferable for the sake of accuracy with original products!),
problems slipped by. There are three major issues with these changes:

1. Users complained that setting a FPS cap broke frame pacing and introduced uneven framerates. Indeed, my own tests showed that this option results in very inaccurate limiting.
In the worst case, limiting the game to 120 FPS resulted in it never rendering at more than 115 FPS!
2. As mentioned earlier, arcade games ran at 60 FPS on PS4. After this patch, users can limit them to 30 FPS on PC, which again breaks them! Issues do not seem to be critical,
but they are here either way.
3. Toylets minigame was **still** unplayable, because it was designed to run at 30 FPS! If users wanted to play it the way it was intended, they had to go through a ritual
of going to Graphics Options, setting FPS limit to 30, playing the minigame, then going back... It's doable, but should not be needed.

Let's go through these issues in order and see how they could have been prevented.

[^1]: I could not believe it, but this game [is a real thing in Japan](https://www.engadget.com/2012/05/01/sega-urinal-game-toylets/).

***

## Broken frame pacing

Since frame pacing is generally as good as it gets when FPS cap is set to Unlimited, but breaks pretty badly when 30/60/120 is selected, a newly added FPS limiter is an obvious candidate.
Locating it in the executable did not take long, and that's how it looks in pseudocode:

```cpp
if ( fpsLimit != 0 )
{
    nextFrameTimeUs += 1000000.0 / (float)fpsLimit;
    curTimeUs = GetTimeInMicroseconds();

    while ( curTimeUs < nextFrameTimeUs )
    {
        if ( nextFrameTimeUs - curTimeUs >= 1000 )
            sleep(1);
        curTimeUs = GetTimeInMicroseconds();
    }
    nextFrameTimeUs = curTimeUs;
}
```

Can you spot the issue? It's right here, called `sleep(1)`.

It is well known that sleeping functions in Windows are relatively relaxed about imposed sleep durations.
Depending on numerous factors like OS load and priority of other threads in the same process, it's not unrealistic to expect a "1 millisecond" sleep to take more than
15 milliseconds!

With that in mind, issues in this code should be easily visible -- it expects `sleep` to take exactly 1ms, since it is called even when there is only 1000μs left for the deadline.
In the worst case, frame limiter may "oversleep" by several milliseconds!
Sounds familiar? Indeed, [Bully's FPS limiter had a nearly identical bug]({% link _games/bully.md %}#silentpatch).

What is the best fix here? I personally think busy looping is actually **preferable** here -- while it does use more CPU, it allows for perfectly accurate limiting.
Burning the CPU is probably less bad than it sounds, especially nowadays when you rarely see processors with less than 4 threads. On top of that, it is not unrealistic to
expect the overhead of putting the thread to sleep to be more wasteful than letting the CPU spin -- especially with
[Meltdown](https://en.wikipedia.org/wiki/Meltdown_(security_vulnerability)) mitigations in place.

SilentPatch solves frame pacing issues by not allowing the FPS limiter to put the thread to sleep. **Success!**

## Minigames FPS caps

Looking at the official changelog, we can learn that the following minigames have been capped to 60 FPS:

> *Please note that FPS is locked to 60FPS in the minigames below:
> Batting Center, BlackJack, Darts, Koi Koi, Mahjong, Oichokabu, Poker, ToyLets, Virtual Fighter 2/2.1, Virtual-On, UFO Catcher and Karaoke.

Why so many minigames?? After all, not a single person complained about minigames like Karaoke, Darts or Poker being broken!
Honestly, that's something I am not able to answer with absolute certainty, but I have a possible theory:

In Yakuza 0 and Yakuza Kiwami, **all** these minigames were indeed broken at high framerates, so I can imagine that in Kiwami 2 after the developers learned about arcade games
and UFO Catcher being similarly broken, they decided to pre-emptively limit those minigames just how Lab42 did. The strongest argument towards this theory is that
QLOC did not limit games introduced in Kiwami 2 -- so batting is locked to 60 FPS, but golf is not, even though realistically speaking **if** they were to have any
significant physics issues, both games should have been similarly broken =)

Aside from minigames being needlessly limited, Toylets is obviously still broken -- it should have been limited to 30, not 60 FPS. It's an easy change with the way
this FPS limiter was written, as each game requests a specific maximum framerate instead of it being an on/off toggle. For this reason, I assume it was just a genuine oversight.
However, thanks to this it makes for a very, very simple fix.

Do note I said minigames request a **maximum** framerate, as opposed to forcing it. This may be preferable for UFO Catcher, but is a bad idea for arcade games.
In order to improve the situation, it should have been possible to opt for either behaviour.

SilentPatch solves those issues by doing the following:
* Capping Toylets to fixed 30 FPS.
* Introducing a new FPS limiting mode for minigames, where a specified framerate is forced instead of treated as an upper bound. Arcade games have been set to use this mode,
so they run at 60 FPS regardless of the FPS cap set by the user.
* Reverting FPS caps for the other minigames, returning them to how they were day 1.

***

For those interested,
full source code of the patch has been published on GitHub, so it can be freely used as a point of reference: \\
<a href="https://github.com/CookiePLMonster/SilentPatchYK2" class="button github" role="button" target="_blank">{{ site.theme_settings.github_icon }} See source on GitHub</a>