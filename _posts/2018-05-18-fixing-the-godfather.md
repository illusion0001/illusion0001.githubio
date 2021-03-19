---
layout: post
title: "Fixing game-breaking issues in The Godfather: The Game - Part 1"
feature-img: "assets/img/posts/godfather/the-godfather-still.jpg"
thumbnail: "assets/img/posts/godfather/thumbnail.jpg"
image: "assets/img/posts/godfather/thumbnail.jpg"
excerpt: "Game crashing on modern systems? Let's see what we can do about it."
date: 2018-05-18 20:00:00 +0200
twitter: {card: "summary_large_image"}
redirect_from: "/2018/05/18/fixing-the-godfather.html"
tags: [Articles]
---
TL;DR - if you are not interested in an in-depth overview of what was wrong with the game and how it was fixed, just go here to grab SilentPatch for The Godfather: \\
<a href="{% link _games/godfather.md %}#silentpatch" class="button" role="button" target="_blank">{{ site.theme_settings.download_icon }} Download</a>

Upon downloading, all you need to do is to extract the archive to game’s directory and that’s it! If you have movies removed or renamed, don’t forget to restore them, since they will work just fine now.

This article is split in two parts:
* [Part 1: Prelude to fixing]({{ site.baseurl }}{% post_url 2018-05-18-fixing-the-godfather %})
* [Part 2: Implementation]({{ site.baseurl }}{% post_url 2018-05-18-fixing-the-godfather-pt2 %})

Prelude to fixing
=================

