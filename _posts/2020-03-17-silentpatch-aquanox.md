---
layout: post
title: "Fixing mouse issues in AquaNox"
thumbnail: "assets/img/games/bg/aquanox.jpg"
feature-img: "assets/img/games/bg/aquanox.jpg"
image: "assets/img/games/bg/aquanox.jpg"
excerpt: "Lot of red herrings on the way, but it all boils down to a single line fix."
date: 2020-03-17 10:00:00 +0100
tags: [Releases, Articles]
---

TL;DR - if you are not interested in an in-depth overview of what was wrong with the game and how it was fixed, just follow the link to grab **SilentPatch for AquaNox**: \\
<a href="{% link _games/aquanox.md %}#silentpatch" class="button" role="button" target="_blank">{{ site.theme_settings.download_icon }} Download SilentPatch for AquaNox</a> \\
Upon downloading, all you need to do is to extract the archive to game’s directory and that’s it!

***

# Introduction

Back at it with a pretty obscure game this time. The game in question is **AquaNox** -- a submarine-based
shooter from 2001. Looking at both [Steam](https://store.steampowered.com/app/39630/AquaNox/) and
[GOG](https://www.gog.com/game/aquanox) pages the game seems to be positively received and despite being
nearly 20 years old, it doesn't seem to have any major incompatibilities with modern systems
-- with one exception.

At the time of writing this post,
[PCGamingWiki page](https://www.pcgamingwiki.com/wiki/AquaNox) (as well as numerous Steam and GOG reviews)
mentions the following under "Issues fixed":

> ### Unresponsive mouse
> <i class="fas fa-wrench"></i> **High polling rate and DPI**
> * Lower mouse polling rate to 200Hz or lower.
> * Lower DPI of a mouse to 200.
>
> **Notes** \\
> <i class="fas fa-info-circle"></i> This is a common problem with many mice.

I received a Steam copy of the game and its sequel in order to look into this (thanks, [Sui](https://twitter.com/Suicide_pl)!)
and sure enough -- with my 500Hz mouse the game is... less than playable. Mouse movement is very stuttery,
sometimes not registering inputs at all or registering only parts of it:
<video controls="controls">
<source src="https://i.imgur.com/yA95mNN.mp4" type="video/mp4">
</video>

Interestingly enough, whatever the issue is, it was fixed in AquaNox 2! Given how similar the games "feel",
comparing code between these two might be reasonably viable (compared to, say,
[The Godfather and The Godfather II]({% post_url 2018-05-18-fixing-the-godfather %})).

PCGW's fixbox is also not completely accurate. Based on my tests I pinpointed that
this issue depends **only** on mouse polling rate, DPI settings do not affect this behaviour specifically.

In my case, I could just set my mouse polling rate to 125Hz or 250Hz and play,
but if you can't change it then playing this game may be a bit problematic.

# Diving into the code

When approaching this issue, I had a few picks in mind. For a 2001 game, only two viable options for
handling mouse input was to do so via Windows messages or DirectInput -- as far as I know,
Raw Input didn't exist back then at all.

## Theory #1 -- Windows messages
Finding WM based input was trivial -- in almost all cases those messages will be handled by the window procedure,
or one of the functions it calls. AquaNox is no exception -- in fact,
in pseudocode generated from a disassembled window procedure they are the **first** messages:

```c
LRESULT __stdcall WndProc(HWND hWnd, UINT Msg, WPARAM wParam, LPARAM lParam)
{
    switch ( Msg )
    {
    case WM_MOUSEMOVE:
        // ...
        break;
    case WM_LBUTTONDOWN:
        // ...
        break;
    case WM_LBUTTONUP:
        // ...
        break;
    // ...
```

At the first glance, code used to handle those messages could be bugged -- all mouse related messages are
serialized to a fixed size circular buffer, like so:

```c
cursor = (g_writeCursor + 1) % 128;
if ( cursor != g_readCursor )
{
    memcpy(&g_mouseInputBuffer[g_writeCursor], &msgData, sizeof(MouseInput));
    g_writeCursor = cursor;
}
```

My initial theory was that this buffer gets filled and extra mouse input gets discarded.
Given the nature of the bug, this seemed plausible. If you remember from earlier, mouse input is glitched
only with high mouse polling rates -- and sure enough, higher polling rates means that more `WM_MOUSEMOVE` messages
get sent to the game, and thus technically this buffer could get filled much more easily!

Needless to say, despite some obvious red flags this code was **not** responsible.
I easily proved it by removing it; if this code was responsible for controlling camera in the game,
removing it would have disabled camera control -- and this did not happen.

In hindsight, Windows messages were a red herring for at least two reasons:
- In AquaNox 2, this code looks identical. Most notably, buffers are of the same size.
- `WM_MOUSEMOVE` reports **absolute**, not **relative** mouse coordinates! In simpler terms,
  it reports "mouse is now at X, Y" instead of "mouse has moved by X, Y". If dropping these messages caused
  issues, they would have quickly caught up, potentially resulting in uneven, but usable camera movement.

## Theory #2 -- DirectInput
With DirectInput, the situation is a bit different -- it's trivial to find whether it's used at all,
as creating it requires a call to `dinput8.dll`. However, since it's a COM interface, tracking the way it's
used is not as trivial as with Windows messages. In the case of this game, `DirectInput8Create` was called
from 3 very similar, yet not identical functions. Hmmm.

With AquaNox, I was in luck because the game left out a lot of logging code, as well as code showing
descriptive error messages. Some of those even left function names in (!!!), and thanks to that I was able
to quickly find code which was obviously responsible for mouse:

```c
if ( dword_679A74 )
    sub_524340("INP_Mouse()", "multiple instances of inp_mouse are not allowed");
this->m_pDInput = nullptr;

// ...

if ( DirectInput8Create(hInstance, 0x800, &IID_IDirectInput8A, &this->m_pDInput, nullptr) )
    sub_524340("This application requires DX8 DirectInput", "");
```

Following the same further, DirectInput device is also created -- and hey, the disassembler spotted `GUID_SysMouse`
by itself! This further proves that this is **the** code I'm looking for.

```c
if ( this->m_pDInput->lpVtbl->CreateDevice(this->m_pDInput, &CLSID_GUID_SysMouse, &this->m_pDInputDevice, 0)
    || !this->m_pDInputDevice )
    return 0;
```

From this point, it was only a matter of documenting the class (namely, `INP_Mouse`) enough to be able to
comfortably follow its uses. Eventually, I figured out that mouse input was read by two functions.
I won't be posting them all, only the parts which are interesting:

#1:
```c
HRESULT hr = this->m_pDInputDevice->lpVtbl->GetDeviceState(this->m_pDInputDevice, sizeof(state), &state);
if ( hr == DIERR_INPUTLOST || hr == DIERR_NOTACQUIRED )
{
    this->m_pDInputDevice->lpVtbl->Acquire(this->m_pDInputDevice);
    return 0;
}
if ( hr )
    return 0;
```

#2:
```c
if ( this->m_bufferedEntriesRead > 0 )
    return sub_51CCE0(this, a2);

DWORD cbEntries = 32;
HRESULT hr = this->m_pDInputDevice->lpVtbl->GetDeviceData(this->m_pDInputDevice, sizeof(this->m_bufferedDeviceData[0]),
                                    this->m_bufferedDeviceData, &cbEntries, 0);
if ( !hr )
{
    this->m_bufferedEntriesRead = v5;
    this->field_290 = 0;
    return sub_51CCE0(this, a2);
}
if ( hr == DIERR_INPUTLOST || hr == DIERR_NOTACQUIRED )
    this->m_pDInputDevice->lpVtbl->Acquire(this->m_pDInputDevice);

return 0;
```

Let's see what MSDN has to say about these functions.

> ### IDirectInputDevice8::GetDeviceState Method
> Retrieves immediate data from the device.
>
> **Return Value** \\
> If the method succeeds, the return value is DI_OK.
> If the method fails, the return value can be one of the following error values:
> DIERR_INPUTLOST, DIERR_INVALIDPARAM, DIERR_NOTACQUIRED, DIERR_NOTINITIALIZED, E_PENDING.

> ### IDirectInputDevice8::GetDeviceData Method
> Retrieves buffered data from the device.
>
> **Return Value** \\
> If the method succeeds, the return value is DI_OK or DI_BUFFEROVERFLOW.
> If the method fails, the return value can be one of the following error values:
> DIERR_INPUTLOST, DIERR_INVALIDPARAM, DIERR_NOTACQUIRED, DIERR_NOTBUFFERED, DIERR_NOTINITIALIZED.

## Piecing it all together
To tell you the truth, a combination of those code snippets and excerpts from the documentation is enough
to figure out what's wrong with these functions -- but what about AquaNox 2?

While their use of `GetDeviceState` is identical, the function using `GetDeviceData` shows some interesting differences...

```c
DWORD cbEntries = 128;
HRESULT hr = this->m_pDInputDevice->lpVtbl->GetDeviceData(this->m_pDInputDevice, sizeof(bufferedData[0]),
                                    bufferedData, &cbEntries, 0);
if ( hr >= 0 )
{
    // Process input...
```

AquaNox 2:
* Buffers 128 entries instead of 32,
* Treats the return value of `GetDeviceData` differently.

It sure looks like this could be **the** fix, but that's both good and bad. At the first glance, it would seem
that increasing the buffer size to 128 entries should be the correct way to go, but it has at least two major issues:
* We'd just be delaying the point over which mouse input breaks. What if we end up getting 2000Hz/4000Hz mice one day?
  If that happens, the bug is going to be back again.
* Unlike with source code access, where increasing the size of this buffer is as easy as modifying a few constants
  and a class definition, growing class fields in assembly is incredibly tiresome. Offsets for **everything** located
  after said buffer would have to be patched, so in practice it's nearly impossible to be completely sure that every single
  use of every class field has been patched.

Thankfully, there is also that other code difference. Recall what MSDN says about the return value of `GetDeviceData`:

> If the method succeeds, the return value is **DI_OK or DI_BUFFEROVERFLOW**.

This seems promising -- with higher polling rates, it would make sense that we receive more data,
and thus the chance of a buffer overflow gets increasingly likely. Furthermore, AquaNox 1 treats only 0 as a success,
(so `DI_BUFFEROVERFLOW` is treated as failure) while AquaNox 2 considers every non-negative return value
as successful (so `DI_BUFFEROVERFLOW` is treated as success). Basing on that, I can say for sure that
the original code looked like this for AquaNox:
```c
if ( hr == DI_OK )
{
    // Success
}
else
{
    // Failure
}
```

...while AquaNox 2 changed it to:
```c
if ( SUCCEEDED(hr) )
{
    // Success
}
else
{
    // Failure
}
```

Could it be as simple as that? Unlike growing buffers, applying this change in assembly is laughably trivial
-- original code testing this return value looks like this:
```asm
call    dword ptr [ecx+28h] ; IDirectInputDevice8::GetDeviceData
test    eax, eax
jnz     Failure
; Success
```

in order to change the comparison from `==` to `>=`, all we have to do is change `jnz` (jump if not zero) to `jl`
(jump if less [than zero]).

Single byte patched, time to check it in game.
<video controls="controls">
<source src="https://i.imgur.com/UueyX3I.mp4" type="video/mp4">
</video>
Sure enough, it works! **Success!**

At this point, formalizing it into a SilentPatch was easy. While this is not as important,
I later found out that the keyboard input code had the same issue, and so I also pre-emptively fixed this.
Therefore, a very annoying glitch got fixed with [two lines of code](https://github.com/CookiePLMonster/SilentPatchAqua/blob/master/source/SilentPatchAqua.cpp#L21).

# Final words

Issue fixed, lesson learned. What can we as developers learn from it? First and foremost, the most obvious answer
is of course "pay close attention to the documentation", but mistakes happen so even that won't guarantee success 😁

On a lower level, this boils down to the way we work with `HRESULT` values from **any** COM classes:
* Avoid deciding on success/failure by comparing return values with specific constants. Try to always use `SUCCEEDED(x)`
  and `FAILED(x)` macros to make a general decision like this. I absolutely agree they look ugly, but they are objectively
  the best way to deal with `HRESULT` variables.
* Compare return values with specific constants only when deciding on how to react to a **specific** error and/or success.
  Both AquaNox and AquaNox 2 did it right by comparing result against `DIERR_INPUTLOST` and `DIERR_NOTACQUIRED`
  in order to re-acquire the device. The important part here is that they both do so **after** determining that
  the returned value should be treated as a failure.

***

For those interested,
full source code of the patch has been published on GitHub, so it can be freely used as a point of reference: \\
<a href="https://github.com/CookiePLMonster/SilentPatchAqua" class="button github" role="button" target="_blank">{{ site.theme_settings.github_icon }} See source on GitHub</a>