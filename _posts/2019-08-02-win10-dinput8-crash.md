---
layout: post
title: "Crashing DInput8 DLL with two lines of code"
excerpt: "Load... unload... crash!"
date: 2019-08-02 22:05:00 +0200
tags: [Articles]
---

*If you are not here for the backstory, scroll down to [**Bug in system DLLs?**](#bug-in-system-dlls) section.*

***

Today's test subject started as a crash found in [PCSX2](https://pcsx2.net/).
I got the sources, compiled it and started poking around it in Application Verifier.
Sure enough, I crashed very quickly -- it crashed when enumerating plugins.

Usually those issues are not article-worthy, as use-after-free or read-past-buffer issues
are often overlooked, with Page Heap allowing to detect those quickly.
However, here the call stack was very unusual:

```
>	6b16c5a3()
 	[Frames below may be incorrect and/or missing]
 	ntdll.dll!RtlRunOnceExecuteOnce()
 	KernelBase.dll!_InitOnceExecuteOnce@16()
 	6b16c574()
 	ntdll.dll!RtlpTpWorkCallback()
 	ntdll.dll!TppWorkerThread()
 	kernel32.dll!@BaseThreadInitThunk@12()
 	ntdll.dll!__RtlUserThreadStart()
 	ntdll.dll!__RtlUserThreadStart@8()
```

Those gaps in a call stack don't point to any valid code, with or without symbols.
Is the call stack corrupted or the code is gone? Maybe it was dynamically generated and got freed...?
To make things even worse, since it crashed on a thread, I can't quickly determine who started it. Dang!

# Bug in PCSX2?

Let's take a step back and go back to PCSX2 code. Since crash happens during enumeration of plugins,
start by stepping through enumeration one by one and stop when it crashes.
The offending plugin turned out to be **LilyPad**, a gamepad plugin.

Since plugins are loaded and quickly unloaded,
an obvious issue would be to have one of them create a thread which does not finish before
the DLL unloads. As a result, thread would attempt to execute code which has already been freed -- and the call stack
would look like the one we observed!

Curiously enough, LilyPad does not spawn any threads on its own at this point.
In fact, it does almost nothing -- just loads DLLs it depends on (or rather Windows does it when loading LilyPad DLL)
and performs some very basic initialization, something which could not have caused this.

If it's not LilyPad, maybe it's one of the dependant DLLs? Those are `setupapi.dll`, `dinput8.dll` and `hid.dll`.
However, all three are system DLLs, and surely they could not cause a crash!

...oh, really? =)

# Bug in system DLLs?

Having figured out a bug in LilyPad is not very likely, I turned my sights to dependent DLLs.
Keeping with the theory of a thread spawned by somebody and not finishing before the DLL is unloaded,
I placed a breakpoint in LilyPad's `DllMain` function and checked other threads.
One of the threads' callstacks looked suspiciously familiar...

```
>	ntdll.dll!_NtWaitForSingleObject@12()
 	[...]
 	dinput8.dll!___delayLoadHelper2@8()
 	dinput8.dll!__tailMerge_ext_ms_win_mininput_inputhost_l1_1_1_dll() <--- This was unresolved previously!
 	ntdll.dll!RtlRunOnceExecuteOnce()
 	KernelBase.dll!_InitOnceExecuteOnce@16()
 	dinput8.dll!<lambda_94ee4f713c9c11749fcbf9a867e011a4>::<lambda_invoker_stdcall>() <--- This was unresolved previously!
 	vfbasics.dll!_AVrfpRtlWorkerCallback@4()
 	ntdll.dll!RtlpTpWorkCallback()
 	ntdll.dll!TppWorkerThread()
 	kernel32.dll!@BaseThreadInitThunk@()
 	ntdll.dll!__RtlUserThreadStart()
 	ntdll.dll!__RtlUserThreadStart@8()
```

This looks very much like the crashing thread! Sure enough,
it would seem like `dinput8.dll` had a thread running at the moment it was unloaded due to LilyPad
unloading. Since `DirectInput8Create` was not called at this point, it would seem like `dinput8.dll`
spawns a thread from its own `DllMain`. Can we prove this?

*Cue the disassembler.* We can load the DLL and its symbols and see what those functions are.
With symbols, locating this code was trivial -- in pseudocode it looks like this:

