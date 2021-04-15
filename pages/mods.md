---
layout: page
title: "Mods & Patches"
subtitle: My modifications and patches for various games.
excerpt: My modifications and patches for various games.
permalink: /mods/
hide: true
---

Looking for an easy to browse list? Try the [**Mod Index**]({% link pages/mod-index.html %})!

***

{% assign games = site.games | where:"parent-series", "" | where_exp:"item","item.order < 100" %}
{% include mods-grid.html items=games %}

***

{% assign consoles = site.games | where:"parent-series", "" | where_exp:"item","item.order >= 100" %}
{% include mods-grid.html items=consoles %}