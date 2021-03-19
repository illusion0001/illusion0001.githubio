---
layout: post
title: "Co-Driver Splitter - Co-driver calls on headphones"
feature-img: "assets/img/posts/codriver-splitter/codriver-splitter.jpg"
thumbnail: "assets/img/posts/codriver-splitter/codriver-splitter.jpg"
image: "assets/img/posts/codriver-splitter/codriver-splitter-full.jpg"
excerpt: "Make racing more immersive by separating co-driver from the rest of the sounds."
date: 2019-04-11 23:15:00 +0200
tags: [Releases]
---

Ever wanted to have Co-driver calls play through headphones while the other sounds play through speakers? Now you are able to without hacking around a convoluted speaker setup!

Co-Driver Splitter is a plugin for recent DiRT, GRID and F1 games splitting audio between multiple devices. Now Co-driver, Spotter and Commentator calls will be played through a Communications device,
while everything else plays over speakers!

<div align="center" class="video-container">
<iframe src="https://www.youtube.com/embed/S4psNp2mhUs" frameborder="0" allowfullscreen></iframe>
</div>

<div align="center" class="video-container">
<iframe src="https://www.youtube.com/embed/X3FQYK2GUuk" frameborder="0" allowfullscreen></iframe>
</div>

***

This is a modification I wanted to do for a long time. I wanted to have calls playing through headphones, with everything else left as is.
Some people claimed it's just impossible due to the way audio is handled on PC -- so I decided to give it a shot myself, and it turned out to be absolutely doable!

Without further ado, you can grab Co-Driver Splitter from here. **Make sure you read the note to know if the game you want to use Co-Driver Splitter with needs 32-bit or 64-bit libraries!** \\
<a href="https://github.com/CookiePLMonster/CoDriver-Splitter/releases" class="button github" role="button" target="_blank">{{ site.theme_settings.github_icon }} Download Co-Driver Splitter from GitHub</a>

# Setting up audio

You don't need to do much to make sure Co-Driver Splitter picks correct audio devices. All you need to do is to ensure that in Sound, your Default Communications Device is set to headphones,
while Default Device -- speakers or other device you want to play environment sounds through.

Example: \\
![Correctly set up audio devices]({% link assets/img/posts/codriver-splitter/audio-devices.jpg %})

# How does it work?

Co-Driver Splitter fools the games into thinking a 5.1 surround setup is used. However, Co-Driver Splitter downmixes the output back to stereo for all channels except the center channel.
Instead, center channel is routed to headphones! This creates a nearly perfect separation of communications and environment sounds.

Co-Driver Splitter is designed to be able to work as a wrapper around XAudio 2.7, XAudio 2.8 and XAudio 2.9. For XAudio 2.7, you may need to apply a few registry tweaks which I will detail soon.

***

If you are interested in getting Co-Driver Splitter to work with GRID or F1 games, check back here and on repository's wiki soon -- I'll be posting detailed set-up information there! \\
<a href="https://github.com/CookiePLMonster/CoDriver-Splitter/wiki" class="button docs" role="button" target="_blank">{{ site.theme_settings.docs_icon }} Visit the Wiki</a>