---
layout: post
title: Don't disable warnings, please
excerpt: Some may be annoying, but they are here for your own good.
date: 2018-06-03 15:30:00 +0200
redirect_from: "/2018/06/03/do-not-disable-warnings.html"
tags: [Articles]
---
[Notepad++](https://notepad-plus-plus.org/download/v7.5.6.html) is compiled with Visual Studio 2015 with a `/Wv:18` compiler option. A quick memo on that from MSDN:
> The compiler can suppress warnings that were introduced after a version you specify by using the /Wv compiler option.
> This is useful for managing your build process when you introduce a new toolset version, and want to temporarily suppress new warnings.

Looking up at the table of values, `18` means all warnings introduced after Visual Studio 2013 are disabled.

Why were those removed? Compiling with the option disabled makes it painfully obvious:
```
1>..\src\Misc\PluginsManager\PluginsManager.cpp(539): warning C4477: 'swprintf' : format string '%d' requires an argument of type 'int', but variadic argument 1 has type 'std::size_t'
1>..\src\Misc\PluginsManager\PluginsManager.cpp(539): note: to simplify migration, consider the temporary use of /Wv:18 flag with the version of the compiler with which you used to build without warnings
1>..\src\Misc\PluginsManager\PluginsManager.cpp(539): note: consider using '%zd' in the format string
```

Since Notepad++ is also compiled with `/WX` (treat warnings as errors), those relatively trivial and harmless warnings make it impossible to compile.
My assumption is, after upgrading the compiler to VS2015, somebody noticed those new warnings and adhered to the hint it gave.

Where's the problem?

While suppressing format related warnings is OK, several other warnings were also hidden in the process. Sadly, some of them are fairly critical:
```
1>..\src\ScitillaComponent\ScintillaEditView.cpp(857): warning C4311: 'reinterpret_cast': pointer truncation from 'const TCHAR *' to 'int'
1>..\src\ScitillaComponent\ScintillaEditView.cpp(857): warning C4302: 'reinterpret_cast': truncation from 'const TCHAR *' to 'int'
1>..\src\ScitillaComponent\ScintillaEditView.cpp(860): warning C4311: 'reinterpret_cast': pointer truncation from 'BufferID' to 'int'
1>..\src\ScitillaComponent\ScintillaEditView.cpp(860): warning C4302: 'reinterpret_cast': truncation from 'BufferID' to 'int'
```

Let's take a look at the code:
```cpp
itoa(reinterpret_cast<int>(userLangContainer->getName()), intBuffer, 10); // use numeric value of TCHAR pointer
execute(SCI_SETPROPERTY, reinterpret_cast<WPARAM>("userDefine.udlName"), reinterpret_cast<LPARAM>(intBuffer));

itoa(reinterpret_cast<int>(_currentBufferID), intBuffer, 10); // use numeric value of BufferID pointer
execute(SCI_SETPROPERTY, reinterpret_cast<WPARAM>("userDefine.currentBufferID"), reinterpret_cast<LPARAM>(intBuffer));
```

Ouch! Both of those are pointers, and they are being truncated to integer values -- you can get away with this when targetting a 32-bit architecture (although it's still nasty),
but on a 64-bit architecture it can cause all sorts of issues, ranging from unnoticeable bugs to critical crashes.

How can we fix this? `itoa` does not have overloads for 64-bit values, but we can go ahead and use `sprintf` to fix this:
```cpp
sprintf(intBuffer, [format], userLangContainer->getName()); // use numeric value of TCHAR pointer
```

What format to use, though? The code expects a decimal value of said pointer, so you may think `%d` is the way to go -- but, it's suited for integer values,
and pointer can be wider than that! There is also `%lld` which would work fine for 64-bit arch, but not for 32-bit arch...

There is `%p`, but it formats the pointer in an architecture-specific format, and that usually means printing it in hexadecimal. That's also a no.

How to solve this without branching the code for different architectures? [I raise you `inttypes.h`](http://www.cplusplus.com/reference/cinttypes/),
introducing format specifier for width-based types.

Since we want to treat the pointer as an unsigned number, we can cast it to `uintptr_t`. A macro for printing values of this type is `PRIuPTR` -- and it
expands to a correct string depending on target architecture. Perfect!

All things considered, correct code should look like this:
```cpp
// Typecasting pointers to intptr types is perfectly fine!
sprintf(intBuffer, "%" PRIuPTR, reinterpret_cast<uintptr_t>(userLangContainer->getName()));
execute(SCI_SETPROPERTY, reinterpret_cast<WPARAM>("userDefine.udlName"), reinterpret_cast<LPARAM>(intBuffer));

sprintf(intBuffer, "%" PRIuPTR, reinterpret_cast<uintptr_t>(_currentBufferID));
execute(SCI_SETPROPERTY, reinterpret_cast<WPARAM>("userDefine.currentBufferID"), reinterpret_cast<LPARAM>(intBuffer));
```

Moral of the story is, **don't suppress warnings**. It's better to fix them, or at least leave them be, rather than suppress -- otherwise you're
potentially missing out on crucial information.

***

Everything presented here has been submitted back to Notepad++ repository as a [pull request](https://github.com/notepad-plus-plus/notepad-plus-plus/pull/4544).
Format strings are still left untouched, so warning remains -- if/when this gets merged, another pull request can be created which sorts out format strings,
and therefore warnings will then be re-enabled to help catch such issues in the future.
