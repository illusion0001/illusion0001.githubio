---
layout: post
title: "Frame Pacing Fix for Crysis Remastered (PS4)"
excerpt: "Resolving inconsistent frame delivery on Crysis Remastered."
categories: patches
thumbnail: "assets/images/CrysisRemasteredFramePacingFix/CrysisRemasteredFramePacingFix_Banner.png"
feature-img: "assets/images/CrysisRemasteredFramePacingFix/CrysisRemasteredFramePacingFix_Banner.png"
image: "https://gs2-sec.ww.prod.dl.playstation.net/gs2-sec/appkgo/prod/CUSA18659_00/6/i_090b53a1ed8854fe34a2b4f986c4b524d543275d140eb81ab6df0fe47c7087da/i/pic0.png"
tags: [Articles, Releases]
twitter: {card: "summary_large_image"}
---

* TOC
{:toc}

# Intro
Most of crytek releases utilizing their cryengine has this issue where the frames aren't being consistently delivered resulting in "bad frame pacing". In a 30FPS game. New frames should be sent to the display every other frame.

```
New frame
Duplicate frame
New frame
Duplicate frame
```

This is what consistent frame pacing should be for a 30 FPS target.

Here's what bad frame pacing looks like.

```
New frame
Duplicate frame
Duplicate frame
New frame
Duplicate frame
New frame
New frame
Duplicate frame
```

As you can probably tell this is not consistent for a 30 FPS update. Can we solve it? Yes! Atbeit with a catch. Input latency will be increased.

## Part 1 - Researching Framerate limit

Searching for the `fps` string comes with with a few results.

Notable one being `sys_MaxFPS` with the description `Limits the frame rate to specified number (int)`

```
FUN_0210f510:02111cc1(*),
FUN_021175a0:0211774c(*),
FUN_0211c640:0211c6bc(*),
FUN_0230ee70:02315821(*)  
```

Peeking on the reference, the last one caught my eye.

```
02315821 48 8d 35        LEA        RSI,[s_sys_MaxFPS_02ca3a03]
         db e1 98 00
02315828 48 8d 15        LEA        RDX,[DAT_04632bf4]
         c5 d3 31 02
0231582f 4c 8d 0d        LEA        R9,[s_Limits_the_frame_rate_to_specifi_02ca3a0e]
         d8 e1 98 00
```

`0x4632bf4` Some testing reveals that this seem to be where the value gets stored.

## Part 2 - Implementing a better 30 FPS Limit

We can write 1 to ESI which `sceVideoOutSetFlipRate` will accept as a parameter.

```
0x0 16.67ms -- 60hz
0x1 33.33ms -- 30hz
0x2 50.00ms -- 20hz
```

```
01fadf4a 31 f6           XOR        ESI,ESI
01fadf4c 41 8b bf        MOV        EDI,dword ptr [R15 + 0x128]
         28 01 00 00
01fadf53 e8 f8 a7        CALL       sceVideoOutSetFlipRate
         bc 00
...
01fadfc4 cd 41           INT        0x41
01fadfc6 e9 58 ff        JMP        LAB_01fadf23
         ff ff
```

Previous code does `XOR ESI,ESI` which just writes 0 to ESI. 

Writing 1 to ESI requires some code cave.

```
01fadf4a eb 78           JMP        LAB_01fadfc4
01fadf4c 41 8b bf        MOV        EDI,dword ptr [R15 + 0x128]
         28 01 00 00
01fadf53 e8 f8 a7        CALL       sceVideoOutSetFlipRate
         bc 00
...
01fadfc4 be 01 00        MOV        ESI,0x1
         00 00
01fadfc9 eb 81           JMP        LAB_01fadf4c

```

```
JMP        LAB_01fadfc4
...
MOV        ESI,0x1
JMP        LAB_01fadf4c
```

First instruction jumps to `0x01fadfc4` then write 0x1 to ESI and then jumps back to `01fadf4c` to contiune the code flow.

We will have to write 0 to sys_maxfps to remove the existing cap entirely.

```
0210e698 48 e8 1b        CALL       SUB_02c818b9
         32 b7 00
...
02c818b9 8b 05 65        MOV        EAX,dword ptr [DAT_03b98424]
         6b f1 00
02c818bf c6 05 2e        MOV        byte ptr [DAT_04632bf4],0x3c
         13 9b 01 3c
02c818c6 c3              RET
```

I picked `0x03b98424` because it is always being executed.

The breakpoint would lead to the address above.

`MOV EAX,dword ptr [DAT_03b98424]` This just runs the previous instruction overwritten by call.

```
01f4fdee 48 8d 35        LEA        RSI,[s_r_DrawNearFoV_02c81850]
         5b 1a d3 00
01f4fdf5 48 8d 15        LEA        RDX,[DAT_03b98424]
         28 86 c4 01
01f4fdfc 4c 8d 05        LEA        R8,[s_Sets_the_FoV_for_drawing_of_near_02c8185e]
         5b 1a d3 00
```

# Results

<p align="center">
<img src="{% link assets/images/CrysisRemasteredFramePacingFix/CrysisRemasteredFramePacingFix_Preview.png %}">
<em>Trdrop Frame Graph Overlay.</em>
</p>

Notice how the Green line is more stable than the Orange line.

<div align="center" class="video-container">
<iframe width="560" height="315" src="https://www.youtube.com/embed/Cmm4thcGYZ0" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
</div>

# Patch

<a href="https://github.com/illusion0001/illusion0001.github.io/blob/main/_patches/CrysisRemastered-Orbis.md#30-fps-limit-proper-frame-pacing" class="button" role="button"><i class='fas fa-download'></i> Patch Code</a>

{% include_relative _supporters.md %}
