---
layout: post
title: "SilentPatch for The Wonderful 101: Remastered, fixing frame pacing issues"
thumbnail: "assets/img/games/bg/w101.jpg"
feature-img: "assets/img/games/bg/w101.jpg"
image: "assets/img/games/bg/w101.jpg"
excerpt: "Pre-release patching."
date: 2020-05-16 12:50:00 +0200
tags: [Releases, Articles]
---

*TL;DR - if you are not interested in an in-depth overview of what was wrong with the game and how it was fixed,
scroll down to [**Download**](#download) section for a download link.*

***

# Introduction

What's better than SilentPatching a game right after the release? Patching it before the release!

Today's subject is **The Wonderful 101: Remastered**. This remaster of a 2013 Wii U exclusive is effectively a refreshed release of the game
for PC, PlayStation 4, and Nintendo Switch. As of the time of writing this article, it hasn't been released yet, with the digital release scheduled
for May 19th. It effectively makes this SilentPatch release a... *Day -3 Patch*, I guess? 🤔

Where's the catch? The remaster has been crowdfunded on Kickstarter, and one of the perks was an Early Access to the game on Steam.
That's how I was able to obtain the game early.

What are the issues people noticed? There were numerous, but the one which gained my attention the most was broken frame pacing.
Wonderful 101 appears to be yet another game from PlatinumGames which locks to 59 FPS instead of 60 FPS, with the earliest known game being Metal Gear Rising from 2014.
While the community has figured out the workarounds, they aren't permanent and thus aren't optimal.
There are a few other issues reported (such as DoF not scaling above 1080p), but those are IMO more likely to be fixed in an official patch than an issue unsolved for at least 6 years.

## Broken frame pacing

First of all, let's use RTSS to visualize the current frame times:
<p align="center">
<img src="{% link assets/img/posts/spw101/rtss1.jpg %}"><br>
<em>Yikes.</em>
</p>

I'm usually the last person to complain about uneven framerates, but this indeed looks bad. Not only the framerate is all over the place, but it also never reaches stable 60 at all.
This is something worth looking into.

The pseudocode of Wonderful 101's frame limiter code (carried over with no changes from earlier titles from PlatinumGames) looks like this:

```cpp
time = (unsigned int)(GetInvFrequency() * (double)GetProcessTime() * 3.0) - curFrameTime3Ms;
while ( frameTime3Ms > time )
{
    timeLeft = frameTime3Ms - time;
    if ( (signed int)(timeLeft / 3) > 0 )
        Sleep(timeLeft / 3);
    time = (unsigned int)(GetInvFrequency() * (double)GetProcessTime() * 3.0) - curFrameTime3Ms;
}
curFrameTimeQPC = GetProcessTime();
curFrameTime3Ms = (unsigned int)(GetInvFrequency() * (double)GetProcessTime() * 3.0);
```

Deja vu? Indeed, this frame limiter code is structured similarly to the one seen in Yakuza Kiwami 2, [which was also broken]({{ site.baseurl }}{% post_url 2020-01-18-silentpatch-yakuza-kiwami-2 %}#broken-frame-pacing).
At the first glance, both issues are identical so I'll just quote the post regarding YK2:

> Can you spot the issue? It's right here, called `sleep(1)`.
>
> It is well known that sleeping functions in Windows are relatively relaxed about imposed sleep durations.
> Depending on numerous factors like OS load and priority of other threads in the same process, it's not unrealistic to expect a "1-millisecond" sleep to take more than
> 15 milliseconds!

Basing on this, I went forward and removed the `Sleep` call, just like I did with Yakuza. This change also makes perfect sense when we consider that the workaround the community has come up
with was to use [Special K](https://store.steampowered.com/curator/25149011/) and its "Sleepless Render Thread" option, which makes `Sleep` calls do nothing when called on the thread calling rendering functions.

Success? Or so I thought, but we're not done yet.

## Floating-point math is hard

Looking at the guides detailing how Special K fixes frame pacing issues, we can spot a very suspicious line:

> Note: This may stop working after two hours

Since my fix is equivalent to SK disabling sleeps on the render thread, it's also affected. But how is this possible? Patches can't undo themselves, which means there must be another bug creeping in the code.

Cue floating-point precision, something [I wrote about a long time ago]({{ site.baseurl }}{% post_url 2018-08-07-high-resolution-timers-and-uptime-headaches %}). To understand what's going on,
I should explain what do the functions used in the frame limiter code return:
- `GetProcessTime()` returns the time since the game has started in `QueryPerformanceCounter` units (a difference between the current time and process start time). It returns an `uint64`.
- `GetInvFrequency()` the inverse of `QueryPerformanceFrequency`, multiplied by 1000.0. It returns a `float`.

In layman's terms, the process time is a **very big** number and it grows fast, while the inverse of frequency is a **very small**, albeit constant, number.
What seems to happen here is that after around 75-80 minutes of gameplay, those calculations start to drift. At around 3 hours, the process time gets so big frame limiter starts to lock at 58 FPS,
then at 4 hours it springs up to 62 FPS (presumably due to `double`'s exponent value changing, but I have not measured that).

As a test, I faked the game's uptime of 20 hours and compared the calculated frame times with a fresh session. At that point, the game's frame limiter just "gave up" and my game was locked to around 120 FPS.
<p align="center">
<img src="{% link assets/img/posts/spw101/limping-frametime.jpg %}">
</p>

Note: While the output says 50ms, it is, in fact, 50/3ms -- notice how the frame limiter code multiplies the frame time by 3.0. It's done because `16.6(6) * 3 = 5`, and therefore
comparisons can be performed on integers.

It is visibly "limping", which is a good indicator of calculations not being very precise. Therefore, to understand where the inaccuracies come from, I began to replicate this faulty
frame limiter code on my own. Here's what I came up with:

```cpp
void EndFrame_Hook()
{
	// This code replicates Platinum Games frame limiter, with all its inaccuracies
	// Requires /arch:SSE

	orgEndFrame();

	// Plain (using variables from the original frame limiter)
	static int lastFrameTime;
	int prevTime = std::exchange( lastFrameTime, time3Ms );

	// Platinum (recreated)
	static unsigned int lastPlatinumTime;
	unsigned int prevPlatinumTime = std::exchange( lastPlatinumTime, GetInvFrequency() * static_cast<double>(GetProcessTime()) * 3.0 ); 

	assert( (lastFrameTime - prevTime) == (lastPlatinumTime - prevPlatinumTime) );

	char frameTime[200];
	sprintf_s( frameTime, "Frame took %ums (normal), %ums (Platinum)\n", lastFrameTime - prevTime, lastPlatinumTime - prevPlatinumTime );
	OutputDebugStringA( frameTime );
}
```

Replicating that code allowed me to finally understand where the inaccuracies come from. Calculations are performed like this:
```cpp
do 
{
    processTime = GetProcessTime();
    T = (InvFrequency * processTime * 3.0) - LastT;
}
while ( 50 > T );
processTime = GetProcessTime();
LastT = (InvFrequency * processTime * 3.0);
```

Notice that only `processTime` is variable, the other variables are constant -- and yet, they are included in both elements of the equation, as we can effectively treat the above as:
```cpp
processTime = GetProcessTime();
T = (InvFrequency * processTime * 3.0) - (InvFrequency * lastProcessTime * 3.0);
```

When typed like this, it should become obvious how this calculation could be simplified. When rewritten like this, `T` will not be losing precision over time:
```cpp
processTime = GetProcessTime();
T = (InvFrequency * (processTime-lastProcessTime) * 3.0);
```

On paper, both expressions always give the same result; in computer science, they don't.
[I discussed this in greater detail]({{ site.baseurl }}{% post_url 2018-08-07-high-resolution-timers-and-uptime-headaches %}#floating-point-precision) in one of my older blog posts,
so here I'll just remind you that the original calculations lose precision because `InvFrequency` is a very small number, while `processTime` is a very big number. Therefore, simplifying
the expression to use `processTime-lastProcessTime` prevents inaccuracies, because the result of this subtraction (performed on integer variables, thus without losing any precision)
will always be a relatively small number, which can be expressed as a double and multiplied by `InvFrequency` without losing precision.

Does it help? Let's see:
<p align="center">
<img src="{% link assets/img/posts/spw101/rtss2.jpg %}"><br>
<em>Nice!</em>
</p>

It does! Frametimes are now smoother than ever, and they don't get any worse even if the game was to run for weeks. **Success!** With this, SilentPatch for The Wonderful 101 is ready.

## Other fixed issues

Since I wished to release SP before the game officially launches, I didn't spend much time fixing other issues. Here are the other things SilentPatch takes care of:
* The FPS limiter can now be disabled. Since the game's logic is tied to the frame rate, an uncapped game is going to speed up and be unplayable. However, it makes sense to disable the FPS
  cap if you use Vertical Sync to limit the frame rate or use an external frame limiter software, such as RTSS.
* The original game switches to the windowed mode when you press Escape. This is a horrible design and I don't know where did the development team get that idea from (maybe from emulators?),
  especially since the only way you can go back is to pause the game, go to options and change the display mode to Fullscreen or Borderless from there. This is incredibly clunky,
  so I've remapped this "feature" to Alt+Enter and also made it cycle between Windowed, Fullscreen, and Borderless modes.

# Download

Without further ado, the modification can be downloaded in *Mods & Patches*. Click here to head to the game's page directly:

<a href="{% link _games/w101.md %}#silentpatch" class="button" role="button" target="_blank">{{ site.theme_settings.download_icon }} Download SilentPatch for The Wonderful 101</a> \\
After downloading, all you need to do is to extract the archive to the game's directory and that's it! Not sure how to proceed? Check the [Setup Instructions]({% link pages/setup-instructions.md %}).

# What's next? (maybe)

This game has a plethora of different issues. The most prominent issue I've not looked into are scaling issues when the game is played at resolutions higher than 1080p. Not only the Depth of Field
does not scale properly, but also some UI elements become misplaced, as well as the camera isn't positioned correctly in some scenes. I suspect this might be the reason Wonderful 101 runs at 1080p
on both PS4 and PS4 Pro, and so I'd hope that out of all known issues this one might be addressed with an official patch. Time will tell, I guess.

That's all for now, so enjoy the game! Fixing the game before its official release is something I've never done before, so it's been an interesting experience even if not too many issues
were fixed. The best part of this project is that most of the knowledge gained here directly transfers to Metal Gear Rising, which will allow me to [pick up SP for MGR again]({% post_url 2019-06-30-mgr-save-paths %})
and make it better than I originally planned 😀

***

For those interested,
full source code of the mod has been published on GitHub, so it can be freely used as a point of reference: \\
<a href="https://github.com/CookiePLMonster/SilentPatchW101" class="button github" role="button" target="_blank">{{ site.theme_settings.github_icon }} See source on GitHub</a>