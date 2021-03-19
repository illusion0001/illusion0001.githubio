---
layout: post
title: Finding a right DLL to hook is harder than you think
excerpt: Know your KnownDLLs.
redirect_from: "/2018/05/13/know-your-dlls.html"
tags: [Articles]
---
Ironically, the biggest challenge behind [SilentPatch for Far Cry]({% link _games/far-cry.md %}#silentpatch) was to find the right DLL to proxy in order to make Ultimate ASI Loader work fine.
Most games work fine with `dinput8.dll`, but since Far Cry is split into around 47.000 separate libraries (not literally), none of the DLLs supported by UAL worked fine.
Finding a right DLL took much more time than I had hoped, but I decided to add support for `wininet.dll`. All fine, ASIs load just when I want them to load, users are happy.

However, [not everyone was lucky enough to get it to work without issues](https://www.vogons.org/viewtopic.php?f=8&t=40913&start=160#p664655). What happened?

The initial hunch was reasonable -- it looked like an incompatibility with a retail version, since I only ever tested it with a Steam version. However, a 64-bit version didn't work either, and that one is identical
for both Steam and retail, since it was released as a post-launch patch.
Further research revealed the problem is not in the ASI itself -- UAL would not inject itself into the process at all! For some reason, my `wininet.dll` ended up being completely ignored by Windows when resolving dependencies for `CrySystem.dll`. One of possible causes could be Known DLLs. **Larry Osterman** explains those on his blog very well:

<https://blogs.msdn.microsoft.com/larryosterman/2004/07/19/what-are-known-dlls-anyway/>

Let's take a look at KnownDLLs in Windows 10 then:

![KnownDLLs from Windows 10]({% link assets/img/posts/known-win10.png %})

As suspected, `wininet.dll` is not there. However, what about Windows 7...?

![KnownDLLs from Windows 7]({% link assets/img/posts/known-win7.png %})

That's it! Since it's a Known DLL, Windows automatically assumes the DLL must be in *system32* and ignores any other possible DLLs.

The moral of the story is: always test stuff like this on multiple systems. My virtual machine with Windows 8.1 is now ready, to make sure I don't repeat this mistake in the future.

And as for SilentPatch for Far Cry, the most likely replacement candidate for an ASI loader is `version.dll` -- the update should arrive shortly.