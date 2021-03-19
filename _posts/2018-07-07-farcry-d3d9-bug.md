---
layout: post
title: "Fixing a Direct3D9 bug in Far Cry"
excerpt: How water reflections revealed a regression in D3D9 - and how it was fixed!
feature-img: "assets/img/posts/farcry/farcry.jpg"
thumbnail: "assets/img/posts/farcry/farcry.jpg"
image: "assets/img/posts/farcry/farcry.jpg"
date: 2018-07-07 18:10:00 +0200
twitter: {card: "summary_large_image"}
tags: [Releases, Articles]
---

TL;DR - if you are not interested in an in-depth overview of what was wrong with the game and how it was fixed, just follow the link to check out a concise changelog and grab **SilentPatch for Far Cry**: \\
<a href="{% link _games/far-cry.md %}#silentpatch" class="button" role="button" target="_blank">{{ site.theme_settings.download_icon }} Download SilentPatch for Far Cry</a> \\
Upon downloading, all you need to do is to extract the archive to game’s directory and that’s it!

# Overview

Far Cry (developed by [Crytek](https://en.wikipedia.org/wiki/Crytek)) once a game considered an example of visual fidelity and de facto a benchmark of then-modern PCs, turns out not to be free of issues.
The main issue bothering people for years were broken water reflections - landmass would not reflect on water if the game is played on anything newer than Windows XP:

<p align="center">
<img src="{{ site.baseurl }}/assets/img/posts/farcry/farcry-broken.jpg">
</p>

You can see trees and rocks are reflecting fine - but bigger chunks of land are not, so reflections look far less impressive than they do on XP!

Community has since found a way to fix this issue - it is possible to use **WineD3D**, a Direct3D to OpenGL wrapper for Windows, and then everything looks fine.
However, it comes at a price -- performance can be lowered by as much as 75%! That can result in unacceptable framerates even on modern PCs.

Can we do better? Of course!

# Research

Of course, as with any graphical related issues, usage of PIX graphics debugger could not be overstated. For starters, I captured several frames on both Windows 10 and Windows XP virtual machine.
Nothing seemed out of place -- XP frames indeed display correctly, while Win10 frames display with a bug.

However, one test proved this issue to be much weirder than anyone could have guessed. I attempted to capture a frame on a XP virtual machine, and then preview it on Windows 10.
This didn't have a very high chance of succeeding, since PIX frames are strictly bound to user's current GPU and drivers -- so previewing captures from different PCs is not always possible.
However, in this case it worked and revealed something bizarre...

That's how a texture for water reflections displays when previewed in PIX on Windows XP:
<p align="center">
<img src="{{ site.baseurl }}/assets/img/posts/farcry/refxp2.png">
</p>

Compare it with **the very same texture previewed on Windows 10**:
<p align="center">
<img src="{{ site.baseurl }}/assets/img/posts/farcry/ref102.png">
</p>

What gives!? This is exactly the same capture, exactly the same sequence of D3D calls leading to a different result! What does that mean?

With almost absolute certainty, what is being observed here is a **regression in D3D9 implementation** --
it is very possible that it was introduced with Windows Vista display driver model (WDDM), a replacement for older Windows XP display driver model (XPDM).
This should not have happened, but for some reason it did. Why? Since it was never noticed by Microsoft nor any game developers,
it hints that the feature which broke is so obscure and minor so very little applications (Far Cry could be one of the few, or even the only game to rely on it)
so nobody ever cared.

With this in mind, I could start looking through PIX capture looking for anything which I wasn't familiar with.

# Diving into the obscurities of D3D9

Turns out, Far Cry indeed uses something which I wasn't even aware of, until now -- [Clip Planes](https://docs.microsoft.com/en-us/windows/desktop/api/d3d9/nf-d3d9-idirect3ddevice9-setclipplane).

Normally, only front (near) and back (far) planes are used when rendering to cut off too close or too distant geometry:
<p align="center">
<img src="https://docs.microsoft.com/en-us/windows/desktop/direct3d9/images/frustum.png">
</p>

However, user clip planes allow developers to further customize the shape of the view frustum.
Sounds handy, but turns out almost no one ever needed it!
The feature was underused so much so it is not natively supported by modern hardware anymore -- instead, [it's emulated](https://stackoverflow.com/a/5618002/9214270).

Let's see how the game looks if we completely disable this feature:
<p align="center">
<img src="{{ site.baseurl }}/assets/img/posts/farcry/farcry-noclip.jpg">
</p>

That's... better. Landmass is reflecting, but we can also spot artifacts on the water. If we then preview water reflection map, it becomes obvious what those are:
<p align="center">
<img src="{{ site.baseurl }}/assets/img/posts/farcry/ref_noclip.png">
</p>

Turns out those are underwater models being drawn as a part of water reflection! It makes perfect sense -- developers set up a clip plane adjacent to water surface,
so anything below it gets clipped and not included in the reflection map. However, if clip planes are broken, we can assume more geometry is clipped than it was needed.

# The solution

You might ask -- why is this feature broken? MSDN **may** have the answer -- notice the fragment from [SetClipPlane documentation page](https://docs.microsoft.com/en-us/windows/desktop/api/d3d9/nf-d3d9-idirect3ddevice9-setclipplane) I emphasised:
> The coefficients that this method sets take the form of the general plane equation. If the values in the array at pPlane were labeled A, B, C, and D in the order that they appear in the array,
> they would fit into the general plane equation so that Ax + By + Cz + Dw = 0. A point with homogeneous coordinates (x, y, z, w) is visible in the half space of the plane if Ax + By + Cz + Dw >= 0.
> Points that exist behind the clipping plane are clipped from the scene.
>
> **When the fixed function pipeline is used the plane equations are assumed to be in world space.**
> **When the programmable pipeline is used the plane equations are assumed to be in the clipping space (the same space as output vertices).**

That is a fundamental difference, because same set of coordinates has a completely different meaning depending on context.
Now, if shader-based (programmable) pipeline assumes coordinates to be in clipping space (and thus change space for each rendered geometry),
what if we assume they somehow invalidate and re-apply them before each draw?

<p align="center">
<img src="{{ site.baseurl }}/assets/img/posts/farcry/farcry-fixed.jpg">
</p>

**It works!** It was a very long shot, but turned out to be accurate enough! Saving requested clip planes and re-applying them for each draw
(if they are enabled, of course) seems to solve the issue completely. Awesome!

# Unanswered questions

At this point, I decided it's enough to call it fixed and release a patch. Unlike WineD3D, this does not affect performance whatsoever -- and the amount of added D3D
calls is negligible, because clip planes are really not used this often.

However, one question remains unanswered -- what **really** invalidates clip planes? They are **not** invalidated after each draw,
so it can be any other D3D call setting geometry up -- shaders, shader constants, some render state...
As much as I'd like to know, figuring it out would take way too much time to be feasible,
especially since the current fix has proven to have absolutely no disadvantages.

***

For those interested,
full source code of the patch has been published on GitHub, so it can be freely used as a point of reference: \\
<a href="https://github.com/CookiePLMonster/SilentPatchFarCry" class="button github" role="button" target="_blank">{{ site.theme_settings.github_icon }} See source on GitHub</a>
