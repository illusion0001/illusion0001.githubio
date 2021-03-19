---
layout: post
title: My thoughts about Wooting SDK
date: 2018-10-25 23:50:00 +0200
tags: [Articles]
---
After having used Wooting SDK in a "real" project, some things which I wish were handled a bit differently.

Do note that those are only my opinions and were transferred to test only for reference -- none of my points are meant to be taken as complaints,
more as suggestions on how **I** would rather see those things work.

Suggestions are ordered by importance (subjective, of course).

# 1. SDK distribution method #
As far as I understand, this specific point is subject to change -- but at the moment,
the way SDK is integrated is fairly confusing and counter intuitive.
There are GitHub repositories with SDK headers (like you'd expect from any SDK),
but the same directory also contains internal headers and source files used to build an app side SDK layer!
However, you can **also** use DLLs and main SDK header file is evidently written in a way to support linking
imports from a DLL -- but there is no `.lib` file to link against!
Because of that, it's not possible to let the loader link against SDK DLLs and function definitions
included in the header can only act as a function definition template -- so the best way for an user to make use of them is to
import functions on your own like this:

```cpp
decltype(::wooting_read_full_buffer)* wooting_read_full_buffer; // Notice the usage of :: - so header's function definition gets picked
HMODULE analogLibrary;

[...]

analogLibrary = LoadLibraryW( L"wooting-analog-sdk" );
if ( analogLibrary != nullptr )
{
   wooting_read_full_buffer = (decltype(wooting_read_full_buffer))GetProcAddress( analogLibrary, "wooting_read_full_buffer" );
}
```

### My suggestion: ###
Either ship `.lib` files so it is possible to implicitly link against SDK DLLs,
or mimick the way Logitech G series SDKs handle it -- by providing a lightweight `.lib` file loading SDK DLL from
Logitech's software directory directly. This way that thin layer of code `.lib` includes could account for presence/absence
of the software suite, making it opaque for an user.

# 2. Odd behaviour of "wooting_kbd_connected" (and maybe "wooting_rgb_kbd_connected") #
That one was horrible! My integration was developed using a [Wooting Visualizer](https://dev.wooting.nl/contest-entries/sdk-visualizer-emulator-by-hollow/),
and after I completed it, tests on a real thing showed analog buttons did not work!
Both `wooting_read_analog` and `wooting_read_full_buffer` would consistently return 0 as input/amount of keys pressed (it wasn't -1, indicating an error!).

Culprit? My code looked like this:
```cpp
wootingConnected = wooting_kbd_connected();
int itemsRead = wooting_read_full_buffer( data, sizeof(data) );
if ( itemsRead != -1 )
{
    // Input handled here...
}
```

For whatever reason, `wooting_kbd_connected` being called either this often or just before read functions caused the issue!
Removed that one call and it's all fine.

### My suggestion: ###
Currently, documentation doesn't stress out you should **not** do it this way -- it is "recommended" to call it once,
but it's not deemed a requirement. Either make it clear that this **must not** be done,
or maybe make `wooting_kbd_connected` do complex operations only once, and return a cached result on later calls.

# 3. "wooting_rgb_array_set_full" with alpha channel #
Something similar to what Logitech SDK has. In addition to current function updating the entire keyboard matrix,
we could have another one which also contains an alpha channel for key.
When updating, check if alpha equals 0 -- and if it does, just ignore that key and move on.
Thus, it would be useful if user wants to overlay a pattern over some other existing layout of colours --
by passing 0 alpha it'd be indicated the update call should not update those specific keys. Easy!

# 4. There are callbacks for when the device disconnects -- what about connecting? #
It's good that it is possible to be notified when the device disconnects, but what about the opposite?
In fact, lack of any "on connected" callback was exactly the reason I queried for connect status this often,
thus causing the issues mentioned in 2.

### My suggestion: ###
`*_set_disconnected_cb` shouldn't exist in the first place IMO -- right now, it is theoretically possible to undo damage by just providing
corresponding `*_set_connected_cb` functions which notify when the device connects.
In retrospect, it would be much easier to have a set of `*_set_state_change_cb` functions,
which get called on both events and reflect that in parameters they pass to the callback, like so:
```cpp
wooting_set_state_change_cb( []( bool connected ) {
	wootingConnected = connected;
} );
```

# 5. "wooting_rgb_array_set_single" by scancode #
We can query for analog value by both (row, column) pair and a scan code -- **so why not allow to set RGB both ways, too?**

# 6. Revise the requirement to call "wooting_rgb_reset" on app shutdown #
I admit I am not sure if this issue has an easy solution. Currently,
it is required to call `wooting_rgb_reset` just before the app terminates, so application defined colours are reset.
This works fine, but **what if the application crashes!?** It cannot be guaranteed this function is going to be called,
and calling it from game's exception handler is potentially very unsafe. Is there a way to remove this requirement?

# 7. Multi-segmented Spacebar LED #
I don't know if this is feasible from engineering standpoint, but it would be nice for RGB LEDs to have a multi-segmented Space bar.
This way the bottommost row would be a fully functional row of "pixels", for any kind of art "drawn" on the keyboard.

***

The list may be expanded with more entries at a later point, so I'll be mentioning any changes over here.

{{ "2018-10-26" | date: page.date-format }} update: Added 7.