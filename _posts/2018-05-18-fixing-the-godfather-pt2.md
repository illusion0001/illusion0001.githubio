---
layout: post
title: "Fixing game-breaking issues in The Godfather: The Game - Part 2"
feature-img: "assets/img/posts/godfather/the-godfather-still.jpg"
thumbnail: "assets/img/posts/godfather/thumbnail.jpg"
image: "assets/img/posts/godfather/thumbnail.jpg"
excerpt: "Now we know what to do, so let's do it."
date: 2018-05-18 20:10:00 +0200
twitter: {card: "summary_large_image"}
redirect_from: "/2018/05/18/fixing-the-godfather-pt2.html"
tags: [Articles]
---
This article is split in two parts:
* [Part 1: Prelude to fixing]({% post_url 2018-05-18-fixing-the-godfather %})
* [Part 2: Implementation]({% post_url 2018-05-18-fixing-the-godfather-pt2 %})

If you haven't read [Part 1]({% post_url 2018-05-18-fixing-the-godfather %}) yet, I strongly encourage to go back and read it first before proceeding with this post.

Implementation
=================================

In Part 1, we have researched the crash and figured out the exact cause of it. As it was listed, there were a few possible approaches to fixing the issue,
but finally only one of them ended up being viable. Now, with most of the code in place, we can check if anything shows up...

Blue rectangle - the breakthrough
=================================

<p align="center">
<img src="{% link assets/img/posts/godfather/image18.jpg %}">
</p>

This is a screenshot from the very first attempt of testing a RwD3D9 wrapper which gave visible results. While it doesn’t seem like it, this blue rectangle is in fact a breakthrough -
it’s drawn using newly written wrapper, which means **we can use it to draw anything we want to**. In other words, this simple example basically proves the idea works out and can be
polished further to display movie data instead of a static image!

Since we are now using D3D9, it is **not** possible to directly display a non-RGB image. However, it is possible to write a shader which outputs RGB data when given a non-RGB texture -
and this is exactly what we are going to do here.

First, replacing static blue fill with data provided by VP6 decoder (without specifying a shader or anything fancy) gives us a first view on the movies. Getting there!

<p align="center">
<img src="{% link assets/img/posts/godfather/image5.png %}"><br>
<em>The very first time movies are displayed on Windows 10 in game (note this is 640x240, for the time being displayed in the corner and not stretched at all).</em>
</p>

Let’s identify and isolate issues we can see here:

1.  Greenish appearance - that’s what typically happens when you display YUV data as RGB - so we can see that in-game decoder indeed provides us with non-RGB data
2.  Duplicated image - caused by interpreting 16-bit wide pixel data as 32-bit
3.  Weird darker stripe - linear filtering failure... very non-obvious at first

We’ll explain 3. at a later point - to understand what’s going on with 1. and 2. let’s see at what data **really** is provided by in-game VP6 decoder.

Image formats - YUY2, YUV, RGB
------------------------------

YUY2 is a 16-bit 4:2:2 format, which is a variation of YUV color format typically used to encode videos. With YUY2, each 32-bit value defines a pair of pixels. Data is laid out as presented:

<p align="center">
<img src="{% link assets/img/posts/godfather/image16.png %}">
</p>

Every pixel has its own Y (luminance) value, with U and V values shared between two neighbouring pixels.

Since to properly decode a pixel we need information from its neighbour (later called “left” and “right” pixels), it’s unfeasible to treat this data as a 16-bit
texture with two 8-bit channels (like L8A8) - what fits this case better is treating texture as a 32-bit 8888 texture with half the width. Then, a pixel shader can use
different components of the source pixel basing on whether the output pixel is a “left” or “right” pixel, like so:

```
Y = fmod(In.UV.x, 2.0) < 1.0 ? sample.b : sample.r
```

Now we see both issues are caused by improper interpretation of texture data - interpreting it as a 32-bit 640x480 image made it show two subsequent scanlines in the same row,
while overall green appearance was caused by interpreting the data as RGB.

With remaining issues identified, fixing them was only a matter of writing a correct YUY2 -> RGB shader and some trial and error around a few other quirks. 
Soon after, **movies finally started showing up correctly**!

<p align="center">
<img src="{% link assets/img/posts/godfather/image23.png %}"><br>
<em>It lives! Still in 640x480, but otherwise fully working!</em>
</p>

This point was finally reached after five attempts and around 2 months of prototyping - finally, both in-game and intro movies show up!

Watching The Godfather trailer from within the game has certain charm to it... (note - at this point in-game movies had a minor filtering issue
which shows on this screenshot as fake “aliasing” around the edges)

<p align="center">
<img src="{% link assets/img/posts/godfather/image4.png %}"><br>
<em>I’m gonna make him an offer he can’t refuse.</em>
</p>

New features, new problems
==========================

On the first glance, everything seems fine. However, we don’t really want to display videos in a tiny 640x480 rectangle, right? I’ve created three different display modes for overlay render queue:

1.  Stretch - stretches the video to fill the entire screen, ignoring aspect ratio. This matches the original behaviour on Windows XP. but is not too visually pleasing -
    640x480 videos stretched to 1080p don’t look right at all.
2.  Letterbox - stretches the video to fill the entire screen, preserving original aspect ratio by adding black horizontal or vertical bars at screen edges. This matches the behaviour of most video players.
3.  Fill - stretches the video to fill the entire screen, cutting off edges of the video. This works best for movies with black borders included on them (like The Godfather trailer shown earlier),
    but can result in important information being cut off in other clips.

While coding those was trivial, upscaling videos revealed a previously overlooked issue. If you take a look at this frame from an upscaled intro video (here rendered in Letterbox mode,
so it ends up being rendered as 1440x1080), you can notice it doesn’t look quite right...

