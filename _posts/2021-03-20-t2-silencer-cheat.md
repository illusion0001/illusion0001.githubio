---
layout: post
title: "Improving an existing cheat in The Last of Us: Part II"
excerpt: "Ellie can now be James Bond"
categories: cheatcodes
tags: tlou2 ps4 cheatcodes
thumbnail: "/assets/images/t2-silencer-cheat/banner2.png"
feature-img: "/assets/images/t2-silencer-cheat/banner2.png"
image: "/assets/images/t2-silencer-cheat/preview.png"
tags: [Articles, Releases]
twitter: {card: "summary_large_image"}
---

{% include_relative _orbis_console_note.md %}

* TOC
{:toc}

With the recent release of the [7.5x exploit](https://wololo.net/2021/03/17/ps4-7-55-jailbreak-sleirsgoevy-releases-updated-version/), I have an opportunity to make a cheat I wanted a reality.

For those who have not played The Last of Us: Part II, there is now a proper cheat system. intended for players who have completed the game once.

# Quiet or they'll find out

When activating a cheat, one expects it to do what it says in the title, right?

There’s a fault with one of the provided cheats, “Infinite Ammo” - Whether this is intended or not is another question, but I wanted to improve on it since it’s been on my mind since the feature was made available.

The issue with this cheat is that ammo still decreases for the silencer, one of the games craftable items for the Pistol.

<div align="center" class="video-container">
<video controls >
  <source src="/assets/images/t2-silencer-cheat/t2-inf-ammo-demo.mp4" type="video/mp4">
</video>
<em>Video showing cheat active. But silencer ammo still decreasing.</em>
</div>

## Find it first or it will be gone

I can do a search for the silencer ammo, find it, change the value, and should be it right, right?

Not so fast. The byte for the ammo itself will shift after sometime idling, this makes the process a little difficult. but after a while, I found its memory region and found it within a couple of minutes.

<p align="center">
<img src="{% link assets/images/t2-silencer-cheat/ps4ch-list.png %}">
<em>Ammo have been changed to 99.</em>
</p>

<p align="center">
<img src="{% link assets/images/t2-silencer-cheat/ps4r-bp.png %}">
<em>Breakpoint at silencer ammo address when firing.</em>
</p>

```
015df257 41 ff 8e        DEC        dword ptr [R14 + 0x814]
         14 08 00 00
```

This is our instruction for the silencer ammo. Nop this and the ammo will not decrease.

## Options are nice

How about implementing this into the cheat that's built into the game. Sounds simple enough.

<p align="center">
<img src="{% link assets/images/t2-silencer-cheat/ps4ch-list2.png %}">
<em>Toggling on and off the cheat option in the menu.</em>
</p>

<p align="center">
<img src="{% link assets/images/t2-silencer-cheat/ps4r-bp2.png %}">
<em>Hit breakpoint when toggling.</em>
</p>

```
003188d0 48 8b 05        MOV        RAX,qword ptr [DAT_037675b0]
         d9 ec 44 03
003188d7 80 b8 ac        CMP        byte ptr [RAX + DAT_000043ac],0x0
         43 00 00 00
003188de 0f 95 c1        SETNZ      CL
003188e1 44 30 f1        XOR        CL,R14B
003188e4 80 f9 01        CMP        CL,0x1
003188ed 44 88 b0        MOV        byte ptr [RAX + DAT_000043ac],R14B
         ac 43 00 00
```

You can see here that the first instruction setups a pointer, the second compares if isn't 0, writes 1 to CL, last instruction then moves from what is in R14B, into the specified address.

Let's make it a toggle.

Knowing that instruction at 0x15df257 decreases the silencer ammo and skipping it does not gives a few clues.

```
015df257 67 67 e8        CALL       FUN_01404590 // call to cave
         32 53 e2 ff
01404590 48 8b 05        MOV        RAX,qword ptr [DAT_037675b0] //set ptr
         19 30 36 02
01404597 80 3d e3        CMP        byte ptr [DAT_035aec81],0x0 // leftover, not needed in final!!
         a6 1a 02 00
0140459e 80 b8 ac        CMP        byte ptr [RAX + 0x43ac],0x0 //cmp ptr +43ac
         43 00 00 00
014045a5 75 07           JNZ        LAB_014045ae if isn't 0 go to ret
014045a7 41 ff 8e        DEC        dword ptr [R14 + 0x814] // decreae as normal
         14 08 00 00
014045ae c3              RET
```

Replaced `DEC` instruction to call to our new location, setup a pointer, compare if is not 0, do nothing, if 0 decrease ammo as normal.

## Result

<div align="center" class="video-container">
<iframe width="560" height="315" src="https://www.youtube.com/embed/ZRg-gr79vp8" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
</div>

# Patch

<a href="https://github.com/illusion0001/illusion0001.github.io/blob/main/_patches/tlou2.md#improved-inf-ammo" class="button" role="button"><i class='fas fa-download'></i> Patch Codes</a>

# One More Thing

As you may have noticed, The blog now has a new theme.

Switched from [Yet Another Theme](https://github.com/jeffreytse/jekyll-theme-yat) to Silent's version of [Type on Strap](https://github.com/Sylhare/Type-on-Strap) for a more cleaner design and a few QoL changes.

I have also ported the [Infected's Severed Head Crash Bug Fix](https://github.com/illusion0001/illusion0001.github.io/blob/main/_patches/tlou1.md#infecteds-severed-head-crash-bug-fix) to the latest version of The Last of Us Remastered. (1.11)

Also, here is Naughty Dog code for loading section on 1.11.

<div align="center">
<img src="{% link assets/images/t2-silencer-cheat/t1-111-load0.png %}">
<em>Official Patch.</em>
</div>

<div align="center">
<img src="{% link assets/images/t2-silencer-cheat/t1-110-load.png %}">
<em>My Patch.</em>
</div>

0x2c is framelock but the fact that it doesn't even work in the menu made me laughed a little.

{% include_relative _supporters.md %}
