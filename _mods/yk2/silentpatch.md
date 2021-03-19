---
title: SilentPatch
title-image: "assets/img/mods/silentpatch-yk2.png"
game-series: "yk2"
excerpt: "Peeing minigame in 30 FPS, but also more FPS related fixes."
date: 17-01-2020
---

Remake of Yakuza 2, ported from PS4 to PC in May 2019 enhanced gameplay experience by lifting a 30 FPS lock, allowing PC
users to better leverage their hardware. This was generally fine, but also introduced some issues to the game. While the ones like
exaggerated physics are not trivially fixable, issues with some minigames have been fixed in an official patch by limiting them to 60 FPS.
That said, this not only is mostly redundant, but also not always correct -- most notably,
Toylets minigame is still essentially unplayable. To make matters worse, the way frame limiting was performed was notoriously
flawed and introduced noticeable frame pacing issues.

## Featured fixes:
### Bug fixes

* Frame pacing has been drastically improved when "Framelimit" option is not set to "Unlimited"
* Toylets minigame has been capped to 30 FPS, essentially making it playable without having to change game settings
* Arcade games are now forcibly capped to 60 FPS, even if "Framelimit" option is set to 30 FPS in options
* The following minigames have been capped by the official patch for no reason and have been uncapped again now:
  - Karaoke
  - Mahjong
  - Oicho Kabu
  - Cee-lo
  - Colliseum (menu only)
  - Batting
  - Darts
  - Blackjack
  - Poker
  - Koi-koi

{% include setup-instructions.html %}

<a href="https://github.com/CookiePLMonster/SilentPatchYK2/releases/latest/download/SilentPatchYK2.zip" class="button" role="button">{{ site.theme_settings.download_icon }} Download</a> \\
<a href="https://github.com/CookiePLMonster/SilentPatchYK2" class="button github" role="button" target="_blank">{{ site.theme_settings.github_icon }} See source on GitHub</a>