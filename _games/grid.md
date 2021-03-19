---
layout: page
title: "GRID"
excerpt: My modifications for GRID games.
image: "assets/img/games/grid.jpg"
feature-img: "assets/img/games/bg/grid.jpg"
game-series: "grid"
order: 18
---

{% assign items = site.games | where:"parent-series", page.game-series %}
{% include mods-grid.html items=items cell-size="33%" %}