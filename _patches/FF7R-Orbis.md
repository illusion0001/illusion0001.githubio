# Final Fantasy 7: Remake

## 60 FPS Patch

[Blog Post](https://illusion0001.github.io/patches/2021/05/20/ff7r-end-60fps/)

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

[Blog Post](https://illusion0001.github.io/patches/2021/05/20/ff7r-end-60fps/)

In file `...\end\content\paks\pakchunk0-ps4.pak`

<details>
<summary>Code (Click to Expand)</summary>

```ini
; This file must be edited in hex editor,
; normal text editors will add more bytes
; and may cause game crashes.

; Developer notes:
; Neo uses 2880x1620 for highest bound?
; https://youtu.be/qyGl5C3Uwak?t=516
; I'm not sure if adjusting min and max for neo is affective,
; since its base is already at 75%
; maybe only adjust base instead?

; For end users:
; There are multiple instances of the following lines,
; be sure to change all occurences.
; When replacing, only search for cvars
; i.e search for: rhi.SyncInterval=2
; Do not search for comments as they don't exist!
; Base Console needs both Framerate and Res Scale Change.
; Pro console needs testing. Separate DynamicRes cvars below.

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
r.DynamicRes.MaxScreenPercentage=67 ; 67% of target ir (720p for base) use 66.6666667 directly in ini with UE4 patch method for higher accuracy

; Pro Console Standard mode

; Res scale for Pro Console
; Under [PS4_Neo DeviceProfile] ; Pro res
; Find:
r.DynamicRes.MinScreenPercentage=83.3333333 ; same targets as base, just hits higher res more often
r.DynamicRes.MaxScreenPercentage=100
  
; Res scale for Pro Console
; Under [PS4_Neo DeviceProfile] ; Pro res
; Replace:
r.DynamicRes.MinScreenPercentage=50.0000000 ; (540p for pro standard)
r.DynamicRes.MaxScreenPercentage=75 ; (810p for pro standard)
  
; Res scale for Pro Console 4K mode
; Under [PS4_Neo_4k DeviceProfile] ; 4k Pro res
; Find:
r.ScreenPercentage=75 ; 1620p
r.DynamicRes.MinScreenPercentage=74.0740741 ; lowest is 1200p
r.DynamicRes.MaxScreenPercentage=100 ; highest is 1620p

; Res scale for Pro Console 4K mode
; Under [PS4_Neo_4k DeviceProfile] ; 4k Pro res
; Replace: (change it to an untested dynamic 900p mode since we're targetting 60fps)
r.ScreenPercentage=50 ; 1080p
r.DynamicRes.MinScreenPercentage=66.6666667 ; 720p
r.DynamicRes.MaxScreenPercentage=83 ; 900p, use 83.3333333 directly in ini with UE4 patch method for higher accuracy

; end of DynamicRes

; Below are for reference only/.

; [PS4 DeviceProfile] ; What Base PS4 uses.
r.DynamicRes.MinScreenPercentage=83.3333333 ; lowest bound is 83%
r.DynamicRes.MaxScreenPercentage=100 ; highest bound is 1920x1080

; [PS4_Neo_4k DeviceProfile] ; What PS4 Pro uses.
r.ScreenPercentage=75 ; always 2880x1620 by default
r.DynamicRes.MinScreenPercentage=74.0740741 ; lowest is 1200p
r.DynamicRes.MaxScreenPercentage=100 ; highest is 1620p
```

</details>