```cpp
DWORD lambda_94ee4f713c9c11749fcbf9a867e011a4_::_lambda_invoker_stdcall_(LPVOID)
{
  InitOnceExecuteOnce(&g_InitOnce, lambda_823c682a5c51c4709611cacc5f9d961c_::_lambda_invoker_stdcall_, 0, 0);
  return GetLastError();
}
```

This lambda was passed as an argument by such callee...

```cpp
BOOL InitializeInputHost()
{
  return QueueUserWorkItem(lambda_94ee4f713c9c11749fcbf9a867e011a4_::_lambda_invoker_stdcall_, 0, 0);
}
```

...which in turn was called by a function called `DllProcessAttach()`! This proves the theory,
although that is not a thread per se -- instead, it's a work item in a **thread pool**.
`QueueUserWorkItem` submits the lambda to be processed asynchronously and resumes execution
immediately -- so `DllMain` does not stall. Where's the catch?

Take a look at [MSDN page for *QueueUserWorkItem*](https://docs.microsoft.com/en-us/windows/win32/api/threadpoollegacyapiset/nf-threadpoollegacyapiset-queueuserworkitem):

> If a function in a DLL is queued to a worker thread, **be sure that the function has completed execution before the DLL is unloaded**.

That seems to be exactly the case here -- work is queued, but nothing waits for the function to finish before unloading the DLL!
I could prove it trivially by placing a breakpoint at the very end of the lambda (below `GetLastError`) -- program crashed
**without** hitting my breakpoint, which means this function has never finished. Nasty!

# Minimal reproducible example

Since the issue seems so trivial, reproducing it in a standalone program boils down to literally a few lines:

{% gist 18632c532d510ddcf5fbb1c84fad672a %}

Load... unload... and wait for crash to happen! Obviously, it won't crash in majority of cases -- but with Application Verifier it's possible to make it consistent.
As mentioned in the snippet, it needs to be ran with **Basics -> Threadpool** tests enabled (I also like using **Heaps** for those cases),
so DLL's code gets released as soon as possible. The reason it works in most cases is because `FreeLibrary` is lazy with releasing module's code and
it persists in memory for long enough for this task to finish. That's not contractual however, and theoretically it can change at any time.

At this point I think it can be said with pretty good confidence that it's a bug in `dinput8.dll` itself.
Based on my tests, it seems to concern both 32-bit and 64-bit apps on Windows 10 1803, 1809 and 1903.

# I'm a developer -- how do I solve this?

I am not aware of any fully reliable ways of solving this other than not unloading the DLL so quickly.
However, this may be done even without any user input -- like in the case of LilyPad, `dinput8.dll` was statically linked,
and so unloading LilyPad also unloaded the linked DLL.

This specific case however can be solved by **delay-loaded DLLs**. Dependant DLLs marked as such are not loaded as soon
as the module depending on them loads -- instead, they are loaded **before the first time a function exported from them is to be called**.
In LilyPad's case, plugin enumeration didn't call any exports from `dinput8.dll`, and it's only been called later,
when LilyPad is **really** being used.
Making `dinput8.dll` a delay-loaded DLL not only works this bug around by not loading it during the plugin enumeration phase,
but it may also potentially speed up the process -- after all, it means less work will be performed when enumerating plugins!

Do note that of course this is **not** an universal remedy to issues of this sort -- but it's very likely to help in your case too,
should you ever encounter this problem.

# How should Microsoft fix this?

The answer seems obvious, but in fact it may not be as trivial as it looks. In theory, it is enough to self-reference `dinput8.dll`
just before queueing the work item, and then release the reference as the callback returns,
e.g. via [*FreeLibraryAndExitThread*](https://docs.microsoft.com/en-us/windows/win32/api/libloaderapi/nf-libloaderapi-freelibraryandexitthread).
The problematic part is increasing DLL's reference count -- obviously, it needs to be done before submitting the work and not inside it itself,
else it'll be as prone to races as earlier. However, work is submitted under a loader lock, and I don't know if
[*GetModuleHandleEx*](https://docs.microsoft.com/en-us/windows/win32/api/libloaderapi/nf-libloaderapi-getmodulehandleexw) with a
`GET_MODULE_HANDLE_EX_FLAG_FROM_ADDRESS` flag can be called under a loader lock or not. If it can, then increasing it from `InitializeInputHost`
and releasing as the lambda exits will be a correct fix.

***

*This issue has been now [reported via the Feedback Hub](https://aka.ms/AA5rr0w).*
*Additionally, I [submitted a pull request to PCSX2](https://github.com/PCSX2/pcsx2/pull/3053) to work around that bug as instructed in the article.*