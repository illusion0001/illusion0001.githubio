---
layout: post
title: Correcting "run over pedestrians" dialogues in SilentPatchSA & 10,000 YouTube subscribers
excerpt: Running over people shall not be ignored ever again.
date: 2018-09-23 15:25:00 +0200
tags: [Progress Reports]
---
First of all, I am proud to announce that **we have reached 10,000 subscribers on YouTube!**
While YT wasn't and isn't my main focus point, it's nice to know that my occasional uploads are interesting for plenty of people.

Even though it's nothing major, feel free to consider this progress update as a very non-unique 10k subscribers non-special! 😁

***

Since I haven't been reporting anything about GTA modding for a long while, a quick progress update on SilentPatch for San Andreas:

San Andreas has a feature which has always been in the game, but remained mostly unused due to the weird way it was implemented.
Passengers can comment on CJ running over people in various ways, just like they comment on car crashes.
However, code did not actually check for peds being run over, instead it checked for vehicle damage which is caused by a pedestrian.
Voice lines would play only if enough damage was done to the car -- unfortunately, most crashes don't deal enough damage so lines never play!

The only ways to get those voice lines to play with a stock game are:
* Use a Quad -- oddly enough, this is the only vehicle which receives enough damage when running over pedestrians. Therefore, this is the only vehicle where this feature works flawlessly.
* Push peds gently against the wall, without knocking them over -- doing so damages the car just as if the player was pushing it against the wall directly, but since vehicle is touching
  the pedestrian, voice lines trigger! From a design standpoint, this makes no sense, as all dialogues imply pedestrians are ran over and killed, whereas in this case we don't even knock them down!

SilentPatchSA Build 30 modifies this feature by making those voice lines play when a ped is being ran over (pushing them against the wall does nothing now).
If done this way, it works with any vehicle, even planes and helicopters!

<div align="center" class="video-container">
<iframe src="https://www.youtube.com/embed/5AryNzWYMYQ" frameborder="0" allowfullscreen></iframe>
</div>

As always, there is **no** ETAs on Build 30, so just sit back and follow the progress :)