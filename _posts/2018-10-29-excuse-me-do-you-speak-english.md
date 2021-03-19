---
layout: post
title: Excuse me PCSX2, do you sprechen Inglés?
excerpt: Mixing languages is irytujące.
date: 2018-10-29 23:40:00 +0200
tags: [Articles]
---
Recently, I upgraded [PCSX2](https://pcsx2.net/) with an intent to check it out for the first time in years.
I got to the Quick Start wizard and selected a preferred language, just to have next page's navigation buttons end up looking like this:

<p align="center">
<img src="{% link assets/img/posts/lang-mix.png %}"><br>
<em>Hold on a sec, that's not English.</em>
</p>

Aside from the text being obviously too long[^1] (who came up with this translation? That's not how Polish localizations handle navigation buttons *at all*),
it's also not displayed in the language it should be! This causes the navigation bar to look kind of like a frankenstein of languages.

Why is only Next button affected?

Well, it appears like only this button has its caption changed on runtime, due to the fact it can have either a Next or Finish caption:
```cpp
const bool hasNext = HasNextPage(m_page);
const wxString label = hasNext ? _("&Next >") : _("&Finish");
if ( label != m_btnNext->GetLabel() )
    m_btnNext->SetLabel(label);
```

What you don't see here is retranslation process being carried on to strings enclosed in `_(x)`.
Retranslation is performed according to currently selected locale -- and it ends up changing between the time of creating wizard's buttons
and Next/Finish button having its caption updated!

Fortunately, it's trivially fixable by performing retranslation when creating buttons -- and caching the result.
This way I can fix it **and** use PCSX2 without wasting too much time and without my OCD weeping due to a single button being glitched...

...and let's hope I don't feel like touching [Xenia](https://xenia.jp/) or [RPCS3](https://rpcs3.net/) soon -- as something similar happened
when I tried out [Dolphin](https://dolphin-emu.org/).

***

*Fix for the issue discussed here has been submitted back to PCSX2's repository via a [pull request](https://github.com/PCSX2/pcsx2/pull/2664).*

{{ "2018-10-31" | date: page.date-format }} update: Fix has been merged into both PCSX2 and wxWidgets. Success!

[^1]: And seemingly me being cursed with a curse of finding issues everywhere.