---
layout: post
title: "Fixing a completely random bug in Cxbx-Reloaded - Star Wars: KotOR"
excerpt: "Default does not have to mean zero. It can mean nothing."
date: 2020-10-20 23:05:00 +0200
tags: [Articles]
---

For the past two weeks, I've been fixing various issues in [Cxbx-Reloaded](https://cxbx-reloaded.co.uk/), the original Xbox emulator.
My mini-goal was to get a particular NASCAR game working, and [I succeeded in that](https://youtu.be/YVpLUdSjzZc),
getting the game from an unbootable state to very close to playable in around one week. Things were going smoothly until I got a report
that one of my changes breaks another game -- [Star Wars: Knights of the Old Republic](https://en.wikipedia.org/wiki/Star_Wars:_Knights_of_the_Old_Republic_(series)#Star_Wars:_Knights_of_the_Old_Republic).
I could not push a change that fixes the game I care about at the cost of breaking others, so I decided to look into it.

# The festival of colors

Sure enough, with my PR the game looked hopelessly broken:
<p align="center">
<img src="{% link assets/img/posts/kotor/release-optimizations.png %}"><br>
<em>It should not be this yellow. It should not be yellow at all.</em>
</p>

At the same time, I was told that this is some old bug that was resolved a while ago, and apparently, it came back.
I didn't know how it is possible so I assumed I introduced a regression, but then RadWolfie, one of the main Cxbx-Reloaded
developers, dropped a very important piece of information:
> Using a VS2019 build will produce a green overlay screen in a similar way yellow to an overlay screen.
> To verify this properly you need to use the VS2017 build.

This turned out to be true for my PR too -- the version compiled with Visual Studio 2017 did not have this artifact.
Even more curiously, neither did the Debug version compiled with Visual Studio 2019. A Release version with optimizations disabled
was glitched, but... it displayed a different color:
<p align="center">
<img src="{% link assets/img/posts/kotor/release-no-optimizations.png %}">
</p>

Since it's a graphical issue, I fired up PIX and inspected the frame. It didn't take long to find a fullscreen draw whose
colors correspond to the artifact visible in-game:
<p align="center">
<img src="{% link assets/img/posts/kotor/pix-yellow-frame.png %}">
</p>

That doesn't help much, though -- I identified the draw but not the reason why it's miscolored.
Since it started happening after an unrelated change, I suspected that some code might be reading past array bounds
and interpreting unrelated variables as color; however, a quick check with page heap proved this not to be the case.
It could have also been reading past array bounds of a global variable, but shuffling around the variables I modified in my PR
also did nothing. A change of approach was needed.

# Understanding push buffers

At this point, I decided to instead understand what are those draws and where they come from.
The sequence of calls there is quite specific -- the code opts to use a fixed-function vertex transformation,
then uses `DrawIndexedPrimitiveUP` to draw primitives from the memory pointer.
I could use this information to find such sequences of calls in the log file.

I did not find them, but instead, I found something else -- namely, push buffers.
I'm not knowledgeable enough about Xbox-specific Direct3D parts to be able to describe them properly,
but in principle, they are a concept nearly identical to an [immediate mode in OpenGL](https://www.khronos.org/opengl/wiki/Legacy_OpenGL).

With this info, I went to the code and disabled push buffers completely -- and sure enough, the tint is gone:
<p align="center">
<img src="{% link assets/img/posts/kotor/removed-push-buffers.png %}"><br>
<em>The letterbox is gone too.</em>
</p>

Back to the PIX frame -- why exactly is the draw producing such results? Checking the output mesh,
one of the input values were... suspiciously high:
<p align="center">
<img src="{% link assets/img/posts/kotor/weird-values.png %}"><br>
<em>Those are definitely invalid values.</em>
</p>

In the frame capture from a Debug version of Cxbx-R, those were zero -- so it's very likely they are the culprit.
Since we know they come from immediate buffers, I inspected the only function which could have submitted them -- `D3DDevice::SetVertexData4f`.

# Uncorrupting the data

By placing a few carefully crafted conditional breakpoints in this function, I was able to determine that for those draws the
game uses a set of 4 texture coordinates of two components each, but **updates only two of those texture coordinates**.
While this might sound counter-intuitive, I've been told that this again matches the behaviour of OpenGL immediate rendering,
which treats all those attributes as states -- setting them once persists them for next vertices,
so they are only updated when they need to be changed.

Knowing this, I can inspect the code which puts this immediate data on the final submitted buffer:

```cpp
for (unsigned int i = 0; i < dwTexN; i++) {
    *pVertexBufferData++ = g_InlineVertexBuffer_Table[v].TexCoord[i].x;
	if (TexSize[i] >= 2) {
		*pVertexBufferData++ = g_InlineVertexBuffer_Table[v].TexCoord[i].y;
		if (TexSize[i] >= 3) {
			*pVertexBufferData++ = g_InlineVertexBuffer_Table[v].TexCoord[i].z;
			if (TexSize[i] >= 4) {
				*pVertexBufferData++ = g_InlineVertexBuffer_Table[v].TexCoord[i].w;
			}
		}
	}
}
```

If a texture coordinate uses two elements, only two are placed on the buffer -- this matches what I could observe in PIX.
Some of the values in `g_InlineVertexBuffer_Table` were interesting, though:
<p align="center">
<img src="{% link assets/img/posts/kotor/texcoords.png %}">
</p>

Some of these values are obviously corrupted or uninitialized, but they are still submitted to the final buffer!
I again inspected all places in the code updating the IVB table, and other than `SetVertexData4f`, there was only one other place writing to it:

```cpp
// Is this the initial call after D3DDevice_Begin() ?
if (g_InlineVertexBuffer_FVF == 0) {
	// Set first vertex to zero (preventing leaks from prior Begin/End calls)
	g_InlineVertexBuffer_Table[0] = {};
```

It might seem like it zeroes the vertex data, but... not quite:
<p align="center">
<img src="{% link assets/img/posts/kotor/zero-not-zero.png %}"><br>
<em>I don't think those TexCoords are zeroed.</em>
</p>

What's going on here? Remember that `x = {}` is not initializing values to "zero". It initializes them via
[value initialization](https://en.cppreference.com/w/cpp/language/value_initialization).
Yes, for primitive types value initialization sets their value to zero, but for more complex types it initializes
them via their default constructor. Knowing that `TexCoord` is defined as `D3DXVECTOR4 TexCoord[4]` (D3DX again!),
it has a constructor defined by Microsoft:

```cpp
//--------------------------
// 4D Vector
//--------------------------
typedef struct D3DXVECTOR4
{
public:
    D3DXVECTOR4() {};
```

An empty constructor, doing nothing! Normally, brace initialization zeroes values,
but because of this empty constructor, they are still left uninitialized!

This explains all the differences -- compiling in Debug or Release, changing optimization settings, moving code around all
can affect the stale values on the stack, and thus change the effects of this code.

Since we cannot change the `D3DXVECTOR4` constructor, IMO the best fix is just to replace brace initialization with `memset`.
With this simple change, the game again looks like it's supposed to!
<p align="center">
<img src="{% link assets/img/posts/kotor/after.png %}">
</p>

# Conclusion

How to avoid such issues in the future? To be honest, it's not trivial because the code was not "wrong" per se. However,
in this specific case my conclusions are:
* `memset` is a relic from C, but sometimes it's still a good solution -- especially when used on primitive types or
trivially constructible structures. Sadly, it is sometimes easy to accidentally make the structure or class non-trivial while
still using `memset` and/or `memcpy`, which just leads to hard to troubleshoot issues.
* D3DX is evil and [this is not the first time it bit me]({{ site.baseurl }}{% post_url 2020-07-19-silentpatch-mass-effect %}).
One more reason for me to avoid it at all costs.

***

Everything presented here has been submitted back to Cxbx-Reloaded in a [pull request](https://github.com/Cxbx-Reloaded/Cxbx-Reloaded/pull/1981).
At the time of publishing this post, those changes have already been merged.
