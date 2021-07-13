---
layout: post
title: "Patches for The Order 1886 60FPS, 16:9 Aspect Ratio and More"
excerpt: "Not so cinematic anymore, eh?"
categories: patches
thumbnail: "assets/images/oodle-res-framerate-patches/oddle-res-thumbnail-2.png"
feature-img: "assets/images/oodle-res-framerate-patches/oddle-res-thumbnail-2.png"
image: "assets/images/oodle-res-framerate-patches/oddle-res-thumbnail.png"
tags: [Articles, Releases]
twitter: {card: "summary_large_image"}
---

* TOC
{:toc}

# Intro

Another short post but I find the process leading up to this fascinating! So let's get to it. 

## Part 1 - The Easy Stuff

Using common searching technique for resolution and framerate came up quite easy, or so I thought.

For framerate, it's easy enough; a simple search for kernel function `sceVideoOutSetFlipRate` lands us here:

```
01878214 8b 1d 3a        MOV        EBX,dword ptr [DAT_025c0d54]
         8b d4 00
0187821a 4c 8b b0        MOV        R14,qword ptr [RAX + 0x98]
         98 00 00 00
01878221 ba 02 00        MOV        param_3,0x2
         00 00
01878226 85 db           TEST       EBX,EBX
01878228 74 1c           JZ         LAB_01878246
0187822a 39 1d 68        CMP        dword ptr [DAT_0270fe98],EBX
         7c e9 00
01878230 74 11           JZ         LAB_01878243
01878232 8d 73 ff        LEA        param_2,[RBX + -0x1]
01878235 44 89 e7        MOV        param_1,R12D
01878238 e8 83 c2        CALL       sceVideoOutSetFlipRate
         67 00
0187823d 89 1d 55        MOV        dword ptr [DAT_0270fe98],EBX
         7c e9 00
```

It's moving value from `0x25c0d54` into EBX, tests it against condition of `if not 0` then skips.

Not sure what's going with the compare below but I simply mov `0x1` into `EBX` and it worked.

```
01878214 c7 c3 01        MOV        EBX,0x1
         00 00 00
```

## Part 2 - The (not so) Easy Stuff

I ran a search for 1920 in 4 byte int and clicked around and found this.

```
025c0d40 80 07 00 00     undefined4 00000780h
025c0d44 00              ??         00h
025c0d45 00              ??         00h
025c0d46 00              ??         00h
025c0d47 00              ??         00h
025c0d48 38 04 00        undefined8 0000000000000438h
         00 00 00 
         00 00
```

A qword? That doesn't seem right.

```
018a1286 41 8b 84        MOV        EAX,dword ptr [R12 + 0x14c]
         24 4c 01 
         00 00
018a128e 41 8b 8c        MOV        ECX,dword ptr [R12 + 0x150]
         24 50 01 
         00 00
018a1296 48 89 05        MOV        qword ptr [DAT_025c0d40],RAX             = 00000780h
         a3 fa d1 00
018a129d 48 89 0d        MOV        qword ptr [DAT_025c0d48],RCX             = 0000000000000438h
         a4 fa d1 00
```

I could try moving a custom value into EAX and ECX.

```
018a1296 48 c7 c0        MOV        RAX,0x780
         80 07 00 00
018a129d 48 c7 c1        MOV        RCX,0x438
         38 04 00 00
```

But.. the game didn't boot when I lowered the resolution..

After digging for a while, I found these two.

One is a launch parameter and another is resolution values.

```
                     s_Use_720p_resolution_in_fullscree_0248a513     XREF[1]:     FUN_01449a10:0144a0bd(*)  
0248a513 55 73 65        ds         "Use 720p resolution in fullscreen mode"
         20 37 32 
         30 70 20 
                     s_Use_1080p_resolution_in_fullscre_0248a53a     XREF[1]:     FUN_01449a10:0144a0e8(*)  
0248a53a 55 73 65        ds         "Use 1080p resolution in fullscreen mode"
         20 31 30 
         38 30 70 
                     s_Use_4K_resolution_in_fullscreen_m_0248a562    XREF[1]:     FUN_01449a10:0144a113(*)  
0248a562 55 73 65        ds         "Use 4K resolution in fullscreen mode"
         20 34 4b 
         20 72 65 
...
0248ae3a 75 73 65        ds         "use4K"
         34 4b 00
0248ae41 75 73 65        ds         "use1080p"
         31 30 38 
         30 70 00
0248ae4b 75 73 65        ds         "use720p"
         37 32 30 
         70 00
...
01444cef3 a8 80           TEST       AL,0x80
01444cef5 74 41           JZ         LAB_0144cf38
01444cef7 c7 85 84        MOV        dword ptr [RBP + local_384],0x500
          fc ff ff 
          00 05 00 00
01444cf01 c7 85 88        MOV        dword ptr [RBP + local_380],0x2d0
          fc ff ff 
          d0 02 00 00
```

I think I found it!

<p align="center">
<img src="{% link assets/images/oodle-res-framerate-patches/oddle-res0.png %}">
<em>1920x1080 Letterboxed.</em>
</p>

<p align="center">
<img src="{% link assets/images/oodle-res-framerate-patches/oddle-res1.png %}">
<em>1280x720 Letterboxed</em>
</p>

A simple jump switch did the trick.

Since it is letterboxed, it will have to push out less pixels than actual 720p.

Affective resolution is `1280x535` approximately.

Also, if you haven't noticed already, there's 4K mode. Not sure if it works so I included a code to enable this path. if anyone who has a PS4 Pro can give it a try.

Another curiosty piece of mine is the letterboxing the game has, let's see what I can do about it.

```
Width: 1920 Height: 1080
Ratio: 1.778
2.07 megapixels (2073600 pixels)
```

Initally, I thought it would be `1.778` or `39 8E E3 3F` in Hex, but no results.

```
Width: 1920 Height: 800
Ratio: 2.4
1.54 megapixels (1536000 pixels)
```

However, `2.4` came up with some promising results.

```
01444ce86 c7 85 18        MOV        dword ptr [RBP + local_3f0],0x4019999a
          fc ff ff 
          9a 99 19 40
```

Around resolution code too. let's give it `1.778` and see what the game will do about it.

<p align="center">
<img src="{% link assets/images/oodle-res-framerate-patches/oddle-res2.png %}">
<em>Letterbox Removed.</em>
</p>

Not so cinematic anymore huh?

Well, the letterbox is removed but it ran worse? That is because it has to push more pixels to the screen.

A full 1280x720 image and not 1280x535 as we saw before.

It seem to also crop in the image, I tried looking for an FOV scale but no luck, maybe someone could? If so open a pull request and make the game better!

# Results

<div align="center" class="video-container">
<iframe width="560" height="315" src="https://www.youtube.com/embed/MRnD9XCb1tY" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
</div>

# Patch

<a href="https://github.com/illusion0001/illusion0001.github.io/blob/main/_patches/TO1886-Orbis.md" class="button" role="button"><i class='fas fa-download'></i> Patch Code</a>

{% include_relative _supporters.md %}
