---
layout: post
title: "Fixing A Rare Bug in Uncharted 2 (PlayStation 3/4)"
excerpt: "Rage quitting causing a game crash? Sounds normal. Went unnoticed for over 12 years."
categories: Patches
tags: uncharted uc2 ps3 rpcs3 ps4 patches bugfix
---

## Intro

There's a rare bug in Uncharted 2 where if a player goes out of the normal playzone, dies, and quit to the menu, the game would crash.

Affected Consoles:

- ~~PlayStation 3~~ Unofficially Patched

- ~~PlayStation 4~~ Unofficially Patched

The issue still persists on latest patch 1.02 on PS4

## First Sight

Looking at instructions and fault address, assuming that 4b4 is SPU instruction and 0x620 for PS4, seems to be reading from an invalid memory address (access violation)

```
PS3 Stop Instruction:

00262e04 a0 1c 04 b4     lhz        r0,DAT_000004b4(r28)

Log from RPCS3

Thread (main_thread) [0x00262e04]} VM: Access violation reading location 0x4b4 (unmapped memory)

PS4 Stop Instruction:

000ade75 66 41 83        CMP        word ptr [R15 + 0x620],0x0
         bf 20 06 
         00 00 00

Mira Log from PS4

# reason: page fault (user read data, page not present)
# fault address: 0000000000000620
# rip: 00000000004ade75 
```

Data stops processing and left the registers with 0 when quitting to the menu during this specific case, causing the game to crash.

RPCS3: During runtime

![](\assets\images\uc2-quit-menu-bug-fix\uc2-dbg-before.png)

RPCS3: During crash

![](\assets\images\uc2-quit-menu-bug-fix\uc2-dbg-after-death.png)

On Playstation 4, this is similar but in register R15 instead.

PS4: Reaper Breakpoint during runtime.

![](\assets\images\uc2-quit-menu-bug-fix\ps4r-uc2-dbg1.png)

PS4: Mira backtrace when crash.

![](\assets\images\uc2-quit-menu-bug-fix\ps4-uc2-crash-backtrace-mira.png)

Looking at register data. R28 (PS3) and R15 (PS4) gets used during gameplay and are always executed and have data.

## Resolution

We add a check to compare against the base register (28 for PS3 and 15 for PS4 respectively)

Skip to load 1 into their needed register skip and return to normal code if isn't 0.

```yml
  # quit to menu fix ppc ver
  - [ be32, 0x00262e04, 0x488ce301 ] #Call to code cave
  - [ be32, 0x00b31104, 0x2f9c0000 ] #Compare r28 to 0
  - [ be32, 0x00b31108, 0x409e000c ] #Skip to lhz if !=0
  - [ be32, 0x00b3110c, 0x38000001 ] #r0 = 1
  - [ be32, 0x00b31110, 0x48000008 ] #Go to return
  - [ be32, 0x00b31114, 0xa01c04b4 ] #Default lhz
  - [ be32, 0x00b31118, 0x4e800020 ] #Return
```

```
  // quit to menu fix x86 ver
000ade75 e8 87 cc        CALL       SUB_006cab01 // code cave
         61 00
                             SUB_006cab01
006cab01 49 83 ff 00     CMP        R15,0x0 // compare r15 to 0
006cab05 0f 85 09        JNZ        LAB_006cab14 // go to lea
         00 00 00
006cab0b 48 c7 c7        MOV        RDI,0x1 // move rdi with 1
         01 00 00 00
006cab12 eb 07           JMP        LAB_006cab1b // Go to Compare
                     LAB_006cab14
006cab14 49 8d bf        LEA        RDI,[R15 + 0x620] // Lea from rdi to r15 + 620, no LEA in original code, does CMP instead
         20 06 00 00
                     LAB_006cab1b
006cab1b 48 83 ff 00     CMP        RDI,0x0 // if rdi 0
006cab1f c3              RET // Return
```

Let's implement this fix and see the results.

<iframe width="640" height="360" src="https://www.youtube.com/embed/UdDs6-ZT8gw?start=31" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

Awesome! No longer crashing when quitting to the menu.

## Patch

To apply patch and for use on an exploitable PlayStation 3 or PlayStation 4 console, you'll to modify the executable with a hex editor and install it back onto the console.

For RPCS3 Users, Simply download and enable through it's built-in patch manager.

In Eboot.bin, Find and Replace

```

PS3 Version

1.00

80 4B 00 04 4E 80 04 21 E8 41 00 28 78 7C 00 20 A0 1C 04 B4

to

80 4B 00 04 4E 80 04 21 E8 41 00 28 78 7C 00 20 48 50 E8 E9

38 A0 FF FF 80 DE 81 A0 C0 3E 82 68 78 A5 00 20 C0 5E 81 98 4B FC 04 B5 60 00 00 00 FC 00 98 90

to

2F 9C 00 00 40 9E 00 0C 38 00 00 01 48 00 00 08 A0 1C 04 B4 4E 80 00 20 60 00 00 00 FC 00 98 90

1.09

80 4B 00 04 4E 80 04 21 E8 41 00 28 78 7C 00 20 A0 1C 04 B4

to

80 4B 00 04 4E 80 04 21 E8 41 00 28 78 7C 00 20 48 8C E3 01

2C 42 2C 25 73 2C 20 30 30 30 2C 20 25 73 0A 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00

to

2C 42 2C 25 73 2C 20 30 30 30 2C 20 25 73 0A 00 2F 9C 00 00 40 9E 00 0C 38 00 00 01 48 00 00 08 A0 1C 04 B4 4E 80 00 20 00 00 00 00

####

PS4 Version

1.00

from

66 41 83 BF 20 06 00 00 00 7E 5A

to

E8 87 CC 61 00 90 90 90 90 7E 5A

from

48 89 E5 41 56 53 49 89 F6 48 89 FB 48 8B 4B 08 48 8B 43 10 48 29 C1 48 C1 E9 02 83 F9 02 77

to

49 83 FF 00 0F 85 09 00 00 00 48 C7 C7 01 00 00 00 EB 07 49 8D BF 20 06 00 00 48 83 FF 00 C3

1.02

Needs porting!

```

## Credits

Thanks to [ZEROx](https://www.youtube.com/user/ZEROx2085) for implementing fix in PowerPC Version.
