---
layout: post
title: "SilentPatch for Scarface: The World is Yours & launching a Patreon campaign"
thumbnail: "assets/img/posts/scarface/scarface.jpg"
feature-img: "assets/img/games/bg/scarface.jpg"
image: "assets/img/posts/scarface/scarface.jpg"
excerpt: "Fixed graphical artifacts, major performance improvements and... Patreon."
date: 2020-03-29 19:30:00 +0200
tags: [Releases, Articles]
---

*TL;DR - if you are not interested in an in-depth overview of what was wrong with the game and how it was fixed,
scroll down to [**Download**](#changelog-and-download) section for a link and
a [Patreon](https://www.patreon.com/{{ site.theme_settings.patreon }}) announcement.*

***

# Introduction

Today's subject is **Scarface: The World is Yours**. This tie-in game to a 1983 movie Scarface was
developed by Radical Entertainment and released in late 2006.

Naturally, by now you likely know where it's going -- what is wrong this time?
As always, [PCGamingWiki page](https://www.pcgamingwiki.com/wiki/Scarface:_The_World_Is_Yours) provides
an answer -- at the time of writing this post, you can see the issue listed as one of the key points:

> <i class="fas fa-thumbs-down"></i> Graphics are corrupted on modern versions of Windows

This doesn't tell much about how bad is it, so what does it look like?

<p align="center">
<img src="{% link assets/img/posts/scarface/screen-bug.png %}"><br>
<img src="{% link assets/img/posts/scarface/ET0GUT7XgAo5jp4.jfif %}"><br>
<img src="{% link assets/img/posts/scarface/ET0GVjTX0AUC6Ii.jfif %}">
</p>

It is... very bad. While results vary across different PC's greatly, nearly **every** modern PC displays this
broken mess, rendering the game unplayable. Of course, community came up with workarounds, but they sadly have their drawbacks:
* dgVoodoo can be used with the game, and it seems to fix the issue. However, dgVoodoo wraps game's Direct3D 9 to Direct3D 11,
which might bump up system requirements considerably. Considering how complex DX9 is, it's also hard to prove that absolutely
everything renders flawlessly.
* Curiously, this issue also goes away when the game runs through... PIX, a graphics debugger! However, similar issues apply
-- given PIX is used to collect data, it might introduce additional overhead. Also, having to always run the game through
it is a bit clunky, isn't it?

While these might suffice for casual play, neither of these workarounds address the root cause of the issue.
However, we want to know what **exactly** went wrong, so those aren't enough -- armed with a debugger and
a virtual machine, I can proceed to figuring this issue out.

# Part one -- critical bugs

Step one to figuring out such issues is of course attaching a debugger. This time, since the issue seems to be
specific to D3D, I aided myself with [DirectX Wrappers](https://github.com/elishacloud/DirectX-Wrappers)
from Elisha Riedlinger. Having a minimal `d3d9.dll` wrapper is excellent for prototyping,
since it allows me to instantly and reliably tap into game's rendering code without any game specific hacking.

That said, this issue could really be anything. That's where experience helps, so I was able to come up with theories
by just observing how the bug looks visually:
* When in game, moving around makes those *shapes* move, possibly corresponding player character's animations.
This means that at least part of this garbage is in fact Tony's model.
* Results vary every session, and on some machines it might even look very different -- for example, 
I have seen variations where most of the environment was not corrupted, but people were T-posing.

I theoretized that this issue relates to vertex buffers not updating properly. At this point it's worth
to mention that people thought this issue shows up only on multicore CPU's. If this is true, then it could mean
that the issue is a classic race condition -- which also would explain why it looks different every time.
Conceptually, it also makes sense -- game could be loading models on multiple threads, setting up D3D resources
concurrently and I imagine that failing to do so in a thread safe manner *could* result in artifacts looking like that.

That theory is trivial to confirm or debunk via a DX9 wrapper. Set up additional code to verify
that all calls to `IDirect3DVertexBuffer9::Lock` and `IDirect3DVertexBuffer9::Unlock` are done from the same thread.
If they are done from multiple threads, it could be a possible culprit.

I set up verification, and... nothing ¯\\\_(ツ)\_/¯ Theory debunked.

Let's take another look at vertex buffer locks though. I spotted that some (not all) locks are done with a
`D3DLOCK_DISCARD` flag. Looking at MSDN docs, it's defined as (emphasis added by me):

> The application discards all memory within the locked region.
> **For vertex and index buffers, the entire buffer will be discarded.**
> This option is only valid when the resource is created with dynamic usage.

I don't know about you, but I see possible room for error here. It is possible to lock only a part of the buffer,
so I can imagine somebody not realizing that locking a part of the buffer with discard flag would throw it away
**entirely**.

However, game **always** locks entire buffers:
```cpp
  if ( v2 )
    v4->Lock(v3, 0, 0, &v11, D3DLOCK_DISCARD);
  else
    v4->Lock(v3, 0, 0, &v11, 0);
```

Analyzing the code further though, I was not fully convinced that after discarding game always fills the entire buffer.
So what if we assume the flag is added there wrongly and remove it?

<p align="center">
<img src="{% link assets/img/posts/scarface/ET-b9GeWsAI9uqk.jfif %}">
</p>

It **works**! It doesn't seem to be a fluke either -- the game was consistently fixed for me and several other people
who tested.

A cautious reader might stop right here and ask a few questions:
* _Clearly, game worked as-is at some point, so how do you know this is not just another workaround and
it'll break again?_
* _Why does using PIX and dgVoodoo fix it?_

# Proving correctness

With a DX9 wrapper and knowledge on when `D3DLOCK_DISCARD` is supposed to be used, I could come up with proof
that this flag was used wrongly. In principle, you'd use this flag when locking a buffer if you did not care
about the buffer's past contents and intended to fill it with new data. If the game used this flag, but also expected
the buffer to retain its old contents, then the flag was misused.

To test whether the game really cares about previous buffer contents, I deliberately filled it with garbage
if the game locked it with `D3DLOCK_DISCARD`:

```cpp
HRESULT hr = ProxyInterface->Lock(OffsetToLock, SizeToLock, ppbData, Flags & ~D3DLOCK_DISCARD);
if ( SUCCEEDED(hr) )
{
	if ( Flags & D3DLOCK_DISCARD )
	{
		memset( *ppbData, 0xFF, m_bufferSize );
	}
}
return hr;
```

Much to my relief, the result was more or less what I expected:

<p align="center">
<img src="{% link assets/img/posts/scarface/garbage-vertices.png %}">
</p>

Graphics were once again broken, which proves that game locks buffers with a `D3DLOCK_DISCARD` flag **and**
expects the contents not to be thrown away! For me that's a satisfactory enough proof of API misuse,
and a proof that removing this flag is not a workaround, but a correct fix.

What about PIX and dgVoodoo? The latter is a wrapper, so I can imagine it not emulating the behaviour of
the discard flag. PIX on the other hand is a graphics debugger, and in order to be able to capture the frame
it likely preserves past and current buffer's contents. While I can't prove it for certain, I wouldn't be surprised
if PIX just ignored this flag overall.

Are we done? Well...

# Part two -- performance

I could technically finish here, as the most important issue has been fixed, and the game is playable.
However, something was still off...

<p align="center">
<img src="{% link assets/img/posts/scarface/EUIv96BXsAAknUi.jfif %}"><br>
<em>I have a i7-6700K and GTX 1070, mind you.</em>
</p>

My PC is nowhere close to "bad", yet I was unable to maintain stable 60 FPS -- in the area presented on a screenshot,
I in fact got consistent 40-45 FPS. To say it's "terrible" would be an understatement.

However, looking up process affinity in Task Manager reveals something... interesting:

<p align="center">
<img src="{% link assets/img/posts/scarface/ET-nl3yX0AI6fdl.png %}"><br>
<em>It is like this on every game launch.</em>
</p>

Looking into the game's code again, it seems like the game voluntarily sets itself to run on only one core.
Why? I don't know for sure, but I theoretized that this might have been a workaround for the aforementioned
graphics corruption bug. Recall that people claimed this bug occurs only on multicore CPU's --
and so I theoretized perhaps in WinXP days limiting CPU affinity to one core "fixed" this issue,
and on newer systems it stopped being the case due to changes in how drivers work (eg. if WDDM drivers
manage buffers out of process, then affinity settings would not affect them). It's a very long shot,
but hey -- [we've seen this before already]({% post_url 2018-07-07-farcry-d3d9-bug %}).

Remove this code so game runs on all cores, and sure enough -- it's smooth as butter now.
It also does not seem to have any visible race conditions (which could have been "hidden" by setting CPU affinity),
because people tested it for hours and encountered no crashes or new bugs.

<p align="center">
<img src="{% link assets/img/posts/scarface/EUIv_AYWsAY_TWB.jfif %}"><br>
<em>Now we're talking.</em>
</p>

On top of that, I also identified the game was creating the D3D device with a multithreaded flag -- which is said
to degrade performance, and was absolutely unneeded for the game. Another performance gain!

# More performance

Is there more we can do? Turns out, yes.

When testing these fixes, [aap](https://github.com/aap) observed that the game ran really poorly on his PC -- technically,
it was full speed, but when driving around it'd hitch a lot. I also observed the same when debugging the game
on a virtual machine, sometimes having the game pause for seconds at a time!

I checked it in a debugger, and much to my surprise, during those hitches the game spent a lot of time...
releasing buffers! I fired up PIX and immediately noticed how happy the game is to create new buffers (and thus
unload old buffers) when driving around:

<p align="center">
<img src="{% link assets/img/posts/scarface/EUMgDBOXsAEvyCb.png %}"><br>
<em>Load it all.</em>
</p>

This is really unhealthy. There is no reason the game can't reuse buffers instead of throwing them away
and creating new ones. Luckily, the game has only two types of vertex buffers (static buffers in a managed pool,
dynamic buffers in a default pool), and one type of index buffer,
which makes the buffer cache relatively simple to implement. It looks like this game might **really** need it.

I settled on implementing a simplistic cache with the following behaviour:
* On creating buffers, try to find a buffer of matching type and size.
* If it doesn't exist, check if there is a bigger buffer of matching type.
  However, to prevent unnecessary hogging of resources, only test for buffers at most twice as big as requested.
* If not found, create a new buffer.

I implemented the cache and ran the same test -- it was really smooth in comparison for both me and aap!
PIX graphs also looked much, much better now:

<p align="center">
<img src="{% link assets/img/posts/scarface/EUOB65VXkAAzzAO.png %}"><br>
<em>Reuse it all.</em>
</p>

With this fix implemented, I finally was satisfied with the state of the game.
Few final touches, and **SilentPatch for Scarface** is good to go!

# Changelog and download

Aside from the "exciting" fixes presented above, I also implemented a few more "boring" changes.
The full changelog is as follows:
* Game-breaking graphical corruptions have been fixed, making the game playable on modern multicore machines
* Allowed the game to use all CPU cores (instead of locking to one core), dramatically improving performance
* Removed an unneeded multithreaded flag from the D3D device, possibly improving performance slightly
* Introduced a cache for some D3D resources used by the game, dramatically reducing the amount of stutter when roaming around the city
* Made the game list all selectable resolutions instead of a cherry picked list
* Moved the game's settings from the Registry to settings.ini in the game directory - this resolves possible issues with saving settings

Head to *Mods & Patches* via the button below to download SilentPatch for Scarface.
But before you do, **stick with me for a bit longer!**

<a href="{% link _games/scarface.md %}#silentpatch" class="button" role="button" target="_blank">{{ site.theme_settings.download_icon }} Download SilentPatch for Scarface</a> \\
Upon downloading, all you need to do is to extract the archive to the game's directory and that's it!
When asked whether to overwrite files, select Yes.

# Announcing Patreon -- a new way to support me

Over the past few months, I've
[launched]({% post_url 2019-12-28-silentpatch-corona-update %})
[several]({% post_url 2020-01-18-silentpatch-yakuza-kiwami-2 %})
[new]({% post_url 2020-03-07-silentpatch-bully-r3 %})
[releases]({% post_url 2020-03-17-silentpatch-aquanox %}),
and I tried to be fairly active on the emulation scene. As I became increasingly productive over this time,
[some people](https://twitter.com/__silent_/status/1242377081926934533) wished to see a new way to support my work,
since GitHub Sponsors -- although very nice -- has too much friction for some people to start using.

With this in mind, together with the release of SP for Scarface I'm starting a Patreon campaign!
Fear not, paid early releases are not happening -- as of now, perks include:
* Place in credits for every release
* Access to my personal to-do list (with the ability to comment/suggest!)

This is not much, but depending on the response I might end up modifying the rewards a bit.

<a href="https://www.patreon.com/bePatron?u={{ site.theme_settings.patreon }}" class="button patreon" role="button" target="_blank"><i class='fab fa-patreon'></i> Become a patron</a>

Thank you!

***

For those interested,
full source code of the patch has been published on GitHub, so it can be freely used as a point of reference: \\
<a href="https://github.com/CookiePLMonster/SilentPatchScarface" class="button github" role="button" target="_blank">{{ site.theme_settings.github_icon }} See source on GitHub</a>