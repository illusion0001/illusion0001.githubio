---
layout: page
title: "DiRT"
excerpt: My modifications for DiRT games.
image: "assets/img/games/dirt.jpg"
game-series: "dirt"
order: 17
---

{% assign items = site.games | where:"parent-series", page.game-series %}
{% include mods-grid.html items=items cell-size="33%" %}