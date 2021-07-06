---
layout: post
title: "The Witcher 3 60FPS and Grahpics Settings Research for PS4"
excerpt: "Geralt fighting against monsters at up to 60 Frames-Per-Second."
thumbnail: "assets/images/W3Witcher-ResPatch/W3Witcher-Banner.png"
feature-img: "assets/images/W3Witcher-ResPatch/W3Witcher-Preview.png"
image: "assets/images/W3Witcher-ResPatch/W3Witcher-Preview.png"
categories: patches
tags: [Articles, Releases]
twitter: {card: "summary_large_image"}
---

{% include_relative _orbis_console_note.md %}

* TOC
{:toc}

# Intro

I recently saw a [video](https://youtu.be/sPViMidRJxY) from Digital Foundry where they covered a [Modded Nintendo Switch](https://wololo.net/2021/06/14/how-to-hack-your-nintendo-switch-in-2021-hwfly-and-sx-clones-sx-rcm-unpatched-vs-patched-trying-to-clear-it-up-for-you/) running the Witcher 3 at up to 60FPS. Albeit with compermise of course. And the fact that no one up to this point has ported the patch to PlayStation 4 it's my time to shine!

## Part 1 - Resolution and Framerate Unlocking

Let's get this out of the way first, since it's all basic stuff by this point for those following my blog for the last few months.

```
                LAB_01e88385 XREF[1]:     01e88250(j)
01e883ad b9 80 07        MOV        ECX,0x780 ; 1920
         00 00
01e883b2 41 b8 38        MOV        R8D,0x438 ; 1080
         04 00 00
```

Base console resolution setup.

```
01e8823c e8 7f 7d        CALL       sceKernelIsNeoMode
         fb 00
01e8824e 84 c0           TEST       AL,AL
01e88250 0f 84 2f        JZ         LAB_01e88385
         01 00 00
...
01e8828b b9 80 07        MOV        ECX,0x780 ; 1920
         00 00
01e88290 41 b8 70        MOV        R8D,0x870 ; 2160
         08 00 00
```

PS4 Pro resolution setup.

A simple change to 1280x720 on Base does the trick nicely, although I have not tested much of the game. So issues may crop up.

You may be wondering why PS4 Pro is using 1920x2160 for its back buffer resolutio. It's likely for Checkerboarding, halving it's Horizontal resolution and keeping Vertical resolution native.

Framerate? Same story.

```
0227361c be 01 00        MOV        ESI,0x1
         00 00
02273621 e8 7a e2        CALL       sceVideoOutSetFlipRate
         bc 00
```

MOV ESI with 0x0 gives us an unlocked framerate with triple buffered VSync.

## Part 2 - Additional Graphics Options and Framerate Stats

A few things I noticed that is, there's native In-Game Frame Statistics and additional Graphics Options.

<p align="center">
<img src="{% link assets/images/W3Witcher-ResPatch/W3Witcher-DF-Snapshot.jpg %}">
<em>In-Game Frame Statistics with FPSGui Overlay.</em>
</p>

`FPS` String search comes up with.. no results but in Unicode, it does.

```
                u_FPS:_%1.3f_(longest_frame:_%1.3f_030141b4     XREF[1]:     FUN_01f1a860:01f1ab0f(*)  
030141b4 46 00 50        unicode    u"FPS: %1.3f (longest frame: %1.3f ms) VSync: 
         00 53 00 
         3a 00 20 
...
01f1aa4a 41 80 bd        CMP        byte ptr [R13 + 0xb319],0x0
         19 b3 00 
         00 00
01f1aa55 0f 84 33        JZ         LAB_01f1ab8e
         01 00 00
...
01f1aab4 eb 3c           JMP        LAB_01f1aaf2
...
                LAB_01f1aaf2                                    XREF[1]:     01f1aab4(j)  
01f1aaf2 48 8d 05        LEA        RAX,[DAT_03b21fe0]
         e7 74 c0 01
01f1ab0f 48 8d 35        LEA        RSI,[u_FPS:_%1.3f_(longest_frame:_%1.3f_030141
         9e 96 0f 01
```

A simple change from `Jump Zero` to `Jump not Zero` gives us framerate stats.

<p align="center">
<img src="{% link assets/images/W3Witcher-ResPatch/W3Witcher-ShowFPS.png %}">
<em>In-Game Frame Statistics on PS4.</em>
</p>

What about Graphics Options? A simple drop in of the `rendering.xml` file from [GBAtemp](https://gbatemp.net/) Switch Modding Section gives us Grahpical Options seen in various Witcher 3 Switch Modded Videos.

<p align="center">
<img src="{% link assets/images/W3Witcher-ResPatch/W3Witcher-RenderingXML.png %}">
<em>Additional Graphics Options.</em>
</p>

## Part 3 - Experimental Graphics Patches through Executable Patching

This is mostly my own findings before I discovered the modified XML file described above.

I started toggling the motion blur option found in the settings menu and landed at `0x3b21e89` memory address.

```
                             DAT_03b21e89 XREF[4]:     FUN_015b53c0:015b74b0(*), 
                                                       FUN_015b53c0:015b74b7(*), 
                                                       FUN_015b53c0:015b74ca(W), 
                                                       FUN_01f61d80:01f62344(R)  
        03b21e89 00              undefined1 00h
```

A read and write reference, hmm.

```
015b7489 4c 8d 25        LEA        R12,[s_PostProcess_02ee8903]
         73 14 93 01
015b7490 48 8d 15        LEA        RDX,[s_AllowMotionBlur_02ee89cd]
         36 15 93 01
...
015b74ca c6 43 29 01     MOV        byte ptr [RBX + 0x29]=>DAT_03b21e89,0x1
```

How curious, A text ref next to the actual byte. At least it makes my life easier.

What about the read?

```
01f62344 44 0f b6        MOVZX      R15D,byte ptr [RSI + 0x29]=>DAT_03b21e89
         7e 29
```

It reads from `RSI + 0x29` to get the final address in memory.

As a test, `MOV` with `0x0` to `R15W` permanently disables motion blur.

Completely pointless but hey, at least we know how the RED Engine lays out it's graphics settings in the executable.

# Patch

<a href="https://github.com/illusion0001/illusion0001.github.io/blob/main/_patches/W3Witcher-Orbis.md" class="button" role="button"><i class='fas fa-download'></i> Patch Code</a>

{% include_relative _supporters.md %}
