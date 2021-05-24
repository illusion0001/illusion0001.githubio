# Final Fantasy 7: Remake

## 60 FPS Patch

[Article](https://illusion0001.github.io/patches/2021/05/20/ff7r-end-60fps/)

In file `...\end\content\paks\pakchunk0-ps4.pak`

<details>
<summary>Code (Click to Expand)</summary>

```ini
; This file must be edited in hex editor,
; normal text editors will add more bytes
; and may cause game crashes.

; For end users:
; There are multiple instances of the following lines,
; be sure to change all occurences.
; When replacing, only search for cvars
; i.e search for: rhi.SyncInterval=2
; Do not search for comments as they don't exist!

; Framerate limit, applies to all console modes.

; Find:
rhi.SyncInterval=2 ; 30hz

; Replace:
rhi.SyncInterval=1 ; 60hz

; end of framerate limit
```

</details>

## Dynamic Resolution Patch

[Article](https://illusion0001.github.io/patches/2021/05/20/ff7r-end-60fps/)

In file `...\end\content\paks\pakchunk0-ps4.pak`

<details>
<summary>Code (Click to Expand)</summary>

```ini
; This file must be edited in hex editor,
; normal text editors will add more bytes
; and may cause game crashes.

; For end users:
; There are multiple instances of the following lines,
; be sure to change all occurences.
; When replacing, only search for cvars
; i.e search for: rhi.SyncInterval=2
; Do not search for comments as they don't exist!
; You may adjust parameters to your liking or use the ones provided below.

; Dynamic Resolution Scale Change

; Base Console

; Res scale for Base Console
; Under [PS4 DeviceProfile] ; base res
; Find:
r.DynamicRes.MinScreenPercentage=83.3333333 ; 83% of target ir
r.DynamicRes.MaxScreenPercentage=100 ; 100% of target ir

; Res scale for Base Console
; Under [PS4 DeviceProfile] ; base res
; Replace:
r.DynamicRes.MinScreenPercentage=50.0000000 ; 50% of target ir (540p for base)
r.DynamicRes.MaxScreenPercentage=67 ; 67% of target ir (roughly ~720p for base, use 66.6666667 directly in ini with UE4 patch method for higher accuracy)

; Pro Console 4K/Supersampling mode
; Supersampling must be enabled in the
; Console system menu for users with 1080p displays.

; Res scale for Pro Console 4K mode
; Under [PS4_Neo_4k DeviceProfile] ; 4k Pro res
; Find:
r.ScreenPercentage=75 ; 1620p
r.DynamicRes.MinScreenPercentage=74.0740741 ; lowest is 1200p
r.DynamicRes.MaxScreenPercentage=100 ; highest is 1620p

; Res scale for Pro Console 4K mode (900p60)
; min 720p max ~900p
; Under [PS4_Neo_4k DeviceProfile] ; 4k Pro res
; Replace:
r.ScreenPercentage=50 ; 1080p
r.DynamicRes.MinScreenPercentage=66.6666667 ; 720p
r.DynamicRes.MaxScreenPercentage=83 ; 900p (use 83.3333333 directly in ini with UE4 patch method for higher accuracy)

; Res scale for Pro Console 4K mode (1080p60)
; min 900p max 1080p
; Under [PS4_Neo_4k DeviceProfile] ; 4k Pro res
; Replace:
r.ScreenPercentage=50 ; 1080p
r.DynamicRes.MinScreenPercentage=83.3333333 ; lowest is 900p, same targets as base, just hits higher res more often
r.DynamicRes.MaxScreenPercentage=100 ; highest is 1080p

; end of DynamicRes

; Below are for reference only/.

; [PS4 DeviceProfile] ; What Base PS4 uses.
r.DynamicRes.MinScreenPercentage=83.3333333 ; lowest is 900p
r.DynamicRes.MaxScreenPercentage=100 ; highest is 1080p

; [PS4_Neo_4k DeviceProfile] ; What PS4 Pro uses.
r.ScreenPercentage=75 ; always 2880x1620 by default
r.DynamicRes.MinScreenPercentage=74.0740741 ; lowest is 1200p
r.DynamicRes.MaxScreenPercentage=100 ; highest is 1620p
```

</details>

## Disable Dynamic Resolution Patch (Optional)

In file `...\end\content\paks\pakchunk0-ps4.pak`

<details>
<summary>Code (Click to Expand)</summary>

```ini
; This file must be edited in hex editor,
; normal text editors will add more bytes
; and may cause game crashes.

; Brief Description: Dynamic Resolution adjusts the primary screen percentage according to the previous frames' GPU workload.

; https://docs.unrealengine.com/en-US/RenderingAndGraphics/DynamicResolution/index.html

; This is optional for users who want static resolution. 
; This ignores the following:
; DynamicRes.MinScreenPercentage
; DynamicRes.MaxScreenPercentage
; May significantly impact performance
; Use with caution.

; Find
r.DynamicRes.OperationMode=2

; Replace
r.DynamicRes.OperationMode=0

; Cvar Description:
; 0 = Disabled
; 1 = Enabled based on the setting used in GameUserSettings.
; 2 = Enabled regardless of the setting used by GameUserSettings. (Default)
```

</details>

## Texture Streaming Improvements (Untested)

In file `...\end\content\paks\pakchunk0-ps4.pak`

<details>
<summary>Code (Click to Expand)</summary>

```ini
; https://github.com/illusion0001/illusion0001.github.io/commit/6d72ffa2a1fe389a3d614eb6756bea80b51439a4#commitcomment-51227749
; https://github.com/dontellmama
; Needs checking

r.Streaming.PoolSize= ?? ; 2600 will crash(CE-34787), 2000-2400 fine
r.Streaming.MaxTempMemoryAllowed=40
MemoryMargin=10
```

</details>
