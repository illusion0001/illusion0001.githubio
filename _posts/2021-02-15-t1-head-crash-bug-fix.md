---
layout: post
title: "Solving the Infected's Severed Head Crash Bug in The Last of Us (PlayStation 3)"
excerpt: "Is it a Head? Is it a bottle? No! It's a game bug. Part 1."
categories: Patches
tags: tlou ps3 rpcs3 patches bugfix
feature-img: "/assets/images/t1r-ps4-head-crash/t1r-head-banner.png"
---

Your move. Naughty Dog.

***

* TOC
{:toc}

# Intro

Usually, when an NPC, actor, or Player's buddies does an action where they would cut Infected's Head off, they leave behind a severed head. 

If a Player decided to throw an object, let's say a brick or bottle. The game would crash. Why's that?

Affected Consoles:

- ~~PlayStation 3~~ [Unofficially Patched](https://illusion0001.github.io/patches/2021/02/15/t1-head-crash-bug-fix/)

- ~~PlayStation 4~~ [Unofficially Patched](https://illusion0001.github.io/patches/2021/02/16/t1r-head-crash-bug-fix/)

- PlayStation 5 [Video](https://youtu.be/HQ7oOmx4mmg?t=127)

- PS4/PS5 versions issue still persists on latest patch 1.11
<div align="center" class="video-container">
<video controls autoplay muted >
  <source src="\assets\images\t1-ps3-head-crash\t1-head-crash-before.mp4" type="video/mp4">
</video>
</div>
## Finding what caused the crash

When the game crashed, using an emulator we can see that it gives an access violation as well as the address it stopped.

![](\assets\images\t1-ps3-head-crash\t1-head-acess-violation.png)

Going to this location in Ghidra gives us some clues.

![](\assets\images\t1-ps3-head-crash\ghidra-head-crash-hint.png)

The game does checks if: 

- Object is throwable

- Object collided with is enemy

We can check using a Debugger with a breakpoint set on the crash address.

Hit on the Head.

![](\assets\images\t1-ps3-head-crash\t1-head-debug0.png)

Hit on an Enemy.

![](\assets\images\t1-ps3-head-crash\t1-head-debug-spu-data.png)

In Register 10 and 29, there's some data here. It seems to be collision data.

Our best guess is that the game thinks the object collied is an enemy and no data generated.

## Solution

We can add some data into Register 10 and additional check to prevent crashing.

```yml
 - [ be32, 0x006d9368, 0x483aa7ed ] #Branch
 - [ be32, 0x00a83b54, 0x3d400001 ] #lis r10 = 0x10000
 - [ be32, 0x00a83b58, 0x7f9d5000 ] #cmp r29 vs r10
 - [ be32, 0x00a83b5c, 0x409d0008 ] #Skip if r29<=r10 // If r29 less or equal to r10 then do nothing
 - [ be32, 0x00a83b60, 0x813d0040 ] #lwz r9,0x40(r29) // load as normal
 - [ be32, 0x00a83b64, 0x4e800020 ] #Return
```

Let's implement this fix and see the results!

<div align="center" class="video-container">
<iframe width="560" height="315" src="https://www.youtube.com/embed/yDHUPHUYr1w?start=17" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
</div>

Success! No longer crashes when throwing an object.

# Patch

To use patch on the RPCS3 Emulator, simply download and enable through Patch Manager.

To apply patch and for use on a real PlayStation 3 console, you'll need to decrypt the executable, apply the patch, save and resign the executable.

<a href="https://github.com/illusion0001/illusion0001.github.io/blob/main/_patches/tlou1.md#infecteds-severed-head-crash-bug-fix" class="button" role="button"><i class='fas fa-download'></i> Patch Codes</a>

## Credits

Thank you to [ZEROx](https://www.youtube.com/user/ZEROx2085) for improving over my inital patch.

And [Aphelion](https://www.youtube.com/c/AphelionGamingTV/) for allowing me to use his footage in the post.
