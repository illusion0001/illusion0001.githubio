---
layout: post
title: "Camera Shake Patch for Uncharted 3 and The Last of Us"
excerpt: "Oh, there's camera shake? 👀"
categories: Patches
tags: tlou uncharted uc3 ps3 ps4 patches
---

{% include_relative _orbis_console_note.md %}

* TOC
{:toc}

Recently, I stumped upon a video by a speedrunner [Anthony Caliber](https://www.youtube.com/channel/UC4PlYBhe8mzGFW4lmIkIQsg) which shows a [Video](https://youtu.be/EaBFyrCyMbs?t=302) comparing differences between camera shakes.

I'm not much of a fan of sprinting camera shake myself so let's see what we can do about it.

# Diving into nothingness

Opening the executable in Ghidra and searching for `Disable Camera` gives few results. But these are just text strings, what are they for?

![](\assets\images\t1-camshake\t1-camshake-ghrda-search.png)

These seem to be leftovers from developer menu code.

For PS3. We can see here that it points to addresses.

```
012b1d98 01 4f 34 12     addr       DAT_014f3412 // byte
012b1d9c 00 ec 86 a0     addr       s_Disable_Camera_Additives_00ec86a0 // string
```

On PS4, there's a submenu dedicated to this.

```c
  uVar1 = FUN_00c81020(0xa0);
  FUN_00a29940(uVar1,"Show Camera Additives",FUN_00a298b0,&DAT_01524d28,0);
  FUN_00a2dcd0(param_1,uVar1);
  uVar1 = FUN_00c81020(0xa0);
  FUN_00a29940(uVar1,"Disable Camera Additives",FUN_00a31890,&DAT_01524d29,0);
  FUN_00a2dcd0(param_1,uVar1);
  uVar1 = FUN_00c81020(0xa0);
  FUN_00a29940(uVar1,"Allow Manual Camera Shake",FUN_00a31890,&DAT_0162d0aa,0);
  FUN_00a2dcd0(param_1,uVar1);
```

Let's activate this menu in-game.

![](\assets\images\t1-camshake\t1-camshake-menu.png)

Three options. What we are interested here is `Disable Camera Additives` Let's see what happens when we enable it.

<div align="center" class="video-container">
<video controls >
  <source src="\assets\images\t1-camshake\t1-camshake-demo.mp4" type="video/mp4">
</video>
<em>No more Camera shakes!</em>
</div>

## Smoking Mirrors

Let's start with the PS3 version.

Going to `0x14f3412` in Memory and setting it to 1, does nothing. Why's that?

`0x14f3412` seems to be `Show Camera Additives` as `final` build configs have most debugging features stripped out, it doesn't do anything. Let's try byte next to it. 

<div align="center" class="video-container">
<video controls >
  <source src="\assets\images\t1-camshake\t1-rpcs3-camshake-demo.mp4" type="video/mp4">
</video>
<em>Got it. No more camera shake for the PS3 version.</em>
</div>

For The Last of Us, setting a breakpoint at startup and loading it's with register 11 as it holds value 1 setup by `009941e4 39 60 00 01     li         r11,0x1` made it work.

```yml
  - [ be16, 0x00994234, 0x997d ] # disable camera addtiv
```

## Causing Headaches

For Uncharted 3 however, things are not as cut and dry as one might think.

```
007e5884 38 00 00 01     li         r0,0x1
007e58ac e8 01 00 b0     ld         r0,0xb0(r1) // this will become important later
007e58cc 9b 9d 00 03     stb        r28,0x3(r29)
```

We can try loading it with register 0. But it doesn't work. Let's see in the debugger to find out why.

![](\assets\images\t1-camshake\uc3-rpcs3-camshake-dbgr.png)

r0 is now taken by instruction ld at `7e58ac` and we can't simply nop this and hope that the game will work. It does not.

Stuck at a deadend? Not so easy.

We can set a read breakpoint on `0x013257f3` and see where it will take us.

`008242cc 88 0b 00 03     lbz        r0,0x3(r11)=>DAT_013257f3`

Loading intermediate with register 0 did the trick.

```yml
  - [ be16, 0x008242cc, 0x3800 ]
```

Let's move on to the PS4 Version.

## Soothing for the eyes

We'll need to use a byte that writes on startup, searching for something, say.. `Enable OIT Assert` and going to it's reference, gives us what we need.

```
// uc3 1.00
0015f299 c6 80 62        MOV        byte ptr [RAX + 0x762]=>DAT_016491d2,0x1
         07 00 00 01
0015f2a0 c6 80 60        MOV        byte ptr [RAX + 0x760]=>DAT_016491d0,0x0
         07 00 00 00

// tlou 1.10
0000c2ff c6 83 ec        MOV        byte ptr [RBX + 0x6ec]=>DAT_0181ae5c,0x1
         06 00 00 01
0000c306 c6 83 ea        MOV        byte ptr [RBX + 0x6ea]=>DAT_0181ae5a,0x0
         06 00 00 00
```

Moving a byte on startup? Check. Let's use instruction next to it.

We know that `0x01524d29` is byte we wanted to load looking from it's menu code, we can change this instruction to load our desired byte instead.

```
// uc3 1.00
0015f2a0 c6 05 a2        MOV        byte ptr [DAT_013f9149],0x1
         9e 29 01 01

// tlou 1.10
0000c306 c6 05 1c        MOV        byte ptr [DAT_01524d29],0x1
         8a 51 01 01
```

<!-- embarrassing solution. why do I make things overcomplicated

but we don't have a lot of space to work with here.

To solve this, we can use a code cave.

Somewhere around `0xe7a553` is an unused function, let's use this address as our starting point.

```
// t1 1.10

0000c2ff 67 67 e8        CALL       SUB_00e7a553
                 4d e2 e6 00

00e7a553 c6 83 ec        MOV        byte ptr [RBX + 0x6ec],0x1 // load original byte
         06 00 00 01
00e7a55a c6 05 c8        MOV        byte ptr [DAT_01524d29],0x1 // load byte for cam shake
                 a7 6a 00 01
00e7a561 c3              RET // return
```

First `CALL` here is jumping to our new location. We *must* run original instruction before proceeding any further to retain original code flow.

`MOV        byte ptr [RBX + 0x6ec],0x1` loads our original byte overwrittin by our jump call.

`MOV        byte ptr [DAT_01524d29],0x1` loads our desired byte. In this case `Disable Camera Additives` as we saw from the snippet of menu code earlier. -->

## Comparison

The video below showcases changes introduced by the Patch.

<div align="center" class="video-container">
<iframe width="560" height="315" src="https://www.youtube.com/embed/Zoz7e9jN6Xs" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
</div>

# Patch

To apply patch and for use on an exploitable PlayStation 3 or PlayStation 4 console, you'll have to modify the executable with a hex editor and install it back onto the console.

For RPCS3 Emulator, simply download and enable through Patch Manager.

<a href="https://github.com/illusion0001/illusion0001.github.io/blob/main/_patches/tlou1.md#disable-camera-shake" class="button" role="button"><i class='fas fa-download'></i> Patch Codes</a>

## Credits

Thanks to [ZEROx](https://www.youtube.com/user/ZEROx2085) for porting to Game of The Year Edition of Uncharted 3 (1.10)
