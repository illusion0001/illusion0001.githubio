---
layout: post
title: "Porting and Improving Cheat Codes in The Last of Us"
excerpt: "Joel can build a spaceship now. 🚀🚀🚀"
categories: cheatcodes
tags: tlou ps3 ps4 cheatcodes
---

{% include_relative _orbis_console_note.md %}

* TOC
{:toc}

Recently, I was browsing through the [ArtemisPS3 repository](https://github.com/bucanero/ArtemisPS3/tree/master/docs/codes) of cheats for PS3 and I found myself wondering... "why aren't these on the PS4?"
Some of them look pretty useful, so let's see if we can port a few over, and perhaps improve them as well!

# Looking for clues

I picked out a few cheats that seem reasonably affect Gameplay:

```
Flashlight Never Dies
bungholio ported by Randy97Killa
0 006ACBC8 3C0043B4
0 006ACBCC 901F05E4

Infinite Everything (Ammo, Items, Skills, Parts)
Medo ported by Randy97Killa
0 000336C0 60000000
```

[Source](https://github.com/bucanero/ArtemisPS3/blob/73b2376/docs/codes/The%20Last%20of%20Us%20BCUS98174%20BCES01584%20BCES01585%20BCAS20270%20v01.11.ncl)

Let's see what these cheats do...

**Flashlight never dies:** self explanatory, makes flashlight never runs out of battery.

**Infinite Parts:** Using Parts, items in-game will result in high value.

Let's start with flashlight. We can see in the disassembled code that it loads a high float value (360.00f) into register 0 and stores it. But can we do better?

```
Original Code
006acbc8 ed ad 00 28     fsubs      f13,f13,f0
006acbcc d1 bf 05 f4     stfs       f13,0x5f4(r31)

Patched code
006acbc8 3c 00 43 b4     lis       r0, 0x43b4
006acbcc 90 1f 05 e4     stw       r0, 0x5e4 (r31)
```

Yes! We can nop out fsub, which subtracts the float value by 0.1, we can achieve same result.

Let's have a look in memory to see what clues we can find to port this to the PS4 version.

![](\assets\images\t1-cheat-porting\ps3-ch-fl1.png)

Remember `ProcessWeaponFlashlight` and a few float values, as these will be helpful later on.

Now on to parts.

We can do a quick search through the games memory for the number of parts in Joel's inventory. 

There are 2 results - One address for the HUD, and another for the "real" value. Attempting to modify the HUD value will result in it being overwritten by the real value, so we can ignore the HUD value and focus only on the real value.

![](\assets\images\t1-cheat-porting\ps3-ch-part1.png)

Unfortunately, peeking into memory here won't do much, since the real value is dynamic and it's address is always shifting when the game is loaded.

## Improving cheats

For flashlight code, we already nop fsub instead of loading 360.00f into it, so there's not much we can do to improve this cheat, so let's focus on the items cheat.

For items, since we gain on item use, let's make it opposite, gain high consistent amount on pickup.

Now let's compare the two codes, on use and on pickup. 

```
// On Use
000336c0 7c 0b 4a 2e     lhzx       r0,r11,r9
000336c4 54 08 04 3e     rlwinm     r8,r0,0x0,0x10,0x1f
000336c8 7c 05 00 50     subf       r0,r5,r0
000336cc 7f 88 28 00     cmpw       cr7,r8,r5
000336d0 40 9c 00 08     bge        cr7,LAB_000336d8
000336d4 38 00 00 00     li         r0,0x0
000336d8 2f 87 00 00     cmpwi      cr7,r7,0x0
000336dc 7c 0b 4b 2e     sthx       r0,r11,r9

// On Pickup
0003495c 7f 7f d2 2e     lhzx       r27,r31,r26
00034960 7f bb 52 14     add        r29,r27,param_8
00034964 4b ff f3 91     bl         FUN_00033cf4
00034968 7f 9d 18 00     cmpw       cr7,r29,param_1
0003496c 40 9d 00 08     ble        cr7,LAB_00034974
00034970 7c 7d 1b 78     or         r29,param_1,param_1
00034974 57 a0 04 3e     rlwinm     r0,r29,0x0,0x10,0x1f
00034978 7f bf d3 2e     sthx       r29,r31,r26
```

We see here that the old cheat code nops out lhzx.

```
Infinite Everything (Ammo, Items, Skills, Parts)
Medo ported by Randy97Killa
000336C0 60000000
/*
60000000 is nop.
*/
```

But we can do better than just a nop.

```yml
# add
  - [ be32, 0x003495c, 0x48e58c35 ] # bl 00e8d590
  - [ be32, 0x0e8d590, 0x3fa0014f ] # r29 14f0000
  - [ be32, 0x0e8d594, 0x881d3412 ] # lbz r0, 0x3412, (r29)
  - [ be32, 0x0e8d598, 0x2f800000 ] # cmp r0, 0x0
  - [ be32, 0x0e8d59c, 0x409e0008 ] # bne 0xe8d5a4
  - [ be32, 0x0e8d5a0, 0x7f7fd22e ] # og lhzx
  - [ be32, 0x0e8d5a4, 0x4e800020 ] # ret
  - [ be32, 0x0034968, 0x2f9d0200 ] # r29 to 512
  - [ be32, 0x0034970, 0x3ba00200 ] # li r29 512

# sub
  - [ be32, 0x00336c0, 0x7e2802a6 ] # mflr r17,LR
  - [ be32, 0x00336c4, 0x48e59f01 ] # call 3
  - [ be32, 0x00336c8, 0x7e2803a6 ] # mtlr LR,r17
  - [ be32, 0x00336cc, 0x3a200000 ] # r17 = 0
  - [ be32, 0x0e8d5c4, 0x7c0b4a2e ] # og lhzx
  - [ be32, 0x0e8d5c8, 0x5408043e ] # og rlwinm
  - [ be32, 0x0e8d5cc, 0x3e00014f ] # r16 = 14f0000
  - [ be32, 0x0e8d5d0, 0x89f03412 ] # lbz r15,0x3412,r16
  - [ be32, 0x0e8d5d4, 0x2f8f0000 ] # r15 vs 0
  - [ be32, 0x0e8d5d8, 0x409e0008 ] # bne
  - [ be32, 0x0e8d5dc, 0x7c050050 ] # og sub
  - [ be32, 0x0e8d5e0, 0x7f882800 ] # og cmpw
  - [ be32, 0x0e8d5e4, 0x39e00000 ] # r15 = 0
  - [ be32, 0x0e8d5e8, 0x3a000000 ] # r16 = 0
  - [ be32, 0x0e8d5ec, 0x4e800020 ] # ret
```

Maybe we went a little overboard but here's what it boils down to.

In the first block of code, if the byte at memory address 14f3412 is 1, the program will skip lhzx and load a value of 512 into register 29. Be it parts, pills, ammo, you name it.

In the second block, if the byte is 1, item usage will not decrease.

## Making Our Own Cheat

For those who have already played the game. You may have noticed that we haven't covered tools yet.

What's the point of having thousands of scrap metal if you can't use it...

Let's try making our own cheat.

Searching for Tools level, we get 2 results. We can ignore 1 in executable space. As it's only for HUD value.

![](\assets\images\t1-cheat-porting\ps4-tool1.png)

Tools value has been changed to 127. Let's set a breakpoint on pickup.

![](\assets\images\t1-cheat-porting\ps4-tool2.png)

```
00068742 ff 84 bb        INC        dword ptr [RBX + RDI*0x4 + 0x16448]
         48 64 01 00
```

Guessing from intruction name, could this be incrementing value?

We can instead, move 5 into this specific address on pickup instead of incrementing by 1.

```yml
  # PS3 1.11
      - [ be32, 0x00082960, 0x38000005 ] # tools level load
      - [ be32, 0x00082788, 0x60000000 ]
```

```
00068742 67 67 e8        CALL       SUB_00068462 //cave
         19 fd ff ff

00068462 c7 84 bb        MOV        dword ptr [RBX + RDI*0x4 + 0x16448],0x5 // move 5 into addr
         48 64 01 
         00 05 00 
0006846d c3              RET //returm
```

Done!

## Ports, Ports, Ports, come get 'em

On PS4 we can do a search for the text we noted down earlier: "ProcessWeaponFlashlight" - These floats will become useful later on when we peek into memory.

![](\assets\images\t1-cheat-porting\ps4-ch-fl0.png)

4 results; let's check for similarities.
Found it. Highlighted are the same floats and values. Sweet!

![](\assets\images\t1-cheat-porting\ps4-ch-fl1.png)

Let's set a breakpoint and see where it takes us.

![](\assets\images\t1-cheat-porting\ps4r-flr.png)

```
00697b83 c5 f2 5c        VSUBSS     XMM2,XMM1,dword ptr [RAX + 0x18] // subtract by 0.1
         50 18
00697b88 c4 c1 7a        VMOVSS     dword ptr [R13 + 0x7d4],XMM2
         11 95 d4 
         07 00 00
```

Let's try changing VMOVSS XMM2 to XMM1.

![](\assets\images\t1-cheat-porting\ps3-ch-fl2.png)

Success! Achieved the same result and no use of nop.

```
00697b88 c4 81 7a        VMOVSS     dword ptr [R13 + 0x7d4],XMM1
         11 8d d4 
         07 00 00
```

How about parts?
Let's try searching for parts... 

Same story here but it's just a little different.

On Pickup, the address will shift, but on use or spending it does not.

![](\assets\images\t1-cheat-porting\ps4ch-part1.png)

```
000894dc 66 42 89        MOV        word ptr [RDI + R8*0x2 + 0xf4],AX
         84 47 f4 
         00 00 00
```

Let's try searching for `f4 00 00 00`. 

![](\assets\images\t1-cheat-porting\ghidra-hint-parts.png)

```
00089022 66 45 89        MOV        word ptr [R15 + RBX*0x2 + 0xf4],R14W
         b4 5f f4 
         00 00 00
```

How about setting a breakpoint?

![](\assets\images\t1-cheat-porting\ps4r-part1.png)

RAX, RCX and R14 stores the newest value ready to be loaded into R13.

Let's try loading our specified value, would it work?

```
MOV word ptr [R15 + RBX*0x2 + 0xf4],0x96F
```

![](\assets\images\t1-cheat-porting\ps4-t1-part-edit.png)

But, we can do better than just loading into a register. How about make it optional?

## Options are nice

I had a thought - what if a user wanted to turn off the cheat without exiting the game and patching the executable manually? that'd be much more convenient...

With My knowledge with the inner workings of the menu code, I came up with a solution: utilizing the Developer Menu.

Since the menu is baked into the executable, I can modify, add, or remove contents within the limits of the executable space.

For the PS3 I came up with this:

```yml
  - [ be64, 0x0e8d5b0, 0x496E66696E697465 ] # str Infinite Everything (Ammo, Items, Skills, Parts)
  - [ be64, 0x0e8d5b8, 0x2045766572797468 ]
  - [ be64, 0x0e8d5c0, 0x696E672028416D6D ]
  - [ be64, 0x0e8d5c8, 0x6F2C204974656D73 ]
  - [ be64, 0x0e8d5d0, 0x2C20536B696C6C73 ]
  - [ be64, 0x0e8d5d8, 0x2C20506172747329 ]
  - [ be32, 0x01286780, 0x00e8d5b0 ] # point addr to e8e490
  - [ be32, 0x01286784, 0x014f3412 ] # point addr to 014f3412
```

First few lines writes the text.

Last two lines points previous addresses to our new location, this replaces debug difficulty in `Gameplay...` Menu

Now on to the PS4.

```
// menu code
0006f23b bf a0 00        MOV        EDI,0xa0 // type entry
         00 00
0006f240 e8 db 1d        CALL       FUN_00c81020 // construct entry
         c1 00
0006f245 48 89 c3        MOV        RBX,RAX
0006f248 48 8d 35        LEA        RSI,[0x10954af] // point to some src code path str
         60 62 02 01
0006f24f 48 8d 15        LEA        RDX,[0x1524d28] // some mem address
         d2 5a 4b 01
0006f256 48 89 df        MOV        RDI,RBX 
0006f259 e8 e2 a5        CALL       FUN_00a29840
         9b 00
0006f25e 4c 89 f7        MOV        RDI,R14
0006f261 48 8b f3        MOV        RSI,RBX
0006f264 e8 67 ea        CALL       FUN_00a2dcd0 // end of entry code
         9b 00

// call
00089022 e8 59 8a        CALL       FUN_00c21a80
         b9 00

// compare ADD code
00c21a80 80 3d a1        CMP        byte ptr [DAT_01524d28],0x0 // compare byte
         32 90 00 00
00c21a87 74 0d           JZ         LAB_00c21a96 // if 0, run normal code
00c21a89 66 41 c7        MOV        word ptr [R15 + RBX*0x2 + 0xf4],0x96f // load 96f into addr
         84 5f f4 
         00 00 00 
00c21a94 eb 09           JMP        LAB_00c21a9f // go to return
                     LAB_00c21a96 
00c21a96 66 45 89        MOV        word ptr [R15 + RBX*0x2 + 0xf4],R14W // load from r14w into addr as noraml
         b4 5f f4 
         00 00 00
00c21a9f c3              RET // rerturn

// compare SUB code
000894d0 e8 1b 86        CALL       SUB_00c21af0
         b9 00

00c21af0 44 89 ce        MOV        ESI,R9D // stolen instruction
00c21af3 80 3d 2e        CMP        byte ptr [DAT_01524d28],0x0 // compare byte
         32 90 00 00
00c21afa 75 02           JNZ        LAB_00c21afe // if not 0 return
00c21afc 29 d6           SUB        ESI,EDX // if 0 sub as normal
00c21afe c3              RET // return
```

That might be a little too much to take in, so here's a quick run down.

The Menu code section replaces `Memory...` in the Quick Menu and makes it a toggle.

The Compare ADD code is the same story as the PS3 - if a byte is enabled, load 0x96f into it's register; if the byte isn't enabled, run the original code.

The Compare SUB allows for infinite items - when byte is enabled, parts, item, etc will not decrease.

## Results

<div align="center" class="video-container">
<iframe width="560" height="315" src="https://www.youtube.com/embed/wesqEfidErg" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
</div>

# Patch

<a href="https://github.com/illusion0001/illusion0001.github.io/blob/main/_patches/tlou1.md#improved-cheat-codes" class="button" role="button"><i class='fas fa-download'></i> Patch Codes</a>

## Credits

[ZEROx](https://www.youtube.com/user/ZEROx2085) for help with compare and porting code to the PS3 version.

[hejran7](https://www.youtube.com/channel/UCdfHDdYuuTRoSNoFVkhPOfA) for help with finding address in the PS4 version.
