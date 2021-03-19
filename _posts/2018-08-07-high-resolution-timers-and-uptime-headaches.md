---
layout: post
title: "High resolution timers and high uptime headaches"
excerpt: You may be dealing with very big numbers, so tread carefully.
image: "assets/img/posts/uptime-img.png"
date: 2018-08-07 2:25:00 +0200
tags: [Articles]
---

Today's task of upgrading the compiler used for an old game from Visual Studio 2003 to Visual Studio 2017 began pretty well.
There are plenty of changes regarding C/C++ standard conformance between those, but half an hour was enough to reliably replace all problematic code parts.
With everything adapted and code compiling in VS2017 just fine, I tested it, and it worked beautifully -- fast and without crashes.

So I sent the EXE off to have it checked by somebody else, and after I while I asked:

> --- _Is it working OK? Worked great for me so I think we can ship it._\\
> --- _Nope, it's super slow, like 5FPS!_\\
> --- _Seriously?_\\
> --- _Yep -- it's just like [the last game]({{ site.baseurl }}{% post_url 2018-06-15-slrr-proof-of-fix %})._

How is it possible? It was literally impossible for a newer compiler to generate "unoptimized" code which performs so badly for one person,
and perfectly well for another. More info arrived soon:

> --- _Hey, it works bad on my Windows 10, but I tried it on a XP SP3 virtual machine and it runs great!_

