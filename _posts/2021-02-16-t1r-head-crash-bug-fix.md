---
layout: post
title: "Solving the Infected's Severed Head Crash Bug in The Last of Us (PlayStation 4)"
excerpt: "Is it a Head? Is it a bottle? No! It's a game bug. Part 2."
categories: Patches
tags: tlou ps4 patches bugfix
feature-img: "/assets/images/t1r-ps4-head-crash/t1r-head-banner.png"
---

Your move. Naughty Dog.

***

{% include_relative _orbis_console_note.md %}

* TOC
{:toc}

# Intro

Usually, when an NPC, actor, or Player's buddies does an action where they would cut Infected's Head off, they leave behind a severed head. 

If a Player decided to throw an object, let's say a brick or bottle. The game would crash. Why's that?

Affected Consoles:

- ~~PlayStation 3~~ [Unofficially Patched](https://illusion0001.github.io/patches/2021/02/15/t1-head-crash-bug-fix/)

- ~~PlayStation 4~~ [Unofficially Patched](https://illusion0001.github.io/patches/2021/02/16/t1r-head-crash-bug-fix/) ([Video](https://youtu.be/KCnMwV-jOoU))

- PlayStation 5 [Video](https://youtu.be/HQ7oOmx4mmg?t=127)

- PS4/PS5 versions issue still persists on latest patch 1.11

<div align="center" class="video-container">
<video controls muted >
  <source src="\assets\images\t1r-ps4-head-crash\t1r-head-crash-before.mp4" type="video/mp4">
</video>
</div>

## Cut to the Chase

Most of the explaination was already covered in our PS3 version, see that post for more details.

[Solving the Infected's Severed Head Crash Bug in The Last of Us (PlayStation 3)](https://illusion0001.github.io/patches/2021/02/15/t1-head-crash-bug-fix/)

We were only able to get the debugger [PS4 Reaper](https://www.psxhax.com/threads/ps4reaper-ps4-rte-debugger-and-trainer-maker-by-shiningami.6077/) working once due to connection and attaching issues but that was enough for what we needed.

![](\assets\images\t1r-ps4-head-crash\ps4r-register.png)

Guessing from PS3 registers, It could be RBX that is holding collision data.

## Solution

We can do a check if RBX isn't 0 we can skip and run normally.

```
006bc849 e8 d0 53        CALL       SUB_00c21c1e // jump to code cave
         56 00
~~~
00c21c1e 48 89 85        MOV        qword ptr [RBP + -0xc20],RAX // pervious instruction overwritten by Call
         e0 f3 ff ff
00c21c25 48 83 fb 00     CMP        RBX,0x0 // check if rbx isn't 0
00c21c29 0f 84 04        JZ         LAB_00c21c33 // skip
         00 00 00
00c21c2f 48 8b 43 40     MOV        RAX,qword ptr [RBX + 0x40] // load as normal
                     LAB_00c21c33
00c21c33 c3              RET // return
```

Let's implement this fix and see the results.

<div align="center" class="video-container">
<iframe width="560" height="315" src="https://www.youtube.com/embed/a5QEZGT7HOU?start=10" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
</div>

Success! Doesn't crash now when throwing an object.

# Patch

To apply patch and for use on a exploitable PlayStation 4 console, you'll need to dump the game, modiify the executable with a hex editor and install the fake patch back onto the console.

<a href="https://github.com/illusion0001/illusion0001.github.io/blob/main/_patches/tlou1.md#infecteds-severed-head-crash-bug-fix" class="button" role="button"><i class='fas fa-download'></i> Patch Codes</a>

## Credits

Thank you to [ZEROx](https://www.youtube.com/user/ZEROx2085) for improving over my inital patch. and for believing in x86!
