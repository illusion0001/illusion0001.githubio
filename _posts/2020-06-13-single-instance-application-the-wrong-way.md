---
layout: post
title: "Coding a Single Instance Application the wrong way"
image: "assets/img/posts/kao/card.png"
excerpt: "Kao: Mystery of Deadlock."
date: 2020-06-13 12:00:00 +0200
tags: [Articles]
---

Today, I am looking into a niche Polish game -- **Kao: Mystery of Volcano**, a third installment in the Kao series.
Since Kao the Kangaroo: Round 2 is currently [free on Steam](https://store.steampowered.com/app/1048540/Kao_the_Kangaroo_Round_2/)
and a new Kao game [has just been announced](https://twitter.com/Kaothekangaroo/status/1270279470994329600), I think this is a great time
to check this game out for the very first time.

I got my hands on the game (sadly it's not on Steam), installed it, tried to open `launcher.exe` and... nothing.
While the game itself doesn't require the launcher and runs fine, it's impossible to configure graphics settings from within the game.
Therefore, the launcher isn't mandatory, but it's a welcome addition.

Surprisingly, a [PCGamingWiki article](https://www.pcgamingwiki.com/wiki/Kao:_Mystery_of_Volcano) for Kao 3 doesn't mention this issue at all.
What can we do about it then? Debug it, of course.

# Debugging the issue

Before jumping to a debugger, I took advice from [RibShark](https://twitter.com/RibShark) and inspected the process closely in the Task Manager.
Of course, it's there, uses no CPU time and memory stays at a constant low value. This hints that the process is stuck on something.
We may be able to find out what it's stuck on by using `Analyze wait chain`. This little dialog shows if the process is waiting for another process,
and if it does -- what process it's waiting for. Sure enough, the launcher is waiting for... `nvcontainer.exe`.

<p align="center">
<img src="{% link assets/img/posts/kao/wait-chain.jpg %}">
</p>

The issue isn't about this one specific process, however -- I've also seen it waiting for `devenv.exe` (Visual Studio) earlier,
so it appears that for some bizarre reason, the launcher is waiting for random running processes.

Time to peek into the launcher's code. I disassembled the executable and... good grief, its assembly code is convoluted
and annoying to read, typical for executables created from Delphi. A quick peek into the binary with CFF Explorer confirms
that theory, as it claims the launcher was built with Borland Delphi 4.0.

One of the first things the launcher does is perform two checks -- they must both return false, or else the process terminates.
The first check is obvious:

```cpp
bool Foo() {
    return FindWindowA("Launcher: Kangurek Kao - Tajemnica Wulkanu", nullptr) != nullptr;
}
```

Interestingly, this string looks like a window name, but it's been passed as a "class name" parameter -- I don't know if it's intended
or a mistake. Wrong or not, this function isn't the one causing trouble.

The other function is much more interesting. It enumerates **all** top level windows and performs some actions on each of them, like this:

```cpp
bool Bar() {
    gFoundMatchingWindow = false;
    EnumWindows(EnumProc, nullptr);
    return gFoundMatchingWindow;
}

BOOL CALLBACK EnumProc(HWND hwnd, LPARAM lParam) {
    bool continueEnum = !WindowFunc(hwnd);
    if ( !continueEnum ) {
        gFoundMatchingWindow = continueEnum;
    }
    return continueEnum;
}

bool WindowFunc(HWND hwnd) {
    char className[30];
    LRESULT msgResult;

    GetClassNameA(hwnd, className, sizeof(className));
    msgResult = SendMessageA(hwnd, 0x8065, nullptr, nullptr);
    return strcmp(className, "TMainMenu") == 0 && msgResult == 1818;
}
```

What's wrong with this code? Everything! Before I elaborate on why this code is bad, we need to understand **why** this code is here in the first place.

# Why all this hassle?

Given that these two checks determine whether the launcher should run or just terminate in the background,
they are here to allow for only a single instance of the launcher. It's a simple concept -- it only means that under no circumstances
it should be possible for two launchers to run at once. Only the first attempt succeeds, then any next attempts just result in the application
terminating before it shows anything for the user.

The launcher application tries to achieve this in two steps:

1. Looks for the window whose class name is `Launcher: Kangurek Kao - Tajemnica Wulkanu`. I expect this to be a mistake, as this is more likely to be a window name.
2. Enumerates through every window and does the following:
   1. Gets the class name for the window
   2. Sends a "magic knock" message to the window (`0x8065` doesn't correspond to any standard Windows message, so it must be defined in the launcher itself) and stores the response
   3. Checks for the correct class name and the correct response

All this logic falls apart in point 2.2. Sending a message to a window via `SendMessage` **blocks** until the message has been processed by the target window!
It's easy to envision how this can fall apart -- since the launcher happily broadcasts the message to all windows, it only takes a single window which will not respond in time.
The code isn't protected against this case, so any process with a window that doesn't process messages all the time causes the launcher to hang, waiting for the answer infinitely.

There are a few ways in which we can improve this code, and I'll present 3 of them. The first two are **not optimal** and you should not consider doing the same in your code.
For a **proper** solution, stick around until the last suggestion.

## Attempt #1 -- wait with a timeout

Since the root cause of this issue is waiting endlessly, we could just add a timeout and consider the check failed if the response does not arrive in time:

```cpp
bool WindowFunc(HWND hwnd) {
    // Don't use this
    char className[30];
    LRESULT msgResult = 0;

    GetClassNameA(hwnd, className, sizeof(className));
    LRESULT sendResult = SendMessageTimeoutA (hwnd, 0x8065, nullptr, nullptr, SMTO_BLOCK, 500, &msgResult); // 0.5s per window is way too generous already
    if ( sendResult == 0 && GetLastError() == ERROR_TIMEOUT ) return false; // Timed out

    return strcmp(className, "TMainMenu") == 0 && msgResult == 1818;
}
```

Works? Works. Should you use it? No. It's nothing but a workaround for root issue, which I'll address in #2.

## Attempt #2 -- check who you're sending the message to first

The `WindowFunc` function makes a very weird mistake which is easily avoided -- it first obtains the class name and sends the "magic knock",
and **then** checks for the class name and the response. This doesn't make sense neither in the code nor in real life -- if one was to knock a "magic pattern"
on somebody's door, they would check if this is the right house before even attempting to knock, not when waiting for the response.

When laid out like this, the fix is trivial -- split the checks:

```cpp
bool WindowFunc(HWND hwnd) {
    char className[30];

    GetClassNameA(hwnd, className, sizeof(className));
    if ( strcmp(className, "TMainMenu") != 0 ) return false; // Not the window we want to send the magic knock to!

    LRESULT msgResult = SendMessageA(hwnd, 0x8065, nullptr, nullptr);
    return msgResult == 1818;
}
```

This resolves the root cause -- because the code will now only send messages to windows it knows, realistically it'll never end up with a scenario
where the window doesn't respond. Problem solved!

However, is this the best solution? No.

## Attempt #3 -- what the customer really wanted

In software engineering, it is crucial to emphasize understanding what does the customer truly wants, or in this case,
what is the precise intention of this code.

This code does **not** intend to allow for only one window with a specific name or class.
The true intention is to allow for a single instance of the **application**.
Even if the window has not been created yet, another launcher instance should not run.

The proper way to solve this problem is to have the launcher instantiate a [named Kernel object](https://docs.microsoft.com/en-us/windows/win32/termserv/kernel-object-namespaces).
If we pick a unique enough name for the object, its presence unambiguously indicates that another launcher instance is already running!
Object's lifetime is bound to the process lifetime (if it's not released explicitly earlier), so this is a fully bulletproof solution.

Scratch `Foo` and `Bar` functions -- we can achieve a better result with way less code. It's up to the programmer to pick a specific Kernel object
to instantiate, but programmers commonly use a mutex:

```cpp
bool AnotherInstanceRunning() {
    HANDLE mutex = CreateMutex(nullptr, FALSE, TEXT("Launcher: Kangurek Kao - Tajemnica Wulkanu")); // Or any other unique name
    return mutex != nullptr && GetLastError() == ERROR_ALREADY_EXISTS;
    // We deliberately leak the handle as we want the mutex to have the same lifetime as the process!
}
```

I often see this functionality implemented by first trying to open an existing mutex, and creating one if it doesn't exist.
This is redundant -- `CreateMutex` docs state that

> If the mutex is a named mutex and the object existed before this function call,
> the return value is a handle to the existing object, and the GetLastError function returns **ERROR_ALREADY_EXISTS**.

Therefore, we can achieve the same with a single step, which also removes any possibility of a race condition between processes -- in theory,
such a race condition could occur if another launcher instance reached the `OpenMutex` step before the first instance created the mutex.
Win-win!

# SilentPatch?

This raises a question -- will there be a SilentPatch fixing this? Right now, no -- for two reasons:
* The launcher app is so lightweight it doesn't link against any library supported by Ultimate ASI Loader.
* It would be a bit weird to release a patch for the launcher and not the game, right? 😄