At that point, I started rolling back the project to use older compiler variations, starting with VS2015. Results were identical for VS2015 and VS2012
(for whatever reason, VS2013 standard library implementation conflicted with something in game's code, so I just gave up), but when compiled in VS2010,
it worked correctly!

[Sadly, it didn't appear to be a common problem, so the issue had to be in the game code itself.](https://www.google.com/search?q=Code+slower+after+upgrading+to+Visual+Studio+2012&oq=Code+slower+after+upgrading+to+Visual+Studio+2012)

Time to switch PCs and test it on a different machine. Another PC also runs Windows 10 and has a bit older hardware, but it still can handle a game from 2005 just fine. I launch the game and...\\
It stutters! Not only intro movies stopped working, but the entire game was **insanely slow**! I also had exact same results as reported -- executable compiled via VS2003 or VS2010 worked fine,
anything newer had unacceptable performance.

Before attempting to debug it further, I figured it makes sense to switch to Windows 7 and test it there. Turns out, all executables work fine.

Switching back to Windows 10 with an intent to start debugging it properly and... **it works fine again**! While it may not seem like it's obvious, I instantly recalled an identical issue from the past,
but I first asked again:

> --- _When did you last reboot your PC?_\\
> --- _Probably months ago, as I am using hibernation. Why do you ask?_\\
> --- ...

***

Those following SilentPatch for San Andreas might be able to recall a very similar bug report from early 2017. It stated that dancing minigames could start to desync and stutter if PC
had approximately over 12 days of uptime. To verify that, I faked years of uptime -- that's how dancing looks (without SilentPatch) with a fake uptime of 12 years:

<div style='position:relative;padding-bottom:57%'><iframe src='https://gfycat.com/ifr/BlushingImprobableChinesecrocodilelizard' frameborder='0' scrolling='no' width='100%' height='100%' style='position:absolute;top:0;left:0;' allowfullscreen></iframe></div>
<p align="center">
<em>For those unfamiliar with GTA San Andreas - arrows shown in the bottom part of the screen are supposed to move smoothly.</em>
</p>


That looks almost identical to what was observed with the game! In the case of GTA, this stuttering is in fact loss of precision -- queried time was converted to a floating point value,
calculations were performed and then the result was converted back to a 64-bit integer value.

Is it similar in the case of the analyzed game? This code snippet has an answer:
```cpp
float Timer::GetTime()
{
    LARGE_INTEGER timer_act;
    QueryPerformanceCounter (&timer_act);
    return ((float) ((float) timer_act.QuadPart - (float) timer_begin.QuadPart) / (float) timer_freq.QuadPart)*1000.0f;
}
```

That is... terrible. In order to understand why, some things need to be explained.

QueryPerformanceCounter
---------------------------------
MSDN defines [QueryPerformanceCounter](https://msdn.microsoft.com/en-us/library/ms644904(v=vs.85).aspx) like this:

> Retrieves the current value of the performance counter, which is a high resolution (<1us) time stamp that can be used for time-interval measurements.
> [...]
> QueryPerformanceCounter reads the performance counter and returns the total number of ticks that have occurred since the Windows operating system was started,
> including the time when the machine was in a sleep state such as standby, hibernate, or connected standby.

QPC does not have any specific units, therefore it can be assumed it counts in units relative to CPU cycles.
For a programmer this means just one thing -- this value is going to increase **very** fast.
This shouldn't be a concern though, since a value returned by [QueryPerformanceFrequency](https://msdn.microsoft.com/en-us/library/ms644905(v=vs.85).aspx) can be used
to convert this value to seconds or milliseconds.

The last bit of quoted info is crucial though -- this timer (as well as several other WinAPI timers) count time from the time Windows was started (thus it depends on uptime).
Naturally, if PC is not restarted for a long time, those values are going to be very high.

This does not explain the issue fully yet, though -- after all, why wasn't this a noticeable problem back in 2005?

Fast Startup
------------
Historically, QPC values often were relatively small, as people usually shut down their PC's at night, and therefore uptime values stayed low.
However, with the introduction of [Fast Startup](https://blogs.msdn.microsoft.com/olivnie/2012/12/14/windows-8-fast-boot/) in Windows 8, uptime stopped resetting after shutting down the PC (since it's now effectively a partial hibernation).
This led to uptimes inflating noticeably for most people, as with Fast Startup enabled it resets only on a full PC reboot.

What it means for us is that code which could have worked great in 2005 can break in 2018, unless you reboot your PC!
By doing so, timers are reset and they are back to values which were common a decade ago.

We are almost here, but we also need to understand why calculations quoted earlier start breaking over time.

Floating point precision
------------------------
Naturally, any data type can store a limited range of data -- that also concerns floating points.
In the case of QPC, values are stored in a signed 64-bit integer, so maximum theoretical value it can hold without overflowing is 9,223,372,036,854,775,807.
No matter how fast your CPU is, you are **not** hitting this uptime value anytime soon.

Things are not looking as good for floating point values though. Let's borrow a small table from Wikipedia and see how many significant digits can floating point numbers have:

| Type   | Number of decimal digits |
| ------ | ------------------------ |
| Single | ~7.2                     |
| Double | ~15.9                    |

As evidenced we can't even use floating points to accurately represent a 8-digit long value. Means values like `1234567.0` or `12.34567` are approximately the most precise values
we can accurately represent. Anything bigger or smaller will start losing precision -- that is, a value of `123456789` will be rounded to something close to `123456800.0`!

Concluding information
------------------------
Now scroll back to the code snippet posted earlier -- it should now be obvious why this routine is so bad! Two potentially huge values are converted to floats
(potentially losing a lot of precision), subtracted, divided by another huge value converted to a float (losing even more precision), then multiplied again (once again potentially losing precision)...

A proper way to handle this is to keep those calculations as integers and only convert the result (which is a relative time value, which should be manageably small) to a float, like so:
```cpp
timer_freq.QuadPart /= 1000; // To convert frequency from 'ticks per second' to 'ticks per millisecond'
return static_cast<float>( (timer_act.QuadPart - timer_begin.QuadPart) / timer_freq.QuadPart );
```

With similar changes applied to the code, even months worth of uptime are not going to cause any problems!
However, we need a way to reliably test it without having to leave PC running for weeks. And so for this, I have created an utility.

***

Uptime Faker
============

Since issues related to high uptime are so common, why not fake high uptime so we can detect issues quickly?
In fact, that's **exactly** what checked builds of Windows do (adding ~49 days of uptime to timers), but... is there anyone using those?
Ideally, there should be a way to enable this mode for specific applications, but much to my surprise, Application Verifier doesn't have such an option
(all it can do is forcibly wrap around GetTickCount, which is not what we need here). Time to write a custom solution then!

**Uptime Faker** is a generic plugin hooking into WinAPI timer functions and adding a specific amount of days to their returned value.
As a plugin based on Detours, it can be injected to a newly created process as a plugin (could even use [Ultimate ASI Loader](https://github.com/ThirteenAG/Ultimate-ASI-Loader/releases)),
or Detours utilities could be used to create a process with this DLL injected right away. Alternately, it can be loaded by the game itself, which is exactly what I did in my case.

Uptime Faker can be downloaded and previewed on GitHub: \\
<a href="https://github.com/CookiePLMonster/UptimeFaker" class="button github" role="button" target="_blank">{{ site.theme_settings.github_icon }} Check Uptime Faker on GitHub</a>

***

However, did all this effort help fixing the issue? Sure it did! Thanks to Uptime Faker I was able to reproduce all issues related to high uptime and resolve them quickly.
Now no matter how much uptime I add to the game with the utility, everything works fine! Therefore, I expect tests on a real machine tomorrow to go smoothly, too.

One unanswered question remains -- **why didn't this code break right away, even when compiled with VS2003**?
I didn't have a chance to disassemble and compare binaries built with those yet, however I assume older VS might have been less strict with obeying user's intent exactly and might have
skipped some of the typecasts, partially or completely reducing the risk of losing precision.
Modern compilers can compile floating point operations in "precise" mode (which is the default nowadays), and probably did not skip or reorder any arithmetic operations.

Bottom line is -- this code should have never worked properly, somebody just got very lucky it broke only when upgrading to a compiler from the next decade.
Who knows how many more bugs are hiding in software like this?