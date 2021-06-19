---
layout: post
title: "60FPS Patch for What Remains of Edith Finch"
excerpt: "House exploring in 60 Frames Per Second on PlayStation 4"
categories: patches
tags: finchgame ps4 patches
thumbnail: "/assets/images/finchgame-60fps/fg-banner2.png"
feature-img: "/assets/images/finchgame-60fps/fg-banner2.png"
image: "/assets/images/finchgame-60fps/fg-banner.png"
tags: [Articles, Releases]
twitter: {card: "summary_large_image"}
---

{% include_relative _orbis_console_note.md %}

* TOC
{:toc}

# Intro

One of my favorite games this past generation was What Remains of Edith Finch. A underrated beautifully crafted game created by [Giant Sparrow](http://www.giantsparrow.com/blog/). But it doesn't run so well on the PlayStation 4 so let's see what's up with its performace and if I can improve on it.

## Inital Performance Impressions

This game targets its proformance profile at 1080p and 30FPS at least for Base Console.

<div align="center" class="video-container">
<iframe width="560" height="315" src="https://www.youtube.com/embed/PD1jGLGeytQ" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
</div>

25 - 30FPS during the opening scene.. the good thing here is that this is the only time the game will give the player to explore this large open space. during the exploration of the house it stays at 30fps almost all the time.

## Bruteforcing

In my [previous article](https://illusion0001.github.io/patches/2021/05/20/ff7r-end-60fps/) I discovered the inital value for Unreal Engine Screen Percentage and Sync Intervals. Let's have a look in the config files and see if there's anything useful there.

<p align="center">
<img src="{% link assets/images/finchgame-60fps/fg-compressed-ini.png %}">
<em>Yikes.. nope.</em>
</p>

This is extracted using the [UT4 QuickBMS](http://aluigi.org/bms/unreal_tournament_4.bms) script and it didn't look exactly great. UnrealPak gives up and doesn't extract correctly either, So let's try bruteforcing by searching for values in memory instead.

100.0 float for screen percentage, and 2 for sync interval.

<p align="center">
<img src="{% link assets/images/finchgame-60fps/ps4ch-fg1.png %}">
<em>Looks like I found it.</em>
</p>

That was simple at least for screen percentage but I ran into a deadend with sync interval.. how about we look in the executable? Could be clues that should lead us somewhere.

```
XREF[1]:     FUN_027149a0:027149f1
                               (*)
042247ac 72 00 2e        unicode    u"r.VSync"
         00 56 00 
         53 00 79 ...
```

Searching for sync came up with only a couple of releavent results. There's `r.VSync` though. let's checkout that cvar and see what's up.

```
027149ee 48 8b 07        MOV        RAX,qword ptr [RDI]
027149f1 48 8d 35        LEA        RSI,[u_r.VSync_042247ac]
         b4 fd b0 01
027149f8 ff 90 80        CALL       qword ptr [RAX + 0x80]
         00 00 00
027149fe 31 c9           XOR        ECX,ECX
02714a00 48 85 c0        TEST       RAX,RAX
02714a03 74 0c           JZ         LAB_02714a11
...
02714a29 48 8b 05        MOV        RAX,qword ptr [DAT_0532b158]
         28 67 c1 02
02714a30 c5 f0 57 c9     VXORPS     XMM1,XMM1,XMM1
02714a34 83 38 00        CMP        dword ptr [RAX],0x0
02714a37 74 18           JZ         LAB_02714a51
02714a39 0f b6 83        MOVZX      EAX,byte ptr [RBX + 0x608]
         08 06 00 00
02714a40 c5 f0 57 c9     VXORPS     XMM1,XMM1,XMM1
02714a44 83 f8 02        CMP        EAX,0x2
02714a47 74 08           JZ         LAB_02714a51
02714a49 c5 fa 10        VMOVSS     XMM1,dword ptr [RBX + 0x60c]
         8b 0c 06 
         00 00
```

A `mov` and `cmp` instructions? Seems to use a pointer that is then allocated in memory but this is *not* eboot space as we'll see later so let's set a breakpoint and see where it takes us.

<p align="center">
<img src="{% link assets/images/finchgame-60fps/ps4r-fg2.png %}">
<em>RAX has our memory location somewhere up high, let's check it out.</em>
</p>

<p align="center">
<img src="{% link assets/images/finchgame-60fps/ps4r-fg3.png %}">
</p>

A row of values. seems to be 4 byte int. setting one of these values to 0 introduces tearing.

<div align="center" class="video-container">
<video controls >
  <source src="/assets/images/finchgame-60fps/fg=demo-tear.mp4" type="video/mp4">
</video>
<em>Oof, not looking good.</em>
</div>

This isn't ideal because we are targetting up to 60FPS and not completely unlocked.

```
02714a34 83 38 00        CMP        dword ptr [RAX],0x0
02714a37 74 18           JZ         LAB_02714a51
```

There's a compare instruction here. Let's set our value back to 1 and set this jump to Jump Not Zero and see what it does.

<div align="center" class="video-container">
<video controls >
  <source src="/assets/images/finchgame-60fps/fg=demo-no-tear.mp4" type="video/mp4">
</video>
<em>60FPS target and no screen tear! Perfect.</em>
</div>

How about screen percentage? Almost forgot about it heh.

```
02893389 48 8b 07        MOV        RAX,qword ptr [RDI]
0289338c 48 8d 35        LEA        RSI,[u_r.ScreenPercentage_0423d4d6]
         43 a1 9a 01
02893393 ff 90 80        CALL       qword ptr [RAX + 0x80]
         00 00 00
02893399 31 c9           XOR        ECX,ECX
0289339b 48 85 c0        TEST       RAX,RAX
0289339e 74 0c           JZ         LAB_028933ac
...
028933bf 48 8b 05        MOV        RAX,qword ptr [DAT_053322f8]
         32 ef a9 02
028933c6 c5 f0 57 c9     VXORPS     XMM1,XMM1,XMM1
028933ca c5 fa 10 00     VMOVSS     XMM0,dword ptr [RAX]
028933ce c5 f8 2e c8     VUCOMISS   XMM1,XMM0
```

Setting a memory breakpoint on that address I found earlier brought us here.

Similar looking code at the bottom but there isn't enough space to write our code here. Let's pick a random spot in the executable nearby to write our code.

Plan is to write a custom float value to that pointer which then will be read by `VMOVSS` instruction and does all the heavy lifting for us. 

```
028933bf e8 5d 07        CALL       SUB_02893b21 ; cave
         00 00
...
02893b21 48 8b 05        MOV        RAX,qword ptr [DAT_053322f8] ; previous instruction
         d0 e7 a9 02
02893b28 c7 00 0a        MOV        dword ptr [RAX],0x4285570a ; write our float value in hex
         57 85 42
02893b2e c3              RET ; return from subroutine
```

Pretty simple right? This should work in-game, let's test it out.

<div align="center" class="video-container">
<video controls >
  <source src="/assets/images/finchgame-60fps/fg-demo-after.mp4" type="video/mp4">
</video>
<em>60FPS indoors and of course no tearing to be found.</em>
</div>

# Result

Image quailty comparisons, can you guess which is which?

{% include_relative _image_note.md %}

<p align="center">
<img src="{% link assets/images/finchgame-60fps/fg-before.png %}">
</p>

<p align="center">
<img src="{% link assets/images/finchgame-60fps/fg-after.png %}">
</p>

<div align="center" class="video-container">
<iframe width="560" height="315" src="https://www.youtube.com/embed/BQaG989KYcs" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
</div>

# Patch

<a href="https://github.com/illusion0001/illusion0001.github.io/blob/main/_patches/finchgame-Orbis.md#60fps-unlock" class="button" role="button"> <i class='fas fa-download'></i> Patch Code</a>

Patches released since Final Fantasy VII: Remake

<a href="https://github.com/illusion0001/illusion0001.github.io/blob/main/_patches/lis/captainspirit.md" class="button" role="button"><i class='fas fa-download'></i> Awesome Adventures of Captain Spirit</a>

<a href="https://github.com/illusion0001/illusion0001.github.io/blob/main/_patches/FD4_FDP-Orbis.md" class="button" role="button"><i class='fas fa-download'></i> Dark Souls III</a>

<a href="https://github.com/illusion0001/illusion0001.github.io/blob/main/_patches/GhostOfTsushima-Orbis.md" class="button" role="button"><i class='fas fa-download'></i> Ghost Of Tsushima</a>

<a href="https://github.com/illusion0001/illusion0001.github.io/blob/main/_patches/Tresgame-Orbis.md" class="button" role="button"><i class='fas fa-download'></i> Kingdom Hearts: III</a>

<a href="https://github.com/illusion0001/illusion0001.github.io/blob/main/_patches/P5R-Orbis.md" class="button" role="button"><i class='fas fa-download'></i> Persona 5: Royal</a>

<a href="https://github.com/illusion0001/illusion0001.github.io/blob/main/_patches/lis/prombasegame-lis2.md" class="button" role="button"><i class='fas fa-download'></i> Life is Strange 2</a>

{% include_relative _supporters.md %}
