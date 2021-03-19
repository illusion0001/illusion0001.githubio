---
layout: post
title: GInput in Wooting CODE IT contest
date: 2018-10-24 22:25:00 +0200
feature-img: "assets/img/posts/ginput-wooting.jpg"
thumbnail: "assets/img/posts/ginput-wooting.jpg"
image: "assets/img/posts/ginput-wooting-social.jpg"
excerpt_separator: <!--more-->
twitter: {card: "summary_large_image"}
tags: [Other]
---
There has been no news about GInput for a very long while, but here's something new!
<!--more-->
GInput for San Andreas has been enlisted in a _CODE IT_ contest from Wooting -- and voting is now up!

**TL;DR:** If you want to cast a vote, go here and pick your favourite! Who knows, maybe GInput is going to be your pick 😊\\
<https://dev.wooting.nl/vote-it/>

What is Wooting? And what's the contest about?
==============================================

[Wooting](https://www.wooting.nl/) is a brand of mechanical keyboards, which unlike **any** other keyboard out there (as of now) has **pressure sensitive buttons**.
On top of that, the keyboards are fully lit with RGB LEDs, with each separate key being fully controllable.

Now, since analog keyboard buttons are a new concept, there have to be some ways of getting them working,
since naturally no existing API can grasp the concept of an analog keyboard key.
Apparently, it is possible to map specific keys to DirectInput and/or XInput gamepads and map them in games this way.
Wooting also provides their own SDK for both analog keys and RGB lighting, but naturally that needs to be supported by the application itself.

You see where I'm going? With GInput, we can do exactly that! The mod already hooks deeply into GTA input processing,
so adding one more feature was in fact fairly easy.

Coincidentally, Wooting has also announced a _CODE IT_ contest,
which encourages people to come up with interesting ways of using keyboard's SDK.
GInput seems like a perfect fit, and thankfully there is a Wooting keyboard emulator (created by **hollow** -- huge props for that!),
so I don't even need to get the keyboard in order to submit something! It wouldn't ship in time anyway...

Cool, but what did you do?
==========================

So far, the list of features is not long. It should be enough for starters, though:
* Full support for analog keys
* Backlight reflects gang activity in the area you are in
* Function keys flash when the police is after you
* Keyboard dims when the player is hidden in the shadows
* During dancing minigames, relevant keys reflect your score while all the other keys are dimmed

If the project picks up, more features may be added!

May I see it?
=============

I prepared a few previews -- treat yourself:
<div align="center" class="video-container">
<iframe src="https://www.youtube.com/embed/wBR2WAtnnJc" frameborder="0" allowfullscreen></iframe>
</div>
<div align="center" class="video-container">
<iframe src="https://www.youtube.com/embed/2rDSIAGcIIk" frameborder="0" allowfullscreen></iframe>
</div>

However, those previews are kind of... boring. What if we could come up with something more entertaining?

I raise you a mix of LEDs and the most known cutscene from the game 😁
<div align="center" class="video-container">
<iframe src="https://www.youtube.com/embed/080RA4-Mccc" frameborder="0" allowfullscreen></iframe>
</div>

I'm sold -- let me vote!
========================

If you want to cast a vote, go here and pick your favourite! Who knows, maybe GInput is going to be your pick 😊\\
<https://dev.wooting.nl/vote-it/>

GInput presentation on the voting page also hides **a download link** -- if you want to give it a try on your own,
go ahead and grab it!