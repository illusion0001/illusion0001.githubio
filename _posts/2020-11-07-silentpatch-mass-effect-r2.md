---
layout: post
title: "Second release of SilentPatch for Mass Effect"
thumbnail: "assets/img/games/bg/mass-effect.jpg"
feature-img: "assets/img/games/bg/mass-effect.jpg"
image: "assets/img/posts/mass-effect/comparison.jpg"
excerpt: "AI works as intended again."
twitter: {card: "summary_large_image"}
date: 2020-11-07 23:55:00 +0100
tags: [Releases]
---

Happy N7 day! Today, [Mass Effect Legendary Edition](https://twitter.com/masseffect/status/1325106163722162177?s=20) has been announced, and on top of that
we're releasing an update for SilentPatch for Mass Effect!

This time, the changelog is **very** short -- after a [report showing that the enemy AI breaks in the Virmire mission when SP is used](https://github.com/CookiePLMonster/SilentPatchME/issues/4)...

<p align="center">
<img src="https://user-images.githubusercontent.com/475132/98448081-87e9b100-20de-11eb-9c1f-2bd8716179a1.gif">
</p>

<p align="center">
<img src="https://user-images.githubusercontent.com/475132/98448155-05152600-20df-11eb-9702-45b1d9b84fcb.gif">
</p>

...Rafael Rivera investigated the issue and found an inconsistency in our implementation of `D3DXMatrixInverse` which mishandled NaN values.
With this fix in place, AI should be back to normal. Sorry for the inconvenience!


# Download

The modification can be downloaded in *Mods & Patches*. Click here to head to the game's page directly:

<a href="{% link _games/mass-effect.md %}#silentpatch" class="button" role="button" target="_blank">{{ site.theme_settings.download_icon }} Download SilentPatch for Mass Effect</a> \\
After downloading, all you need to do is to extract the archive to the game's directory and that's it! Not sure how to proceed? Check the [Setup Instructions]({% link pages/setup-instructions.md %}).

***

For those interested,
full source code of the mod has been published on GitHub, so it can be freely used as a point of reference: \\
<a href="https://github.com/CookiePLMonster/SilentPatchME" class="button github" role="button" target="_blank">{{ site.theme_settings.github_icon }} See source on GitHub</a>
