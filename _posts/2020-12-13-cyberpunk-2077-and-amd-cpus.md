---
layout: post
title: "About Cyberpunk 2077 and AMD CPUs"
excerpt: "Let's try to clarify some things and get facts straight."
thumbnail: "assets/img/posts/cp2077-amd/banner.jpg"
feature-img: "assets/img/posts/cp2077-amd/banner.jpg"
image: "assets/img/posts/cp2077-amd/banner.jpg"
date: 2020-12-13 19:10:00 +0100
tags: [Articles]
twitter: {card: "summary_large_image"}
---

**{{ "2020-12-19" | date: page.date-format | upcase }} UPDATE:**\\
Today, CD Projekt RED released a patch 1.05, which includes a fix for the issue covered in this post:

> [AMD SMT] Optimized default core/thread utilization for 4-core and 6-core AMD Ryzen™ processors.
> 8-core, 12-core and 16-core processors remain unchanged and behaving as intended.
> This change was implemented in cooperation with AMD and based on tests on both sides
> indicating that performance improvement occurs only on CPUs with 6 cores and less.

This aligns with the findings of the community regarding the range of CPUs benefiting from this change!

***

As we all know, Cyberpunk 2077 had a rocky start, but it's not unexpected with a game of this scope.
It came to my attention that it's been investigated that the game doesn't take advantage of [SMT](https://en.wikipedia.org/wiki/Simultaneous_multithreading) 
on AMD CPUs and that there's a fix for it. While the discovery was legitimate and interesting,
it was quickly followed by misinformation and strong statements. In this post,
I'll summarize my findings previously scattered around Reddit and **try** to clarify some of the statements.

**DISCLAIMER: I tried to present those findings the best I could. However, it is not possible to be 100% certain about the choices made
by the dev team nor whether the fix is universally beneficial. You may rely on profiling data gathered from the community,
but only in-depth profiling of the game's code can provide an indisputable answer. This is not something we can do.**

***

First of all, credits for finding this go to [UnhingedDoork](https://www.reddit.com/user/UnhingedDoork/)
(aka [BS_BlackScout](https://www.reddit.com/user/BS_BlackScout/)).
He dived into the game's binary and found an interesting piece of code, checking via `cpuid` for the CPU vendor and some other
attributes (later revealed to be CPU family). That's how this function looks in pseudocode (variables omitted for readability):

```c
int sub_142A82270()
{
  sub_142A822F0(&v17, &v16);
  cpuid(0);
  v13 = _RDX;
  v14 = _RCX;
  Str1 = _RBX;
  if ( strcmp(&Str1, "AuthenticAMD") != 0 )
    return v16;

  cpuid(1);
  v10 = (_RAX >> 8) & 0xF;
  if ( v10 == 15 )
    v10 = (_RAX >> 20) + 15;
  result = v17;
  if ( v10 == 21 )
    result = v16;
  return result;
}
```

The proposed fix neutralizes the `strcmp` check, making all CPUs take the `return v16` code path, previously taken only by Intel.
This very simple change reportedly can result in huge performance boosts on some configurations -- while I haven't been able to verify it myself,
reports came from various places and they generally seem great, with some people getting moderate gains while others getting huge speedups.
Do note, however, that there have also been reports of no performance increase, so this change should not be considered a "silver bullet for performance".

Sadly, this great find quickly sparkled incorrect conclusions. I first learned of this issue from a topic named

> *Cyberpunk 2077 used an Intel C++ compiler which hinders optimizations if run on non-Intel CPUs*

which naturally caused people to come up with conspiracy theories. However, knowing that the Visual Studio compiler is pretty much an industry-standard
I'd be very surprised if this was true, so I dived and checked what this code truly is.

I originally posted my findings [on Reddit](https://www.reddit.com/r/pcgaming/comments/kbsywg/cyberpunk_2077_used_an_intel_c_compiler_which/gfknein/?utm_source=reddit&utm_medium=web2x&context=3),
and they are as follows:
* This check doesn't come from ICC, but from [GPUOpen](https://github.com/GPUOpen-LibrariesAndSDKs/cpu-core-counts/blob/master/windows/ThreadCount-Win7.cpp#L69), a library from AMD which is also mentioned
  in the game's credits. Comparing the game's function with the linked one reveals they are identical. There is no evidence that Cyberpunk uses ICC,
  furthermore according to VirusTotal Visual Studio 2015 compiler has been used.
* With this knowledge, we can see that on non-Bulldozer AMD CPUs the function returns the number of physical cores, while for Intel and AMD Bulldozer chips it returns the number of logical cores.
  With this change in place, the function returns the number of logical cores regardless of the CPU vendor.
* This function is used in numerous places, but most notably it is used in what appears to be an initialization function of the game's job dispatcher system.
  This value seems to drive a `MaxHWConcurrency` attribute, so in my opinion, it may be used e.g. to decide how many worker threads to spawn. This would at least explain higher CPU usage,
  since this change most likely gives the game's job system twice the amount of threads, thus distributing the work across logical cores more evenly.

Why was it done? I don't know, since it comes from GPUOpen I don't think this check is "wrong" per se, but maybe it should not have been used in Cyberpunk due to the way it utilizes threads.
Even the comment in this code snippet advises caution, after all. Again, **we can't know for certain**, but in my opinion, this decision could have been made due to any of the following:
* At some point, this change resulted in a performance boost but later changes and/or optimization may have made it less effective.
* Reportedly, this change helps AMD CPUs with one CCX unit and can make the game slower on CPUs with multiple CCX units. This code may have been tailored to only one type of CPU
  and not the other.

Lastly, I'd just like to highlight what [AMD says about this GPUOpen functionality](https://gpuopen.com/learn/cpu-core-count-detection-windows/) (emphasis mine):

> Our sample code, linked below, errs on the side of caution for our Ryzen processors and encourages you to profile: the `getDefaultThreadCount()` function draws attention to that fact, 
> **returning a starting default count equal to the number of physical processor cores on Ryzen.**


# I want to test this!

It goes without saying that I cannot guarantee any improvements with this change applied, but you're welcome to experiment.
Again, the original author of those instructions is [UnhingedDoork](https://www.reddit.com/r/Amd/comments/kbp0np/cyberpunk_2077_seems_to_ignore_smt_and_mostly/gfkmc7d/?context=3)
and I am only mirroring them here. Since I don't have an AMD CPU, I cannot test those:

1. Open the EXE with HxD (Hex Editor).
2. Look for a `75 30 33 C9 B8 01 00 00 00 0F A2 8B C8 C1 F9 08` hex string.
3. Change it to `EB 30 33 C9 B8 01 00 00 00 0F A2 8B C8 C1 F9 08`.

Alternately, you may try a [PerformanceOverhaulCyberpunk](https://github.com/yamashi/PerformanceOverhaulCyberpunk) mod which applies those changes, similarly to my ASI-based mods.
Again, I have not tested this, so I cannot guarantee any improvements. Use with caution.

***

*Usually, I don't feel like I have to say this, but in this case, I do -- please, stay kind and don't criticize specific people for the state of the game.*
*You might be mad, but anger isn't helping anyone. This post is also meant to be purely informative and I am not the right person to judge if this issue is*
*caused by a deliberate choice or an oversight. Be patient and enjoy the game if you are currently playing it!*
