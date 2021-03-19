---
layout: post
title: New vehicle animations in SilentPatchSA
excerpt: Some details people asked about.
redirect_from: "/2018/05/27/vehicle-anims-silentpatch.html"
tags: [Progress Reports]
---
A week ago, new builds of SilentPatch were released (you can grab them from [Mods]({% link _games/gta.md %}) section). Looking at the changelog, one of the features added to San Andreas was:

> Several vehicles now have extra animated components: Phoenix hood scoops, Sweeper brushes, Newsvan antenna, radars on several boats, extra flaps on Stuntplane and Beagle

While it is explained pretty clearly what features were added, some people asked for more specific info and/or a showcase of those. Thus, I guess a follow up is not too much to ask for...

<div align="center" class="video-container">
<iframe src="https://www.youtube.com/embed/ZLvFfPGG32o" frameborder="0" allowfullscreen></iframe>
</div>

This clip shows all new animations added to the game in Build 29. As usual when adding something which is not purely a fix, I've been asked if those really belong to the 'patch and if they are custom.
Let me briefly go through all added features and explain the choices behind them, then:

- Phoenix hood scoop -- present in Vice City, absent from San Andreas. Stock Phoenix still has all necessary parts as separate components
(they received new names, as with all other "misc" components on vehicles), so this is a port of Vice City code to San Andreas[^1].
- Sweeper brushes -- code not present, but once again, parts are separate components. However, this time they have been named incorrectly (*misca* and *miscb* instead of *misc_a* and *misc_b*),
so SilentPatch corrects the names from code **and** animates them on its own.
- Newsvan antenna -- code not present, antenna is a separate (and properly named) component. I have tried several other animations (including pointing towards a fixed point),
but ultimately what is shown on the vid was decided as the best choice.
- Extra ailerons on Beagle -- bottom part of the wing was not animated, yet it's a separate component. SilentPatch mirrors its rotation from the aileron placed on the tail wing.
- Extra ailerons on Stuntplane -- top wing has two extra not animated components. SilentPatch mirrors their rotation with ailerons from the bottom wing.
- Rotating radar on Reefer, Tropic, Predator -- in this case, code for this animation was finished and fully working, but components were not properly named (all three named it *boat_moving*, while the game expected *boat_moving_hi*) so SilentPatch only corrects that from code.
I am not sure if there are any boats which did not have this broken :)

In addition to those, Build 30 will feature one[^2] more additional animation -- functional ladder on Firetruck used in End of the Line:

<p align="center">
<video preload="auto" autoplay="autoplay" loop="loop" style="max-width: 90%; height: auto;">
  <source src="https://i.imgur.com/sLN1J3E.mp4" type="video/mp4">
</video>
</p>

So what went wrong here? This ladder is supposed to swing left and right. However, turns out the code also supports moving it up and down, but the controls for that were disabled.
Re-enabling them made the feature work fully correctly! Now, right analog stick on the gamepad or Num2/Num8 keys (or whatever you have bound for hydraulics) will move the ladder up,
and it slides back down on its own when user lets go of it.

As always, there is **no** ETAs on Build 30, so just sit back and follow the progress :)

[^1]: A minor implementation issue has slipped into Build 29 -- this component does not animate when reversing, while it does in Vice City. This has been fixed for Build 30.
[^2]: That's assuming I don't find any more vehicles with components which could be animated, but I'm pretty sure I went through them all by now.
