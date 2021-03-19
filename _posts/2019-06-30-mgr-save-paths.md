---
layout: post
title: "How NOT to operate on paths - Metal Gear Rising"
image: "assets/img/posts/MGR-social.jpg"
excerpt: "Cue ultimate guesswork on where Documents are located (and more)."
date: 2019-06-30 21:30:00 +0200
tags: [Articles]
---

Today's test subject is [Metal Gear Rising: Revengeance](https://en.wikipedia.org/wiki/Metal_Gear_Rising:_Revengeance), released in January 2014.
A spinoff of Metal Gear Solid series, developed and ported in-house by [Platinum Games](https://www.platinumgames.com/), also known for e.g. NieR: Automata.

# Paths are hard?

I've been told that Metal Gear Rising exhibits very weird behaviour with the location of its saves/configurations directory.
Indeed, [a quick look at game's PCGamingWiki article](https://pcgamingwiki.com/w/index.php?title=Metal_Gear_Rising:_Revengeance&oldid=764289) reveals that the game doesn't really handle user paths properly
(this link directs to the version of the article as of the moment of publishing this post, not the most up to date one -- that's because information is to be corrected with this post as a reference).
PCGamingWiki outlines issues such as:

> <i class="fas fa-thumbs-down"></i> The game uses a fixed path for its data instead of using the standard Windows variables, if the `C:` drive is missing or the aren't `Documents` folder under the user folder,
> the game will not be able to save and will crash. Possible workarounds include manually creating empty `Documents` folder or using symbolic linking.

> <i class="fas fa-info-circle"></i> Steam Cloud will not sync any files if the system doesn't have a `C:` drive.

> <i class="fas fa-info-circle"></i> Even when the correct drive is present, the game might still crash from **ntdll.dll** as a faulty module. \\
> <i class="fas fa-thumbs-down"></i> The game uses a fixed path for its data instead of using the standard Windows variables, if the `C:` drive is missing the game will not be able to save and will crash.

That all sounds... scary, so let's actually find out how the game behaves! Technically, I should be partially affected by this -- while my `Users` directory is on C drive, my `Documents` have been relocated to D drive.
I run the game, and... sure enough, MGR creates a new `Documents` directory inside `Users\[username]`, just for itself:

<p align="center">
<img src="{% link assets/img/posts/mgr-documents.png %}"><br>
<em>Yes, I <b>do</b> have more documents on this account. Just not here.</em>
</p>

Yes, this is still a C drive, but it's important to note that nowadays Windows always assumes the installation partition is C, **and you cannot change it**.
So while it was possible not to have a C drive on Windows XP (which MGR still supported), it's not possible on anything newer -- and you really shouldn't care about XP in mid-2019 ¯\\\_(ツ)\_/¯

Despite this, we can still look into the code and try to figure out what is really going on. So, **let's do this!**

# Cracking the code open

It didn't take long to find code responsible for saving and loading. While every case is slightly different, they more or less all look like this code opening a `GraphicOption` file,
which stores all settings:
```cpp
getenv_s(&ReturnSize, DstBuf, 256, "USERPROFILE");
sprintf_s(FileName, 256, "%s\\Documents\\MGR\\SaveData\\GraphicOption", DstBuf);
fileHandle = CreateFileA(FileName, GENERIC_READ, FILE_SHARE_READ, NULL, OPEN_EXISTING, FILE_ATTRIBUTE_NORMAL, NULL);
```

It's... bad, but not as bad as people said it would be. The procedure here goes as follows:
1. Expand `%USERPROFILE%` variable (typically `C:\Users\[username]`).
2. Append `Documents\MGR\SaveData` path and `GraphicOption` file name.
3. Open that file.

Just as expected, game **will** handle lack of C drive properly (provided `%USERPROFILE%` points to valid path), but it totally ignores directory relocations.
What should have been done here instead:
1. Obtain path to `Documents` via `SHGetFolderPath` or `SHGetKnownFolderPath`.
2. Append `MGR\SaveData` path and `GraphicOption` file name.
3. Open that file.

But wait a second, so if this is the case, then why do guides mention that C drive must be present? [A news post from MGR's Steam page provides the answer for that](https://steamcommunity.com/games/235460/announcements/detail/1387412155878494752):
> - Fixed crash that would occur when no "C Drive" was found.

Therefore, what I **think** the game used to do was build the Documents path like this:
```
C:\Users\%USERNAME%\Documents\MGR
```
and it has been patched to be this instead:
```
%USERPROFILE%\Documents\MGR
```

Both are wrong, but well... they tried. 😉

On top of that, notice how they are using ANSI variations of filesystem functions -- if your user name contains characters from more than one code page (eg. Cyrillic and German characters together),
prepare to have the game not save your progress at all (or read further for an announcement).

# Steam Cloud

The mystery of directories being in wrong places is solved, but what about Steam Cloud? PCGamingWiki again mentioned that C drive is required, and so does the official patch changelog:
> [...] saved data will not synchronize with Steam Cloud unless there is a "C Drive" on the system.

Is this really true? Let's take a look at [SteamDB](https://steamdb.info/app/235460/ufs/):
```
0/path: MGR/SaveData
0/pattern: MGR.sav
0/platforms/1: Windows
0/recursive: 1
0/root: WinMyDocuments
```

Cloud has been configured to read from a **true** Documents directory, and therefore is configured **correctly** (doesn't need a `recursive` flag, but that doesn't affect anything).
Therefore, the patch note is **wrong**, and the correct way to explain behaviour is _"Saved data will not synchronize with Steam Cloud if Documents have been relocated to a custom location"_.
Steam Cloud obtains a correct Documents path, while the game does not -- so if they don't match, syncing will not take place.

***

# A cure for issues -- SilentPatch for Metal Gear Rising

Since those issues are documented and understood, why not do something about it? SilentPatch for Metal Gear Rising aims to do just that.
With most of the saving/loading code rewritten to correct all listed issues (and more), as well as operate on UTF-8 strings (not on wide chars due to better space efficiency of UTF-8),
saving and loading already works flawlessly, even if Documents directory has been relocated to something as crazy as `X:\\ŻąłóРстуぬねのは-documents`.
This also means Steam Cloud will sync saves correctly in all cases!

Check SilentPatch for Metal Gear Rising on GitHub: \\
<a href="https://github.com/CookiePLMonster/SilentPatchMGR" class="button github" role="button" target="_blank">{{ site.theme_settings.github_icon }} Check on GitHub</a>

This is not the only fix I intend to include, so it may take some more time before the release is ready -- so if you're interested, check back in a few days for an update!