<p align="center">
<img src="{% link assets/img/posts/godfather/image11.png %}"><br>
<em>Artifacts are not obvious, but are clearly noticeable when the logo is fullscreen</em>
</p>

What happens here? This clip is being a victim of a fairly severe problem coming from **non-linear texture scaling**. We can clearly see this phenomenon when providing the game with a regular pattern.
In this example, I have prepared a 640x480 texture consisting of a 1px white column, followed by 1px black column - repeated. Then, this texture was drawn as 1440x1080 on screen:

<p align="center">
<img src="{% link assets/img/posts/godfather/image13.png %}">
</p>

Notice something wrong? One doesn’t need to zoom in to notice how irregular this pattern is, even though the source texture was completely regular! Why is this happening?
Take a look at the factor source texture had to be upscaled:

```1440 / 640 = 2.25```

That’s where term **non-integer scaling** comes from - texture needs to be upscaled by a value which is not an integer!
This means it is impossible for an upscaled image to be a perfect representation of the source image - for that, each 1 pixel from the source image would have to be represented
as 2.25 pixels on an upscaled image!

Usually, those scaling issues don’t result in issues as jarring as the ones we have. Issues are so severe in this case because of an unusual texture data format, mentioned earlier.
Usually, an image upscaled this way would have some of the pixels duplicated, but they are still left in correct order. However, in our case each 32-bit pixel from the source image
corresponds to two rendered pixels (remember a shader code snippet mentioned earlier), so scaling leads to duplicating pairs of output pixels!
This causes them to display out of order - mountain shown on the Paramount logo makes it obvious:

<p align="center">
<img src="{% link assets/img/posts/godfather/image7.png %}"><br>
<em>Since we are displaying some pixels out of order, mountain becomes jaggy</em>
</p>

Solution - image preprocessing
------------------------------

As mentioned earlier, non-integer scaling would not be a noticeable problem if each 32-bit value from the source image corresponded to a single output pixel. Thankfully, we can achieve that easily!

The best solution is to preprocess image on the CPU, transforming it from YUY2 to YUV. Data doubles in size, but with 640x480 movies this is a non-issue. We can transform pixels in place
(that is, without having to allocate temporary memory) like this:

```cpp
while ( source != end )  
{
        const uint32_t yuy = *source++;
        *destination++ = (yuy & ~(0xFF0000)) | ((yuy & 0xFF) << 16);
        *destination++ = yuy;
}
```

After this transformation, data is laid out like this:

<p align="center">
<img src="{% link assets/img/posts/godfather/image12.png %}">
</p>

Now, each pixel from source texture corresponds to exactly one output pixel, so pixel shader does not need to check what “side” the pixel is on!
This solves all scaling issues we had and also allows to opt for bilinear filtering instead of nearest filtering, resulting in a more visually pleasing output image:

<p align="center">
<img src="{% link assets/img/posts/godfather/image22.png %}"><br>
<em>Perfect! Not stretched and with no filtering issues.</em>
</p>

Success! We can finally call it working correctly. Not only intros work fine - in game clips display just as well:

<p align="center">
<img src="{% link assets/img/posts/godfather/image8.png %}"><br>
<em>While these tutorial clips don’t have too much educational value, it’s good to see them work fine.</em>
</p>

It's time to see those movies in action!

<div align="center" class="video-container">
<iframe src="https://www.youtube.com/embed/tkCyBY5z5dc" frameborder="0" allowfullscreen></iframe>
</div>

Fixing other issues
===================

With movies fixed, the game is fully playable. However, since a generic approach did not work out and patch is going to be game specific anyway,
I included a few other bug fixes mainly to ensure other possible issues with getting the game to run smoothly are resolved (technical fixes always go first before gameplay related fixes!).
Those are:

1.  Game has been disallowed from writing to `HKEY_LOCAL_MACHINE` and stopped relying on registry keys to obtain installation path - this issue should never appear with properly installed games,
    however I encountered it when moving the game between different test PCs and operating systems. This took much more time to troubleshoot than I had hoped,
    so it’s not unlikely a regular user would get completely stuck if stumbled on the issue like this.
2.  Game log has been relocated to the game directory, so in case of a crash the game does not try to write to `C:\GF_Excpt.txt` anymore.
3.  Exposed FPS cap to the INI file - technically, this doesn't belong to a modification about fixing, but it was added in to replace the current method of changing FPS cap in Godfather.
    Now it's not needed to replace game EXE with a custom one, making it much less invasive than the old method.
4.  Added an option to skip intros - it's great we have them working again, but an unskippable 30 seconds long intro sequence can get annoying really quick.
    On the flip side, you still have plenty of movies in game, so it's not like all the fixing goes to waste.

Future plans
============

A few weeks since the time of release, current feature set seems to be working flawlessly (there isn't a single known instance of the patch not fixing things as intended),
however people seem to experience more issues with the game.

Namely, a relatively common complaint is that game tends to crash when picking the a player name. As of now the root cause of the issue is unknown, but it would be a no. 1 priority if an update is ever released.

Finale
======

That draws the end of it! Developing the fix was quite a journey, and I am hoping so was the post-mortem article documenting it. For those interested,
full source code of the patch has been published on GitHub, so it can be freely used as a point of reference: \\
<a href="https://github.com/CookiePLMonster/SilentPatchGF" class="button github" role="button" target="_blank">{{ site.theme_settings.github_icon }} See source code on GitHub</a> 

For the time being, enjoy a playable game and stay tuned for more fixes for various older games! **Next up: Far Cry water reflections not working properly on modern Windows versions.**
