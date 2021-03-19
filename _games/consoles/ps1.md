---
layout: page
title: "Sony PlayStation"
excerpt: My GameShark cheats for PlayStation games.
image: "assets/img/consoles/ps1.jpg"
feature-img: "assets/img/consoles/bg/ps1.jpg"
game-series: "ps1"
order: 100
---

{% assign items = site.games | where:"parent-series", page.game-series %}
{% include mods-grid.html items=items cell-size="33%" %}