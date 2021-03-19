---
layout: post
title: "Automation & BeamNG.drive joint update is the best thing since sliced bread"
feature-img: "assets/img/posts/automation/automation-img.jpg"
thumbnail: "assets/img/posts/automation/automation-img.jpg"
image: "assets/img/posts/automation/automation-img.jpg"
date: 2018-07-14 18:00:00 +0200
twitter: {card: "summary_large_image"}
tags: [Other]
---
What a day! Yesterday, [Automation](https://store.steampowered.com/app/293760/) and [BeamNG.drive](https://store.steampowered.com/app/284160/) released their joint updates,
a result of collaboration allowing players to drive and crash cars they create.

How cool is that? I don't recall any other cross-game collaborations like this, so maybe these two games are going to show the others that integrations like this
are possible and worth going for -- after all, it's a very common trend in web development, so why couldn't games also integrate with each other?

And it works great! I found it fun both to design the cars in Automation...

<p align="center">
<img src="{{ site.baseurl }}/assets/img/posts/automation/party1.jpg">
<em>My very own '93 Silentworks Party Coupe.</em>
</p>

...and to test them on my own in BeamNG!

<p align="center">
<img src="{{ site.baseurl }}/assets/img/posts/automation/party2.jpg">
</p>

# Personal wishlist

Of course, what we are seeing is only the first release of the exporter and issues occur.
Also, there are some things I would love to see implemented:

- Breakable lights -- right now no material can break, which can look odd. I understand the choice in the case of windows (since cars in Automation have no interiors),
  but breakable lights don't sound like an impossible feat, especially since they seem to have insides modelled.
- Combining exports -- right now, one exported vehicle is equal to one vehicle in BeamNG, so multiple variations of the same car cannot be combined into one.
  It would be cool to see those be treated by BeamNG as a single vehicle with multiple body types:
  ![]({{ site.baseurl }}/assets/img/posts/automation/mk1.jpg)
  Also multiple engine types per car anyone?
- Correct light materials -- an exporter bug you can actually observe on my screenshots posted above. Notice how headlights look different in Automation and BeamNG?
  This seems to be a known bug, so I'm hoping for a timely fix for those -- but for now it renders some interesting setups impossible.
  For example, compare tail lights from the same sports car I created -- left is Automation, right is BeamNG:
  ![]({{ site.baseurl }}/assets/img/posts/automation/tail.jpg)
  Bummer =(

***

*This is not a tech post at all, but hey -- [I did warn you]({{ site.baseurl }}{% post_url 2018-05-12-launching-blog %}).*