---
layout: post
title: "Fixing Mass Effect black blobs on modern AMD CPUs"
thumbnail: "assets/img/games/bg/mass-effect.jpg"
feature-img: "assets/img/games/bg/mass-effect.jpg"
image: "assets/img/posts/mass-effect/comparison.jpg"
excerpt: "Graphical artifacts caused by a CPU, not GPU? Sure thing."
twitter: {card: "summary_large_image"}
date: 2020-07-19 13:00:00 +0200
tags: [Releases, Articles]
---

*TL;DR - if you are not interested in an in-depth overview of what was wrong with the game and how it was fixed,
scroll down to [**Download**](#download) section for a download link.*

***

**{{ "2020-11-07" | date: page.date-format | upcase }} UPDATE:**\\
Build 2 has been released! This update resolves a bug introduced by SilentPatchME which broke enemy AI during the Virmire mission.

***

* [Introduction](#introduction)
* [Part 1 -- Research](#part-1)
    * [Prelude](#prelude)
    * [PIX](#pix)
* [Part 2 -- A closer look into D3DX](#part-2)
* [Part 3 -- Standalone tests](#part-3)
* [Part 4 -- Putting it all together](#part-4)
    * [Download](#download)

***

# Introduction {#introduction}

**Mass Effect** is a popular franchise of sci-fi roleplaying games. The first game was initially released by BioWare in late 2007 on Xbox 360 exclusively as a part of a publishing deal with Microsoft.
A few months later in mid-2008, the game received PC port developed by Demiurge Studios. It was a decent port with no obvious flaws, that is until 2011 when AMD released their new Bulldozer-based CPUs.
When playing the game on PCs with modern AMD processors, two areas in the game (Noveria and Ilos) show severe graphical artifacts:

<p align="center">
<img src="{% link assets/img/posts/mass-effect/me1-blobs.jpg %}"><br>
<em>Well, that doesn't look nice.</em>
</p>

While not unplayable, it's definitely distracting. Thankfully, workarounds exist -- such as
[disabling lighting via console commands](http://abesmissioncontrol.blogspot.com/2015/04/mass-effect-fixing-blocky-player-models.html)
or [modifying the game's maps to remove broken lights](https://www.nexusmods.com/masseffect/mods/181), but seemingly the issue has never been fully understood.
Some sources claim that an FPS Counter mod can also fix that issue, but I couldn't find much information about it and the mod's sources don't seem to be available online,
and there is no documentation on how the mod tackles this error.

What makes this issue particularly interesting? Vendor-specific bugs are nothing new, and games have had them for decades. However, to my best knowledge, this is the only case where a graphical
issue is caused by a **processor** and not by a graphics card. In the majority of cases, issues happen with a specific vendor of GPU and they don't care about the CPU, while in this case, it's the exact opposite.
This makes the issue very unique and worth looking into.

Looking up existing discussions online, this issue seems to affect AMD FX and Ryzen chips. Compared to the older AMD chips, these lack a [3DNow! instruction set](https://en.wikipedia.org/wiki/3DNow!).
Unrelated or not, the community consensus was that this was the cause of the bug and that the game tried to use those instructions upon detecting an AMD CPU.
Given that there are no known cases of this bug occurring on Intel CPUs and that 3DNow! instructions were exclusive to AMD, it's no surprise the community assumed that this is the issue.

Is this really the issue, or is it caused by something entirely different? Let's find out!

# Part 1 -- Research {#part-1}

## Prelude
Even though the issue is trivial to reproduce, I couldn't look into it for the longest time for a simple reason -- I don't have access to any PCs with AMD hardware!
Thankfully, this time I'm not approaching research alone -- [Rafael Rivera](https://withinrafael.com/) got my back during the entire process of R&D,
providing a test environment with an AMD chip, insights, ideas as well as putting up with hundreds of blind guesses I usually throw around when trying to find the way to the root of such unknown problems.

Since we now had a good testing environment, the first theory to test was of course `cpuid` -- if people are right in assuming that 3DNow! instructions are to blame, there should a place in the game's code
where they check for their presence, or at the very least check for the CPU vendor. That reasoning is flawed, though; if it was true that the game attempts to use 3DNow! instructions any time it runs on an AMD chip,
without checking if they are supported, the game would most likely crash when trying to execute an illegal instruction. Moreover, a quick scan around the game's code reveals that the game **doesn't**
check for CPU capabilities. Therefore, whatever is up with this issue, it doesn't appear to be caused by the game mis-detecting CPU features, because it seemingly doesn't care about them in the first place.

When this started looking like an undebuggable case, Rafael came back to me with a realization -- disabling **PSGP** (Processor Specific Graphics Pipeline) fixes the issue and the characters are properly lit!
PSGP is not the best documented term, but in short, it's a legacy (concerning only older DirectX versions) feature allowing Direct3D to perform processor-specific optimizations:

> In previous versions of DirectX, there was a path that allowed to do vertex processing called the PSGP. Applications had to take this path into account and support a path for vertex processing
> on the processor and graphics cores.

Putting it this way, it makes sense why disabling PSGP fixes artifacts on AMD -- the path taken by modern AMD processors may be somehow broken.
How to disable it? Two ways come to mind:
* It is possible to pass a `D3DCREATE_DISABLE_PSGP_THREADING` flag to `IDirect3D9::CreateDevice`. It's defined as: \\
  > Restrict computation to the main application thread. If the flag is not set, the runtime may perform software vertex processing and other computations in worker thread
  > to improve performance on multi-processor systems.

  Sadly, setting that flag doesn't fix the issue. Looks like, despite the flag having "PSGP" in name, it's not what we are looking for.
* DirectX specifies two registry entries to disable PSGP in D3D and to disable PSGP only for D3DX -- `DisablePSGP` and `DisableD3DXPSGP`. Those flags can be set system-wide or process-wide.
  For information on how to set them only for a specific process, see [Rafael Rivera's guide on enabling application-specific Direct3D flags](https://withinrafael.com/2020/07/11/specify-application-specific-direct3d-flags/).

`DisableD3DXPSGP` appears to be a viable fix for that issue. Therefore, if you have an aversion towards downloading third party fixes/modifications or you must fix this issue without making
any changes to the game, it's a perfectly fine way of doing it. As long as you set that flag only for Mass Effect and not system-wide, it's fine!

## PIX
As always with graphical issues, PIX is likely the most useful tool one could use to diagnose them. We captured similar scenes from Intel and AMD hardware and compared the results.
One difference was instantly noticeable -- unlike with my past projects, where [captures did not carry the bug with them]({{ site.baseurl }}{% post_url 2018-07-07-farcry-d3d9-bug %}) and the same capture
would look different on different PCs (indicating a driver or d3d9.dll bug), these captures carry the bug with them! In other words, a capture from an AMD hardware opened on a PC with Intel hardware
**does** show the bug.

An AMD capture on Intel looks no different than on the hardware it was taken from:

<p align="center">
<img src="{% link assets/img/posts/mass-effect/me1-pix1.jpg %}">
</p>

What does this tell us?
* Since PIX does not "take screenshots" but instead captures the sequence of D3D commands and executes them on hardware, we can observe that executing the commands captured from an AMD box
  results in the same bug when executed on Intel.
* This strongly implies that the difference is not caused by the difference in **how** the commands are executed (that's how you get GPU specific bugs), but **what** commands are executed.

In other words, it's almost certainly not any sort of a driver bug. Instead, the way inputs for the GPU are prepared seems to be somehow broken[^1]. That is indeed a very rare occurrence!

At this point, finding the bug is a matter of finding any jarring differences between captures. It's tedious, but that's the only viable way.

After a long while spent poking the capture, a full body draw call caught my attention:

<p align="center">
<img src="{% link assets/img/posts/mass-effect/me1-pix2.jpg %}">
</p>

On an Intel capture, this draw outputs most of the character's body, together with lighting and textures. On an AMD capture, it outputs a plain black model. This looks like a good trail.

The first obvious candidate for checking would be bound textures, but they seem to be fine and are consistent across captures.
However, some of the pixel shader constants looked weird. Not only do they have NaNs (Not a Number), but they also seem to only appear on the AMD capture and not the Intel capture:

<p align="center">
<img src="{% link assets/img/posts/mass-effect/me1-pix3.jpg %}"><br>
<em>1.#QO indicates a NaN</em>
</p>

This looks promising -- NaN values causing strange visuals are not unheard of. Funnily enough, a PlayStation 3 version of Mass Effect 2
[had a very similar looking issue in RPCS3](https://github.com/RPCS3/rpcs3/issues/7397) which was also related to NaNs!

However, before we get too excited, those values could just be leftovers from previous draws and they might end up being unused for the current draw.
Luckily, in this case it's clearly visible that those NaNs get submitted to D3D for this specific draw...

```
49652	IDirect3DDevice9::SetVertexShaderConstantF(230, 0x3017FC90, 4)
49653	IDirect3DDevice9::SetVertexShaderConstantF(234, 0x3017FCD0, 3)
49654	IDirect3DDevice9::SetPixelShaderConstantF(10, 0x3017F9D4, 1) // Submits constant c10
49655	IDirect3DDevice9::SetPixelShaderConstantF(11, 0x3017F9C4, 1) // Submits constant c11
49656	IDirect3DDevice9::SetRenderState(D3DRS_FILLMODE, D3DFILL_SOLID)
49657	IDirect3DDevice9::SetRenderState(D3DRS_CULLMODE, D3DCULL_CW)
49658	IDirect3DDevice9::SetRenderState(D3DRS_DEPTHBIAS, 0.000f)
49659	IDirect3DDevice9::SetRenderState(D3DRS_SLOPESCALEDEPTHBIAS, 0.000f)
49660	IDirect3DDevice9::TestCooperativeLevel()
49661	IDirect3DDevice9::SetIndices(0x296A5770)
49662	IDirect3DDevice9::DrawIndexedPrimitive(D3DPT_TRIANGLELIST, 0, 0, 2225, 0, 3484) // Draws the character model
```

...and the pixel shader used for this draw references both constants:
```
// Registers:
//
//   Name                     Reg   Size
//   ------------------------ ----- ----
//   UpperSkyColor            c10      1
//   LowerSkyColor            c11      1
```

Both constants appear to [come straight from Unreal Engine](https://github.com/abaelhe/unrealengine-old/search?q=UpperSkyColor) and judging by the name,
they might directly influence the lighting. Bingo!

A quick in-game test further confirms the theory -- on an Intel machine, a vector of 4 NaN values was never submitted as pixel shader constants;
meanwhile, on an AMD machine, NaNs would start showing up as soon as the player entered the area where lighting breaks!

Does it mean work is done? No, far from it, as finding broken constants is only half of the success. The question remains, where do they come from, and can they be replaced?
An in-game test replacing NaN values with zeros partially fixed the issue -- ugly black blobs disappeared, but characters were still way too dark:

<p align="center">
<img src="{% link assets/img/posts/mass-effect/me1-dark-lighting.jpg %}"><br>
<em>Almost correct... but not quite.</em>
</p>

Given how important these light values might be for the scene, it's not feasible to settle with a workaround like this. We know we are on the right track though!

Sadly, any attempt to track down the origin of these constants pointed towards something resembling a render thread and not the real place of submission.
While not undebuggable, it's clear that we needed to try a fresh approach before potentially spending an infinite amount of time following the data flow between game-specific
and/or UE3-specific structures.

[^1]: In theory, this could also be a bug inside d3d9.dll, which would complicate things a bit. Thankfully, it wasn't the case.

# Part 2 -- A closer look into D3DX {#part-2}

Taking a step back, we realized that we overlooked something earlier on. Recall that to "fix" the issue, one of two registry entries had to be added -- `DisablePSGP` and `DisableD3DXPSGP`.
Assuming their naming is not misleading, then `DisableD3DXPSGP` should be a subset of `DisablePSGP`, with the former disabling PSGP in D3DX only, and the latter disabling it in both D3DX and D3D.
With this assumption, we turned our sights to D3DX.

Mass Effect imports a set of D3DX functions by linking against `d3dx9_31.dll`:
```
D3DXUVAtlasCreate
D3DXMatrixInverse
D3DXWeldVertices
D3DXSimplifyMesh
D3DXDebugMute
D3DXCleanMesh
D3DXDisassembleShader
D3DXCompileShader
D3DXAssembleShader
D3DXLoadSurfaceFromMemory
D3DXPreprocessShader
D3DXCreateMesh
```

Looking at the list, if I approached it without prior knowledge gained from the captures I would have expected `D3DXPreprocessShader` or `D3DXCompileShader` to be possible culprits -- shaders
could be wrongly optimized and/or compiled on AMD, but fixing that could be insanely challenging.

However, with our current knowledge one function stands out from this list -- `D3DXMatrixInverse` is the only function that could reasonably be used to prepare pixel shader constants.

The function is called from only one place in the game:

```cpp
int __thiscall InvertMatrix(void *this, int a2)
{
  D3DXMatrixInverse(a2, 0, this);
  return a2;
}
```

It's... not too well made, though. A quick peek inside `d3dx9_31.dll` reveals that `D3DXMatrixInverse` does not touch the output parameters and returns `nullptr`
if matrix inversion fails (due to the input matrix being singular), yet the game doesn't care about this at all. Output matrix might be left uninitialized, boo!
Inverting singular matrices does indeed happen in the game (most frequently in the main menu), but no matter what we did in an attempt to make the game handle them better
(e.g. zeroing the output or setting it to an identity matrix), visuals wouldn't change. Oh well.

With this theory debunked, we are back to PSGP -- what is PSGP doing exactly in D3DX? Rafael Rivera looked into that and the logic behind it turns out to be quite simple:

```cpp
AddFunctions(x86)
if(DisablePSGP || DisableD3DXPSGP) {
  // All optimizations turned off
} else {
  if(IsProcessorFeaturePresent(PF_3DNOW_INSTRUCTIONS_AVAILABLE)) {
    if((GetFeatureFlags() & MMX) && (GetFeatureFlags() & 3DNow!)) {
      AddFunctions(amd_mmx_3dnow)
      if(GetFeatureFlags() & Amd3DNowExtensions) {
        AddFunctions(amd3dnow_amdmmx)
      }
    }
    if(GetFeatureFlags() & SSE) {
      AddFunctions(amdsse)
    }
  } else if(IsProcessorFeaturePresent(PF_XMMI64_INSTRUCTIONS_AVAILABLE /* SSE2 */)) {
    AddFunctions(intelsse2)
  } else if(IsProcessorFeaturePresent(PF_XMMI_INSTRUCTIONS_AVAILABLE /* SSE */)) {
    AddFunctions(intelsse)
  }
}
```

Unless PSGP is disabled, D3DX picks functions optimized to make use of a specific instruction set. That makes sense and ties back to the original theory.
As it turns out, D3DX has functions optimized for AMD and 3DNow! instruction set, so the game is indirectly making use of those after all.
With 3DNow! instructions removed, modern AMD processors take the same code path as Intel processors -- that is, `intelsse2`.

To summarize:
* Disabling PSGP makes both Intel and AMD take a regular `x86` code path.
* Intel CPUs always take an `intelsse2` code path[^2].
* AMD CPUs supporting 3DNow! take a `amd_mmx_3dnow` or `amd3dnow_amdmmx` code path, while CPUs without 3DNow take an `intelsse2` code path.

With this information, we put forward a hypothesis -- _something is possibly wrong with AMD SSE2 instructions, and the results of matrix inversion calculated on AMD with an `intelsse2` path
are either too inaccurate or completely incorrect._

How do we verify that hypothesis? By tests, of course!

P.S.: You may be thinking -- "well, the game uses `d3dx9_31.dll` but the newest D3DX9 library is `d3dx9_43.dll`, surely that must be fixed in later revisions??".
We tried to verify that by "upgrading" the game to link against the newest DLL -- and nothing changed.

[^2]: Of course, assuming they have an SSE2 instruction set, but any Intel CPU not having those instructions falls below the minimum system requirements of Mass Effect.

# Part 3 -- Standalone tests {#part-3}

We prepared a simple, standalone program to verify the precision of matrix inversions. During a short game session in the "bugged" game area, we recorded every input and output
of `D3DXMatrixInverse` to a file. Later, this file was read by a standalone test program and the results were recalculated again. To verify correctness, outputs from the game were then compared
with outputs calculated by the test program.

After several attempts basing on data collected from Intel and AMD chips and with PSGP enabled/disabled, we cross-checked the results between machines.
The results are as follows, ✔️ indicating success (results were equal) and ❌ indicating failure (results were not equal). The last column indicates whether the game handles this data fine
or if it glitches out. We deliberately did not take imprecision of floating-point maths into the account and instead compared the results with `memcmp`:

| Data source    | Intel SSE2 | AMD SSE2 | Intel x86 | AMD x86 | Accepted by game? |
| -------------: | ---------- | -------- | --------- | ------- | ----------------- |
| **Intel SSE2** | ✔️ | ❌ | ❌ | ❌ | ✔️ |
| **AMD SSE2**   | ❌ | ✔️ | ❌ | ❌ | ❌ |
| **Intel x86**  | ❌ | ❌ | ✔️ | ✔️ | ✔️ |
| **AMD x86**    | ❌ | ❌ | ✔️ | ✔️ | ✔️ |

<p align="center">
<em>Tests results for D3DXMatrixInverse</em>
</p>

Interesting -- the results show that:
* Calculations with SSE2 do not transfer across Intel and AMD machines.
* Calculations without SSE2 **do** transfer across machines.
* Calculations without SSE2 are "accepted" by the game despite not being identical to the ones from Intel SSE2.

This raises a question -- what exactly is wrong with calculations from AMD SSE2 so they end up glitching the game?
We don't have a precise answer for that, but it seems to be a product of two factors:
* SSE2 implementation of `D3DXMatrixInverse` might be poor numerically -- seems like some SSE2 instructions give different results on Intel/AMD (possibly different rounding modes),
  and the function is not written in a way which would help mitigate the inaccuracies.
* The game's code is written in a way that is too sensitive to accuracy issues.

At this point, we were ready to put forward a fix which would replace `D3DXMatrixInverse` with a rewrite of an x86 variation of the D3DX function and call it a day.
However, before proceeding I had one more random idea -- D3DX is deprecated and got replaced with [DirectXMath](https://docs.microsoft.com/en-us/windows/win32/dxmath/directxmath-portal).
I figured that since we were to replace that matrix function anyway, I could try replacing it with `XMMatrixInverse` being a "modern" replacement for `D3DXMatrixInverse`.
`XMMatrixInverse` also uses SSE2 instructions so it should be equally optimal to the D3DX function, but I was nearly sure it would break the same way.

I hacked it together quickly, sent it off to Rafael and...

**It works fine!?**

What we were sure to be an issue coming from tiny differences in SSE2 instructions may have been a purely numerical issue after all. Despite also using SSE2, `XMMatrixInverse` gave perfect results
on both Intel and AMD. Therefore, we re-ran the same tests and the results were surprising, to say the least:

| Data source    | Intel | AMD | Accepted by game? |
| -------------: | ----- | --- | ----------------- |
| **Intel** | ✔️ | ✔️ | ✔️ |
| **AMD**   | ✔️ | ✔️ | ✔️ |

<p align="center">
<em>Tests results for XMMatrixInverse</em>
</p>

Not only the game works fine, but results are perfectly identical and transfer across machines!

With this in mind, we revised the theory behind that bug -- without a doubt, the game is at fault for being too sensitive to issues, but with additional tests, it seems like
D3DX may have been written with fast math in mind, while DirectXMath may care more about precise calculations. This makes sense -- D3DX is a product of the 2000s and it is perfectly reasonable
that it was written with performance being the main priority. DirectXMath has the "luxury" of being engineered later, so it could put more attention towards precise, deterministic computations.

# Part 4 -- Putting it all together {#part-4}

It took a while to get here, so I hope you're still not bored to death. To summarize, that's what we went through:
* We verified that the game does not use 3DNow! instructions directly (only the system DLLs do).
* We found out that disabling PSGP fixes the issue on AMD processors.
* Using PIX, we found the culprit -- NaN values in pixel shader constants.
* We nailed down the origin of those values to `D3DXMatrixInverse`.
* We fuzzed that function and found out that it does not give consistent results between Intel and AMD CPUs when SSE2 instructions are used.
* We accidentally found out that `XMMatrixInverse` does not have this flaw and is a viable replacement.

The only thing that's left is to implement a proper replacement! That's where **SilentPatch for Mass Effect** appears.
We have decided that the cleanest way to fix this issue is to provide a replacement `d3dx9_31.dll`, which forwards every function exported by Mass Effect
to the system DLL, except for `D3DXMatrixInverse`. For this function, we have developed a replacement using `XMMatrixInverse`.

A replacement DLL makes for a very clean and bulletproof installation, and it's been confirmed to work fine with both Origin and Steam versions of the game.
It works out of the box, without the need for an ASI Loader or any other third party software.

To our best knowledge, the game now looks exactly how it should, without any downgrades in the lighting:

<div align="center" class="video-container">
<iframe frameborder="0" class="juxtapose" width="100%" height="1080" src="https://cdn.knightlab.com/libs/juxtapose/latest/embed/index.html?uid=fd1b56b0-c9a7-11ea-bf88-a15b6c7adf9a"></iframe>
</div>
<p align="center">
<em>Noveria</em>
</p>

<div align="center" class="video-container">
<iframe frameborder="0" class="juxtapose" width="100%" height="1080" src="https://cdn.knightlab.com/libs/juxtapose/latest/embed/index.html?uid=1630ab78-c9a8-11ea-bf88-a15b6c7adf9a"></iframe>
</div>
<p align="center">
<em>Ilos</em>
</p>

## Download

The modification can be downloaded in *Mods & Patches*. Click here to head to the game's page directly:

<a href="{% link _games/mass-effect.md %}#silentpatch" class="button" role="button" target="_blank">{{ site.theme_settings.download_icon }} Download SilentPatch for Mass Effect</a> \\
After downloading, all you need to do is to extract the archive to the game's directory and that's it! Not sure how to proceed? Check the [Setup Instructions]({% link pages/setup-instructions.md %}).

***

For those interested,
full source code of the mod has been published on GitHub, so it can be freely used as a point of reference: \\
<a href="https://github.com/CookiePLMonster/SilentPatchME" class="button github" role="button" target="_blank">{{ site.theme_settings.github_icon }} See source on GitHub</a>
