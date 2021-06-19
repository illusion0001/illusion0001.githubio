---
layout: post
title: "Improved Loading for The Last of Us Remastered"
excerpt: "Long Loading screens are things of the past."
categories: Patches
tags: tlou ps4 patches bugfix
---

*TL;DR - if you are not interested in an in-depth overview of how the patch was made, scroll down to the [**Patch**](#patch) section for a download link.*

***

{% include_relative _orbis_console_note.md %}

* TOC
{:toc}

# Curiosity

It's been a while since The Last of Us Remastered was released for the PlayStation 4 since then I have been wondering why is the loading so slow when using the 30FPS mode. So I decided to investegate.

# Intro

Normally on Naughty Dog Titles, The developers used adaptive sync and allows the framerate to go completely unlocked and to stream assets faster.

However this is absent from this specific release.

## Finding what causes loading to go fast.

Finding out where this adaptive sync or "Allow Screen Tear" as it's named in the developer menu came easy.

The Uncharted Collection developed by Bluepoint Games used "Allow Screen Tear" during it's load screen so searching for Byte `0` and `1` got me to the address I needed.

![](\assets\images\t1r-fast-load\ps4ch-uc3-1.png)

Setting this byte to 1 during gameplay allows for Unlocked FPS!

![](\assets\images\t1r-fast-load\UC3-tear.png)

Sweet.

During the loading screen another option gets disabled. It's called Main Draw.

Main Draw draws 3D scene. When Disabled it gives us a black screen. This is used during load screen to reduce rendering load.

![](\assets\images\t1r-fast-load\T1R-maindraw0.png)

300+ FPS! Not so exciting huh.

## Making it tick.

Using Ghidra, I was able to find out where this main draw byte is disabled during load screen

![](\assets\images\t1r-fast-load\ghidra-maindraw-ref.png)

Going to 0x647631 gives us what we need.

```
        00647629 41 c6 87        MOV        byte ptr [R15 + 0x1d8],0x1
                 d8 01 00 
                 00 01
        00647631 c6 80 df        MOV        byte ptr [RAX + 0x6df]=>DAT_0181ae4f,0x0
                 06 00 00 00
```

We can add our screen tear byte and set it to 1. But where?

```
        006474eb 0f 8f 47        JG         LAB_00647638
                 01 00 00
~~~
        0064760f 74 27           JZ         LAB_00647638
~~~
        00647631 c6 80 df        MOV        byte ptr [RAX + 0x6df]=>DAT_0181ae4f,0x0
                 06 00 00 00
                             LAB_00647638
        00647638 44 89 f0        MOV        EAX,R14D
        0064763b 48 83 c4 08     ADD        RSP,0x8
        0064763f 5b              POP        RBX
        00647640 41 5e           POP        R14
        00647642 41 5f           POP        R15
        00647644 5d              POP        RBP
        00647645 c3              RET
```

We can add our code to 0x647638 and move the related instructions few bytes forward as well as changing jump address to our new location.

```
        006474eb 0f 8f 4e        JG         LAB_0064763f
                 01 00 00
~~~
        0064760f 74 2e           JZ         LAB_0064763f
~~~
        00647631 c6 80 df        MOV        byte ptr [RAX + 0x6df]=>DAT_0181ae4f,0x0
                 06 00 00 00
        00647638 c6 80 30        MOV        byte ptr [RAX + 0x30],=>DAT_0181a7a0,0x1
                 00 00 00 01
                             LAB_0064763f
        0064763f 44 89 f0        MOV        EAX,R14D
        00647642 48 83 c4 08     ADD        RSP,0x8
        00647646 5b              POP        RBX
        00647647 41 5e           POP        R14
        00647649 41 5f           POP        R15
        0064764b 5d              POP        RBP
        0064764c c3              RET
```

Done!

Let's test this change in game. The following video showcases this change aswell as a timer on screen.

<div align="center" class="video-container">
<iframe width="560" height="315" src="https://www.youtube.com/embed/tZjIuAdALhE?start=9" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
</div>

# Patch

To install the patch, open `eboot.bin` with a hex editor and do a search and replace. build and install the patch back onto the console.

<a href="https://github.com/illusion0001/illusion0001.github.io/blob/main/_patches/tlou1.md#improved-loading" class="button" role="button"><i class='fas fa-download'></i> Patch Codes</a>