The Godfather: The Game is a 2006 game developed by [EA Redwood Shores (Visceral Games)](https://en.wikipedia.org/wiki/Visceral_Games). It initially released on PC, Xbox and PlayStation 2, later followed with additional releases on PlayStation Portable, Xbox 360,
Wii and PlayStation 3. The game is designed as an open world action-adventure around the plot of The Godfather movie. It was received relatively well.

However, PC version of the game has one major flaw which surfaced only several months after game’s debut in March 2006. At that time, the most popular Windows version was Windows XP
(though the game is said to support Windows 2000 too), with Windows Vista not reaching its public release until nine months later, in January 2007.

Upon launching the game, users are greeted with a copyright screen, followed by two movies. On Windows XP they get to see this:

<p align="center">
<img src="{{ site.baseurl }}/assets/img/posts/godfather/image3.png"><br>
<em>Yes, this is a virtual machine. You get the point, though.</em>
</p>

...while on Windows Vista or **any** newer system (including Windows 10, of course) movies look more like this:

<p align="center">
<img src="{{ site.baseurl }}/assets/img/posts/godfather/image6.png"><br>
<em>This is Windows 10, but I assure it’s just the same on Vista, 7, 8 and 8.1.</em>
</p>

Basing on countless workarounds and compatibility shims people attempted to use, all of which unsuccessful, there is **absolutely no way to get movies to work, at all**.
Forced compatibility modes, windowed modes, D3D wrappers - nothing fixes the issue. Well, it is possible to enable a certain compatibility shim to prevent the crash - however,
while the crash is gone and audio works fine, there is absolutely no video output, ever.

So how have people been dealing with this issue until now? Look no further than [game’s article on PCGamingWiki.com](https://pcgamingwiki.com/wiki/The_Godfather:_The_Game):

<p align="center">
<img src="{{ site.baseurl }}/assets/img/posts/godfather/image19.png">
</p>

That’s nasty. With movies removed, game works just fine but you get to lose FMVs and clips for good - surely you are missing out parts of the full experience.
So, looks like there is no win-win situation. Either crash, or have no movies. Bad luck.

We don’t have too much information about the issue - we know it’s movies causing the crash, but that’s about it. However, we have a key to a successful fix - issue is totally **reproducible**,
so proper research and debugging can be performed.

What is going on with movies anyway?
====================================

First off, let’s see what kind of movies game uses. All clips in the game are stored as **VP6** files, a proprietary format which seemed to have been EA’s format of choice for videos in their mid-2000’s games.
While this is not relevant for further analysis, a few pieces of information regarding the format specs can be found here:  
<https://wiki.multimedia.cx/index.php/Electronic_Arts_VP6>

The Godfather is not the only game to use this format - several Need for Speed titles, as well as The Godfather II and Dead Space (both developed by the same studio) also use VP6.
However, there is one important difference between GF and all those other titles - **their VP6 movie players work just fine**!

No point in dragging this topic further, so let’s just say movies themselves turned out to be just fine. External media players can play them just fine:

<p align="center">
<img src="{{ site.baseurl }}/assets/img/posts/godfather/image14.png">
</p>

Other games using VP6 also play Godfather’s movies fine, while Godfather still crashes when trying to play their movies. Thus, files themselves are not the issue - the issue lies somewhere else.

Debugging the crash
===================

Having a well documented database is crucial for efficient debugging when source code is not available, so at this point it was worth spending some time gathering known information about game’s internals.
Two important key points were quickly identified:

* Game’s VP6 decoder is at least partially shared with Need for Speed: Carbon, which is also a mid-2000’s game from EA. Wii version of the game somehow has symbols left over,
so we can use those to name most of VP6 decoding functions in Godfather.
* It became apparent that the game uses [RenderWare](https://en.wikipedia.org/wiki/RenderWare) (version 3.7) for graphics.
This is actually very good info, since all RW experience I have gained throughout the years of working with GTA III, Vice City and San Andreas could be useful at a later point (spoiler: it was crucial).
These days we have symbols for huge parts of their binaries (thanks to mobile ports leaving them over), so literally any RW function used in Godfather can be indexed and properly named.

With this info applied, it is obvious that the crash is actually a **software triggered breakpoint** coming from an assertion:

<p align="center">
<img src="{{ site.baseurl }}/assets/img/posts/godfather/image17.png"><br>
<em>REAL_abortmessage triggers a breakpoint internally, which means the “crash” we have been encountering is a controlled “something is wrong, you need to investigate asap”
debug measure which somehow found its way into the shipping build.</em>
</p>

The message says we are out of frames. That’s where things are getting interesting - it’s obviously game’s own code, why would it suddenly stop working on Vista?
Maybe a memory corruption which remained undetected on older systems? Or maybe game is exploiting some undocumented feature which got changed in a new OS and code started failing?

To find out what’s wrong, flow of execution between games running on Windows XP and Windows 10 needs to be compared.
Thankfully, it’s possible to remotely debug the game running on a XP virtual machine - the catch is, the newest Visual Studio working on XP is 2010,
so remote debugging had to be performed simultaneously from two different environments. Armed with two separate Visual Studios and stepping through both games, they finally started behaving differently:

<p align="center">
<img src="{{ site.baseurl }}/assets/img/posts/godfather/image2.png"><br>
<em>This is almost like yin and yang - except here one of them is correct and the other is not.</em>
</p>

What can we read from this? Game running on Windows XP follows the order of actions as expected:

```GetFrame → ReleaseFrame → GetFrame → ReleaseFrame → ...```

while when running on Windows 10 game seems to keep grabbing new frames without releasing anything:

```GetFrame → GetFrame → ...```

The game has a limited amount of frames it can grab at once, so at some point **BAM!** -- game runs out of frames and triggers a fatal error.

Why is the game not releasing frames? We can spot it in the function responsible for updating movie player state (functions and variables named basing on NFS Carbon symbols as well as guessing and observations):

<p align="center">
<img src="{{ site.baseurl }}/assets/img/posts/godfather/image10.png">
</p>

Apparently parts of the code are not called when something goes wrong during movie player initialization. You could ask - why is the code skipping only **some** parts of movie player logic
instead of skipping it all? This most likely was not an original intent, but it’s not unlikely this was never properly tested (because initialization was very, very unlikely to fail on XP)
and an implementation flaw eventually slipped under the radar.

Next, we need to figure out what causes movie player initialization to fail. Finding an initialization function proved to be fairly easy, and it provided critical information:

<p align="center">
<img src="{{ site.baseurl }}/assets/img/posts/godfather/image9.png">
</p>

Now we see what really is used to play movies - it’s DirectDraw7! Granted, it’s an ancient graphics API, but it still doesn’t explain the failure. Answer to that lies a few lines below that:

<p align="center">
<img src="{{ site.baseurl }}/assets/img/posts/godfather/image15.png"><br>
<em>Caps means capabilities, not headwear.</em>
</p>

Two device capabilities tested here are `DDCAPS_OVERLAY` and `DDCAPS_OVERLAYFOURCC`. The first one is a flag indicating overlay surfaces are supported,
the other one indicates whether overlay surfaces are capable of converting colour space (so, for example, YCbCr surfaces can be rendered correctly with no need for a conversion first).
In order for initialization to succeed, both features need to be supported, which means we don’t support at least one of those two.

It's possible to check if either of those cause an initialization failure by strategically putting a few breakpoints in the code. Sure enough,
launching the game this way reveals one of the flags is not supported:

<p align="center">
<img src="{{ site.baseurl }}/assets/img/posts/godfather/image20.png"><br>
<em>We hit this code path, which means a flag check has failed.</em>
</p>

To verify, let’s consult one of the tools from DirectX SDK - DirectX Caps Viewer:

<p align="center">
<img src="{{ site.baseurl }}/assets/img/posts/godfather/image21.png">
</p>

On my machine `DDCAPS_OVERLAY` is unsupported - just like we observed in game! This is where things are getting interesting,
but in order to fully understand the issue (as well as understand why overlays are used for movie player in the first place),
we need to explain what exactly overlay surfaces are and why are they so problematic with modern systems.

What are overlay surfaces? The only pages left on MSDN regarding overlays explain them in way too much detail which is not relevant here,
so let’s define them (less accurately, but accurate enough for this paper) as follows:

> An overlay surface is a surface that can be displayed on top of a primary surface, without altering the surface underneath it.
> Overlay operation is performed on hardware,and thus such surfaces are efficient for using with high frame rate video content.
> Additionally, they can have a format different than the primary surface.

Additionally, MSDN **used to** have a fairly simple overview of overlay surfaces - we can still refer to it thanks to web archiving:  
<https://web.archive.org/web/20110314013849/https://msdn.microsoft.com/en-us/library/ee491472.aspx>

Overlays are convenient to use for movies, because the responsibility of handling high framerate material gets shifted to the display driver.
However, I think the main reason they were used in Godfather was their ability to directly display data in non-RGB formats - VP6 movies are decoded to YUY2 format which will be explained in detail later.
Displaying high framerate material couldn't have been a reason, as every video in the game is 30 FPS!

This all sounds good so far, but why would overlays cause issues on modern systems?

One of the features Microsoft dropped in Windows Vista are... DirectDraw hardware overlays! According to Microsoft, DirectDraw primary surface is not lockable anymore because DWM
needs an exclusive access to those - and it is mandatory to lock surfaces in order to update overlays (thus, if we can’t lock the surface, we can’t update it either).
Security can also be a factor - hardware overlays could be drawn over any window (even a log on screen). With this in mind,
dropping their support could be considered a security measure and thus align with a lot of other changes introduced in Windows Vista.

Now we see that both OS and the game are at fault here - video player would not work as is no matter what developers did,
but on the other hand it wasn't supposed to crash - that is caused by an improper failure handling (goes to show how important accounting for failure is - something which “works for me”
today may not necessarily work next week).

What about a compatibility shim mentioned earlier? We can enable it for The Godfather and thus prevent the game from crashing, with movies not displaying at all (audio works fine, though).
Said shim is called... *EnableOverlays*! As you may have guessed from the name, it overrides certain device capabilities, reporting to the game that overlays are supported
(namely, makes `DDCAPS_OVERLAY` report as supported) - thus the game gets tricked and never reaches the code which caused it to crash.

Ideas for a fix
===============

In order to tackle this issue, something needs to be done to either replace or fix deprecated functionalities. Before coming up with a proper fix, there were several failed attempts.
For the sake of completeness, here listed are all the attempts:

1. Fixing game’s DirectDraw usage -- in theory, there is a slim chance that the game may be misusing DirectDraw overlays and it is possible to come up with either a generic or a game specific fix for this.
   Here, I’d like to direct you to a blog post from 2007 about a very similar issue:  
   <https://mhaggag.wordpress.com/2007/04/11/directdraw-overlays-on-vista-dderr_outofcaps/>\\
   While this idea may have worked with Vista (as blog’s author states, overlays still sort of work for him when DWM is disabled),
   it’s more or less useless for Windows 8/8.1 and 10, since there is no straightforward way to disable DWM there. That said, applying the fix mentioned in said post does nothing (at least on Windows 10).
   Moreover, not even a single “DirectDraw overlays” sample I found online works with Windows 10, so the chances of this being a game bug are close to zero.

2. Backporting Godfather II movie player to Godfather -- that idea was dropped very quickly - Godfather II runs on a different graphics engine and most likely has a different VP6 decoder.
   Since decoder itself works fine in the first game too, this would virtually mean rewriting parts of the code from scratch and reversing GFII code does not have any measurable benefits.

3. Generic DirectDraw -> D3D9Ex wrapper with hardware overlays -- the initial idea for the fix was to come up with a generic wrapper, making use of hardware overlays reintroduced in D3D9Ex for Windows 7.
   While this idea could potentially work out for windowed applications, GF prefers exclusive fullscreen mode (and changing that would be too intrusive and potentially prone to unwanted side effects)
   It means that: only one D3D device can draw to the window **and** only one exclusive window can be displayed. \\
   Because it's impossible to obtain D3D device coupled to game’s window in a generic manner nor “upgrade” it from D3D9 to D3D9Ex (they have minor differences in behaviour which makes it
   annoyingly complicated to generalize as a drop-in replacement), this idea sadly can't work. At least it produced valid output in PIX graphics debugger...

4. Generic DirectDraw -> D3D9 wrapper with a new window -- did not work for the same reasons as the previous idea. Such approach works fine in windowed mode, but not in exclusive fullscreen.

5. Game specific DirectDraw -> RwD3D9 wrapper -- as we know, generic fix is not really a viable solution here. Therefore, game specific fix allows for much more freedom when it comes to
   selecting ways of approaching the issue.  
   It was possible to use game’s D3D9 device directly, however RenderWare has its own wrapper around D3D9, called RwD3D9. Using it will make the fix coexist nicely with existing rendering in the game,
   namely extra layers of render state caches RenderWare builds on top of D3D9.

By now it should be pretty obvious that attempts #1 to #4 failed miserably. #5 is in fact most likely the most complex solution of them all, but was still worth a try.
Indeed, thanks to RenderWare functions being known (so tracking them in game’s executable was relatively straightforward) and RenderWare being well researched in general
(thank GTA modding community for this), it didn’t take long before the first results could be seen.

In order to emulate DirectDraw overlays via RwD3D9, two separate tasks had to be solved:

Custom interface implementation
-------------------------------

The easiest way to gut DirectDraw out is to provide the game with a custom implementation of DirectDraw interfaces. Two interfaces had to be implemented from scratch - that is,
*IDirectDraw7* and *IDirectDrawSurface7*. Thankfully, their usage in game is pretty straightforward and could be summarized as follows:

1.  Game creates a *IDirectDraw7* interface.
2.  Game queries the interface for current display mode dimensions (and ignores the result...).
3.  Game creates a primary surface for displaying movies.
4.  Game queries for hardware overlay capabilities (this is where the stock game fails).
5.  Game queries for an availability of a YUY2 texture format.
6.  Game creates an overlay surface in YUY2 format, with dimensions corresponding to movie dimensions (all stock movies are 640x480, but technically game can support any dimensions).

After initialization is done, each movie frame goes through the process as follows:

1.  Game locks the overlay surface.
2.  VP6 decoder outputs a new YUY2 frame to the overlay surface.
3.  Game unlocks the overlay surface.
4.  Game updates the overlay surface, overlaying primary surface.

However, implementing those is not enough - if we try to display the surface via normal D3D9 calls at any point, they will not show. To fix that, a second task had to be solved...

Custom overlay render queue
---------------------------

*Overlays* truly stand for their name and have a trait we cannot easily replicate in Direct3D - no matter when they are updated, they are eventually rendered above anything else.
We could solve this by ensuring nothing else gets drawn after updating our newly created RwD3D9 overlays, however that would require an unknown amount of patches in game’s code itself
and ends up being unfeasible. A more reliable solution is to come up with a custom overlay renderer, hooked into the game just before it presents the final framebuffer data.
That’s **exactly** how overlay software (like Steam Overlay) works - the difference is that they hook D3D9, while we have the luxury of being able to tailor a game specific RwD3D9 hook wherever we see fit.

RenderWare based games are required to render everything within camera update calls (think of cameras as of render targets, so update calls essentially specify we want to render to this specific target) - 
and after that’s done, contents of the camera are being presented.
We can hook game’s *RwCameraShowRaster* function (which presents the final framebuffer on screen) to also draw our overlays, like so:

```cpp
if ( RwCameraBeginUpdate( rwCamera ) )  
{  
        for ( auto entry : m_queue )  
        {  
                RenderOverlay( entry );          
        }  
        RwCameraEndUpdate( rwCamera );  
}  
RwCameraShowRaster( rwCamera ); // Original call we are overwriting
```

Queue gets populated by *UpdateOverlay* calls issued by the game. With both parts in place...

...we have reached the end of Part 1! If you are not too overwhelmed by technicalities already, follow the links below to jump to [Part 2]({{ site.baseurl }}{% post_url 2018-05-18-fixing-the-godfather-pt2 %}). Otherwise, come back soon!