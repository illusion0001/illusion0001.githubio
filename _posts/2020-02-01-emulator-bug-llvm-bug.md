---
layout: post
title: "Emulator bug? No, LLVM bug"
excerpt: "The fact it sounds unlikely does not mean it's not true."
date: 2020-02-01 21:50:00 +0100
tags: [Articles]
---

*If you are not here for the backstory, scroll down to [**A deeper dive into the code**](#a-deeper-dive-into-the-code) section.*

***

Today's test subject started as a crash found in [RPCS3](https://rpcs3.net/) when randomly testing
one of the demos I own - FIFA 17 Demo[^1]. While this game runs really well on my PC, it's building up a really big SPU cache -- 5 or so minutes of playtime
would result in over 20.000 SPU objects being cached!

<p align="center">
<img src="{% link assets/img/posts/llvm-bug/spu-cache.png %}"><br>
<em>This takes a while, but later saves CPU time when actually playing - nice.</em>
</p>

This worked fine on the first try, but trying to compile them again on the next boot
(RPCS3 compiles all cached SPU objects on startup, so subsequent sessions perform better as they don't need to do this work on runtime)
would **consistently** crash the emulator for me. Even worse, I seem to be alone with this issue -- other users testing the same game did not have such issues.

Thankfully, I am able to test RPCS3 on more than one PC, which differ only in CPU:
* My main PC has Intel i7 6700K -- CPU with 4 cores and 8 threads. This is the PC which has a consistent crash.
* Another PC has Intel i7 7800X -- CPU with 6 cores, 12 threads and AVX-512. On this PC **I was unable to reproduce this crash!**

Looking at differences between those two CPU's, I could identify two possible causes -- more CPU cores or presence of AVX-512.
The first claim could be tested easily by modifying CPU affinity, while the second could be verified by configuring RPCS3 to ignore presence of AVX-512.
Sadly, neither got me any closer to the root cause -- 7800X still would not crash, whereas 6700K would refuse to work even if I tried to transfer the cache.

The only variable 6700K **was** affected by was CPU affinity - I could reduce the likelihood of a crash by messing with it, but that seemed counter-intuitive
-- after all, 7800X has more cores (and thus threads), so reducing them should have been unrelated. *And yet...*

All in all, at the first glance it's a classic race condition -- hardware dependent, somewhat random, unpredictable.

[^1]: I really don't like games like this, but my brother loves them.

# A shallow dive into the code

Trying again with a debugger attached, results could be observed quickly -- crashing always with the same call stack:

```
 	rpcs3.exe!llvm::RuntimeDyldELF::resolveRelocation(class llvm::SectionEntry const &,unsigned __int64,unsigned __int64,unsigned int,__int64,unsigned __int64,unsigned int)
 	rpcs3.exe!llvm::RuntimeDyldELF::processRelocationRef(unsigned int,class llvm::object::content_iterator<class llvm::object::RelocationRef>,class llvm::object::ObjectFile const &,class std::map<class llvm::object::SectionRef,unsigned int,struct std::less<class llvm::object::SectionRef>,class std::allocator<struct std::pair<class llvm::object::SectionRef const ,unsigned int> > > &,class std::map<class llvm::RelocationValueRef,unsigned __int64,struct std::less<class llvm::RelocationValueRef>,class std::allocator<struct std::pair<class llvm::RelocationValueRef const ,unsigned __int64> > > &)
 	rpcs3.exe!llvm::RuntimeDyldImpl::loadObjectImpl(class llvm::object::ObjectFile const &)
 	rpcs3.exe!llvm::RuntimeDyldELF::loadObject(class llvm::object::ObjectFile const &)
 	rpcs3.exe!llvm::RuntimeDyld::loadObject(class llvm::object::ObjectFile const &)
 	rpcs3.exe!llvm::MCJIT::generateCodeForModule(class llvm::Module *)
>	rpcs3.exe!jit_compiler::add(std::unique_ptr<llvm::Module,std::default_delete<llvm::Module>> module) Line 1174
 	rpcs3.exe!spu_llvm_recompiler::compile(spu_program && _func) Line 4782
```

It's not even crashing inside RPCS3 code - instead, it's crashing inside [LLVM](https://llvm.org/) code. LLVM is used by RPCS3 to compile translated PPU/SPU code to host machine assembly (in this case, x86-64),
so it's a critical part of the emulator. Yet, it's crashing very deep inside LLVM code -- so it's either a bug in LLVM itself, or RPCS3 feeds it wrong input, causing it to crash.

Sadly, as you may see on the callstack, source/line information has been stripped from LLVM library. To proceed, we need to run in Debug.

# A deeper dive into the code

RPCS3, and in turn LLVM compiled in Debug, so a test can be redone. Sure enough, not only the call stack is more detailed, but this time it also has source information retained:

```
>	rpcs3d.exe!llvm::support::endian::write<unsigned int,1>(void * memory, unsigned int value, llvm::support::endianness endian) Line 101
 	rpcs3d.exe!llvm::support::endian::write<unsigned int,1,1>(void * memory, unsigned int value) Line 111
 	rpcs3d.exe!llvm::support::detail::packed_endian_specific_integral<unsigned int,1,1,1>::ref::operator=(unsigned int NewValue) Line 262
 	rpcs3d.exe!llvm::RuntimeDyldELF::resolveX86_64Relocation(const llvm::SectionEntry & Section, unsigned __int64 Offset, unsigned __int64 Value, unsigned int Type, __int64 Addend, unsigned __int64 SymOffset) Line 311
 	rpcs3d.exe!llvm::RuntimeDyldELF::resolveRelocation(const llvm::SectionEntry & Section, unsigned __int64 Offset, unsigned __int64 Value, unsigned int Type, __int64 Addend, unsigned __int64 SymOffset, unsigned int SectionID) Line 941
 	rpcs3d.exe!llvm::RuntimeDyldELF::processRelocationRef(unsigned int SectionID, llvm::object::content_iterator<llvm::object::RelocationRef> RelI, const llvm::object::ObjectFile & O, std::map<llvm::object::SectionRef,unsigned int,std::less<llvm::object::SectionRef>,std::allocator<std::pair<llvm::object::SectionRef const ,unsigned int>>> & ObjSectionToID, std::map<llvm::RelocationValueRef,unsigned __int64,std::less<llvm::RelocationValueRef>,std::allocator<std::pair<llvm::RelocationValueRef const ,unsigned __int64>>> & Stubs) Line 1710
 	rpcs3d.exe!llvm::RuntimeDyldImpl::loadObjectImpl(const llvm::object::ObjectFile & Obj) Line 377
 	rpcs3d.exe!llvm::RuntimeDyldELF::loadObject(const llvm::object::ObjectFile & O) Line 256
 	rpcs3d.exe!llvm::RuntimeDyld::loadObject(const llvm::object::ObjectFile & Obj) Line 1331
 	rpcs3d.exe!llvm::MCJIT::generateCodeForModule(llvm::Module * M) Line 225
 	rpcs3d.exe!jit_compiler::add(std::unique_ptr<llvm::Module,std::default_delete<llvm::Module>> module) Line 1174
 	rpcs3d.exe!spu_llvm_recompiler::compile(spu_program && _func) Line 4782

```

```Unhandled exception at 0x000000000203A259 in rpcs3d.exe: 0xC0000005: Access violation reading location 0xFFFFFFFFFFFFFFFF.```

"Classic" access violation, attempting to reference invalid memory. Curiously, one of the arguments to the crashing functionm `void* memory` equals `0xddddddddddddde29`, 
which looks like a very suspicious value and not just "random garbage". Move up the call stack to `RuntimeDyldImpl::resolveX86_64Relocation` function,
and one of its arguments, `const SectionEntry& Section`, reveals what's up:

<p align="center">
<img src="{% link assets/img/posts/llvm-bug/nice-object.png %}"><br>
</p>

ＵＳＥ  ＡＦＴＥＲ  ＦＲＥＥ．

When running Debug builds, freed memory is filled with `DDh`. Thanks to this, it's easily identifiable in memory, and is more than likely to yield invalid values if used.
As evidenced, in this case it did help.

However, what is `Section` and why is it freed? `RuntimeDyldImpl::processRelocationRef` provides an answer:

```cpp
SectionEntry &Section = Sections[SectionID];
```

`Sections` is a member of class `RuntimeDyldImpl`, defined as:

```cpp
// A list of all sections emitted by the dynamic linker.  These sections are
// referenced in the code by means of their index in this list - SectionID.
typedef SmallVector<SectionEntry, 64> SectionList;
SectionList Sections;
```

**That's it!** LLVM's type [SmallVector](https://llvm.org/doxygen/classllvm_1_1SmallVector.html) is used, which is similar to `std::vector`,
but unlike the STL container LLVM one allows to allocate space for some entries "up front", so space is dynamically allocated only if more entries need to be stored.
In this case, it seems like most instances of `Sections` stored less than 64 entries, and so this optimization was made to reduce computation cost (because of less frequent reallocations).

Sadly, it also seems to have hidden a **major** bug. Because this vector "usually" does not reallocate, most of the time storing a reference to one of its elements
in a local variable is fine. Problems arise when one of the functions called by that function (and `processRelocationRef` is a very, very long function!)
adds more elements to this vector -- it might cause the vector to grow, invalidating all references! While this is a problem with any vector-like type, `SmallVector` concealed the problem
by preallocating 64 entries, so in most cases it never reallocates!

That's exactly what happens here and can be proven by changing the "preallocation" size of `Sections` to 1 element -- causing reallocation to trigger very quickly
and indeed, crash becoming nearly unavoidable.

To solve this, we either need stability of references or we need to stop assuming elements cannot move.

## Proposed fix #1: Stability of references

Looking at the code, it seems like `Sections` needs to allow for random access (so `std::list` is out of the question). Thankfully, there is a container which allows
for random access **and** guarantees reference stability when growing -- `std::deque`.

This change is way too simple -- change the type, recompile LLVM and RPCS3, run the game again and...

<p align="center">
<img src="{% link assets/img/posts/llvm-bug/fifa.png %}"><br>
<em>Woop woop.</em>
</p>

**It works!** Not just an one off -- it works consistently, both in Release and Debug, with and without page heap (which elevates such memory issues). **Success!**

Is this the best solution, though? Vector itself is not at fault -- the issue lies in an assumption that it will not relocate. Let's try something else.

## Proposed fix #2: Removing temporaries

Another way to possibly fix this would be to remove any code assuming vector will not relocate. In our case, it means we most likely need to say goodbye to a `Section` local variable.
It's an easy refactor in the sense that compiler will complain until it's fully done. As long as `SectionID` never changes, it should be a simple copy & paste change -- and in this case,
it thankfully appears to be an immutable class member.

The [diff file](https://gist.github.com/CookiePLMonster/8b024171f180a8e289f43d814fbfe740) is quite long but it was all replaced en masse.
Compiling that and testing **also results in success**. Turns out, "caching" references in local variables was the root issue all along.
Drop it and compilation never fails, just as with the deque-based fix.

# Final words

The issue is now fixed and both proposed fixes are clean enough. I think at this point it's fair to say that unlikely problems are not always impossible.
When approaching this I had serious doubts that it's a bug in LLVM, because "it is a well tested and widely used library". Turns out that was a wrong approach,
and issues can happen everywhere.

***

*As of time of writing this post, Nekotekina adopted [one of the solutions](https://github.com/RPCS3/llvm/commit/9fb67ecaea741afe9ca59d4a8ba8b6428df76c99) to RPCS3's LLVM fork.* \\
*Issue is yet to be reported to upstream LLVM.*