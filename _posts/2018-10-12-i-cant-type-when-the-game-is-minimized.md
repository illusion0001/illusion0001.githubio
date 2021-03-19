---
layout: post
title: "I can't type properly when the game is minimized, why?"
excerpt: Sleeping with keyboard input may sound fun, but it makes everyone else upset.
date: 2018-10-12 20:30:00 +0200
tags: [Articles]
---

**Saints Row: The Third** has always had a minor annoyance showing up when you minimized the game. Although on the first glance everything seemed to behave as expected,
if you tried to write anything you would find that as the game is running in the background, typing becomes ridiculously slow!

This does not happen with Saints Row IV, means it is an issue or a side effect of something which got changed between SRTT and SRIV.
Moreover, it happens with both DX9 and DX11 executables, so it's safe to assume it has nothing to do with graphics API used.

Message hooks
=============

It is safe to assume that whatever SR's process is doing is affecting other applications.
A potential candidate would be [message hooks](https://docs.microsoft.com/en-us/windows/desktop/winmsg/using-hooks).
It is important to know that some types of hooks may be installed thread-wide -- however, some hooks get installed system-wide,
so a single process **can affect all other processes running on the same level of permissions**![^1]

That matches our current observations, so let's see if SRTT uses message hooks. Sure enough, a low-level keyboard input hook is installed:

```cpp
if ( KeyboardLLHk == NULL )
{
  v2 = GetModuleHandleA(0);
  if ( v2 != NULL )
    KeyboardLLHk = SetWindowsHookExA(WH_KEYBOARD_LL, fnKeyboardLLHook, v2, 0);
}
```

Some hook types may be installed only in global (system) scope, while some may be both global or per-thread. `WH_KEYBOARD_LL` can **only be installed in global scope**.
Saints Row installs its own hook for all non-elevated processes running on the system!

Investigation
=============

Before we proceed, let's see what SR is doing when it's minimized -- attaching a debugger while the game is minimized and stopping execution a few times
is bound to reveal the most frequently executed code spot. The answer is actually fairly obvious:

```
>	ntdll.dll!_NtDelayExecution@8()
 	KernelBase.dll!SleepEx()
 	KernelBase.dll!_Sleep@4()
 	SaintsRowTheThird_DX11.exe!00dc86eb()
 	[Frames below may be incorrect and/or missing, no symbols loaded for SaintsRowTheThird_DX11.exe]
 	kernel32.dll!@BaseThreadInitThunk@12()
 	ntdll.dll!__RtlUserThreadStart()
 	ntdll.dll!__RtlUserThreadStart@8()
```

As any sensibly coded game should do, SR is sleeping a lot when minimized (because there is no point to the game hogging resources while it's idling in the taskbar)
-- to be precise, it sleeps for as long as 500ms.
That is nice, but how does it relate to other processes acting slow?

Devil is in the details -- if we refer to MSDN docs for a `WH_KEYBOARD_LL` hook type, something very suspicious is mentioned (text emphasis added by me):
> This hook is called in the context of the thread that installed it. **The call is made by sending a message to the thread that installed the hook**.
> Therefore, the thread that installed the hook must have a message loop.

That's the catch! Turns out, the way this specific hook works requires immediate input from the application which installed them -- "sending" is synchronous, "posting" would be asynchronous.
You may think of it like this -- each process in the system, upon receiving a keyboard input message, pokes SR's process by sending it a message:
> *"Hey, here is my keyboard input -- may you process it? Oh yes, sure, I'll wait."*

Suddenly, sleeping changes from a good practice to a horrible, horrible idea.
Because of this hook, **any** process receiving keyboard input waits for SR to wake up from this sleep and process the message before jumping back to its own code and processing the input!
This is exactly the reason SR limits typing speed to approximately 2 characters per second when minimized. Scary.

Fixes
=====

How do we fix it? Of course, we could just remove the offending sleep call, but that would make SR spin needlessly when minimized.
If there only was a way to sleep and wake up when SR needs to process the message...

...or is there?

Thankfully, there is -- [MsgWaitForMultipleObjects](https://docs.microsoft.com/en-us/windows/desktop/api/winuser/nf-winuser-msgwaitformultipleobjects) can do exactly what we need.
We can even specify what specific messages we want to wake up on. Initially, I expected the game needs to wait for a `QS_INPUT` type of messages (that is, any input from mouse, keyboard or raw input),
however this was not the case.

Recall what was said about the way this hook notifies the thread which installed the hook, though -- it does so by **sending** a message,
therefore we need to wake up if any message is sent to the thread. Thankfully we have `QS_SENDMESSAGE` to do just that!

A simple replacement function injected instead of a stock sleep call does the trick wonderfully. This "message sleep" will sleep for `dwMilliseconds` (in our case, 500ms),
unless a message is sent to the process -- if that happens, it will wake up immediately.
```cpp
void MsgSleep( DWORD dwMilliseconds )
{
  MsgWaitForMultipleObjects( 0, nullptr, FALSE, dwMilliseconds, QS_SENDMESSAGE );
}
```

I can now minimize the game and type full speed -- **success**!

Appendix
========

An important thing to note is that in this case I concentrated purely on preventing an issue caused by using a keyboard input message hook.
In reality, you would rather ask yourself a question -- do you really, REALLY, **REALLY** need your application to install global message hooks?
And if it does, do they really have to be in global scope and not just per thread?

You see, Saints Row: The Third did **not** need them.

My proposed fix is not present in the ~~game~~ patch (13 Oct edit) anymore -- because, at a later point, I went back and checked if SR really needs those hooks.
Turns out it does not, and removing them avoids any issues investigated above.

Finding fixes is a neat thing, but sometimes it's even better to take a step back and look at the bigger picture. In this case, it paid off.

EDITS:\\
{{ "2018-10-14" | date: page.date-format }}: Rephrased a sentence to be more accurate.

[^1]: If you were ever to say "but running games as admin is totally safe and does no harm!" -- well, that is one of (several) reasons why it may not be a good idea.