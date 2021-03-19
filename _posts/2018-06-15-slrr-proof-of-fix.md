---
layout: post
title: "Identifying issues in Street Legal Racing: Redline v2.3.1"
excerpt: Getting through disassembly of an old game in hopes to improve it.
feature-img: "assets/img/posts/slrr-img.jpg"
thumbnail: "assets/img/posts/slrr-img.jpg"
image: "assets/img/posts/slrr-img.jpg"
date: 2018-06-15 23:25:00 +0200
redirect_from: "/2018/06/15/slrr-proof-of-fix.html"
tags: [Articles]
---

This is a follow-up to a tweet I posted a few days ago:

<div align="center">
<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">Developer not finding several memory corruption issues which could lead to random crashes?<br>Hold my beer.<a href="https://twitter.com/hashtag/SilentPatchIt?src=hash&amp;ref_src=twsrc%5Etfw">#SilentPatchIt</a></p>&mdash; Silent (@__silent_) <a href="https://twitter.com/__silent_/status/1005960668950990848?ref_src=twsrc%5Etfw">10 June 2018</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>
</div>

# The game
A quick introduction to the subject -- I am analyzing **Street Legal Racing: Redline v2.3.1** -- officially endorsed expansion/enhancement of original Street Legal Racing: Redline,
which originated as an unofficial modification, before Raxat, author of v2.3.0 and v2.3.1 received source code of SLRR from [Invictus Games](https://en.wikipedia.org/wiki/Invictus_Games_(company)),
original developers.

Original game was released back in 2003 so it's not possible to obtain it digitally anywhere. However, v2.3.1 can be obtained from Steam:

<div align="center">
<iframe src="https://store.steampowered.com/widget/497180/" frameborder="0" width="900" height="200"></iframe>
</div>

Concept of this game is pretty great, and I have pretty fond memories of playing v2.2.1 Miran Wichur Mod (yet another unofficial patch/expansion to the game, made by fellow Poles) around 2006.
However, this game has one major problem -- it's **absolutely riddled with bugs**!
This was made obvious by the amount of official patches the game received, as well as numerous modifications overhauling the game and fixing some of its long standing issues
(like forementioned v2.2.1 MWM and v2.3.0).

# Subject
There is no better way to reunite with a game after years than to buy its digital re-release, and so I did!
However, it appears like game got just a bit less buggy than it used to be years ago...

Can we fix this?
Of course! A new SilentPatch is out of question however -- the game is still being maintained, so SilentPatching it could potentially turn into a non-hostile game of cat-and-mouse.
Thankfully Raxat, main developer, is active online, so a much better way to help is to relay found issues directly! Therefore, since this post is meant to work as a documentation
of found issues, it may get **very** technical.

**DISCLAIMER: Although I am pointing out bugs in the game, its developer should not be blamed for them. Bear in mind that this is a code inherited from another developers,**
**so most/all found issues trace back to the original game and not Raxat's take on it. Please think twice before calling somebody out on "how buggy the game is".**

For the record, none of the found issues are normally fully reproducible, so it's not like the game crashes for everybody all the time.
There is a noticeable amount of random crashes, which seem hard to reproduce -- documented issues may be one of several reasons behind those.

# Assessment results
Several hours of debugging, poking the game with different tools and disassembly analysis resulted in several findings:

## Off-by-one memory copies
With memory debugging tools enabled, I couldn't even get to the game. Crashing function was very suspicious, but I quickly identified it as `memmove`.
Turns out, there are **a lot** of places in the code which do things like:
```cpp
dest = (BYTE *)allocateMem(size + 1);
memmove(dest, src, size + 1);
```

`src` is of size `size`, but often copies get performed like this. I guess somebody thought allocating an extra byte at the end makes the code safer, because then
the string can be null terminated. However, all they managed to achieve is make `memmove` crash on copying the last byte, since `src[size]` does not have to be valid memory!

My fix for this is not to copy the last byte, but instead zero it (like it might have originally been intended):
```cpp
void* result = memmove( dest, src, size );
uint8_t* lastByte = static_cast<uint8_t*>(dest);
lastByte[size] = 0;
```

This is of course a sub-optimal fix, but with no source access it's as good as it gets.
Ideally, each of those cases should be curated to check if allocating one extra byte is useful or not.

## Off-by-one string copies
Very similar to the last one, also may be occuring in a lot of places. Code like this was crashing when trying to copy a string which wasn't null terminated
(and I don't know why it wasn't):
```cpp
dest = (char *)allocateMem(size + 1);
this->text = dest;
strncpy_terminate(dest, src, size + 1);
this->text[size] = '\0';
```

`strncpy_terminate(dest, src, size)` appears to be a custom string copy function, which copies up to `size` characters from `src` to `dest`, and then overwrites the last character in `dest` with `\0`.
Once again, logic in here is off by one -- before being overwritten by zero, it is possible for the last character from `src` to be copied, and `size + 1` logic present in the code once again
attempts to copy character from memory which may be invalid! It doesn't matter that it is being overwritten shortly after -- any attempt to read from invalid memory may result in a crash.

Once again, my fix is to `strncpy` one character less and zero the last character on my own.

## Broken string tokenization
CFG reader used by the game seems to tokenize lines by newline characters. Annoyingly, tokenization does not seem to stop at a null terminator, so if the file does not end with a newline,
unexpected things may occur. Unfortunately, this seems to be the case with at least 3 CFG files in the game -- and it's been a cause of a rather comical crash during testing,
where I (finally) got into the game without a crash after fixing two previous issues, and then crashed... as soon as I pressed throttle!

Files missing a newline are:
- asphalt.cfg
- graph.cfg
- graph_fade.cfg

Addressing that mistake in files of course helps, but ideally CFG parser should be updated to properly handle these cases (this game hates null terminated strings! Why?).

## "Too clever" tag detection
Again (most likely) related to CFG file parsing. Somebody came up with a "nice" optimization of parsing four-character long tags by interpreting them as a 4-byte value:

```cpp
tag = *(uint32_t*)line;
if ( tag == 'ALCI' )
{
    // ...code...
}
```

While this seems harmless, it was a yet another cause of my crashes! This is a harmless optimization, as long as you ensure that **at least 4 bytes of `line` are readable**.
In my case, CFG reader tried to obtain `tag` close to the end of `line`, so it attempted to read invalid memory.

Proper fix for that is to scrap this "optimization" and replace it with a `strncmp` function, which will gracefully return if `line` has less than 4 characters left.

***

All documented fixes have been assembled into a proof-of-concept plugin.
It has been tailored **only** for Build 936, I do not know what are the effects of running it with a newer/older game version.

<a href="https://github.com/CookiePLMonster/SLRR-Proofix" class="button github" role="button" target="_blank">{{ site.theme_settings.github_icon }} See source code on GitHub</a>

**{{ "2018-07-14" | date: page.date-format | upcase }} UPDATE:**\\
A few days ago, SLRR was updated without changing the build number. Therefore, this fix is now **incompatible** with an up to date version of the game.

# Unresolved issues
Of course, there are a lot more issues with the game, mainly concerning random gameplay issues (collision detection failures) and D3D misuses (**so** many D3D calls each frame!).
They are not as trivial to fix though, so... maybe spare them for Part 2 :)
