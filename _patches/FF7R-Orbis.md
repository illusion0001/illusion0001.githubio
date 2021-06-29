# Final Fantasy VII: Remake

## 60 FPS + Resolution Patch (ASM Ver.)

Author: [illusion](https://github.com/illusion0001)

In file `eboot.bin`

<details>
<summary>Code 1.00 (Click to Expand)</summary>

```
8B 1C 8B C5 EB 2A C3

67 67 E8 C8 AD E3 01

C5 FA 10 40 04 C5 F2 2A 0D 6F 47 3A 04

E8 24 7E 5D 01 C5 F2 2A 0D 6F 47 3A 04

55 48 89 E5 41 57 41 56 41 55 41 54 53 48 81 EC E8 01 00 00 48 8B 1D E5 5F 6F 02 48 8B 03 48 89 45 D0 48 8B 7A 50 48 85 FF 75 22

C3 C6 04 8B 01 8B 1C 8B C5 EB 2A C3 C3 C7 40 04 81 55 85 42 C5 FA 10 40 04 C3 02 48 8B 03 48 89 45 D0 48 8B 7A 50 48 85 FF 75 22

# Presets:

# 720p for 1080p Output (Base)

81 55 85 42 -> 81 55 85 42 # 66.67f // no change needed.

# 900p for 1080p Output (Pro)

81 55 85 42 -> F6 A8 A6 42 # 83.33f

# 972p for 4K Output (Pro) // slightly higher than 900p because rounding issues.

81 55 85 42 -> 00 00 34 42 # 45.00f, 44.44f would be much closer to 900p though.

# 1080p for 4K Output (Pro)

81 55 85 42 -> 00 00 48 42 # 50.00f

```

</details>

## Config INI Tweaks

Below are **optional** config ini tweaks. If you only want resolution and framerate patches, use eboot patch above.

## 60 FPS Patch

Author: [illusion](https://github.com/illusion0001)

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

Author: [illusion](https://github.com/illusion0001), [FlyingBananaTree](https://github.com/FlyingBananaTree), [dontellmama](https://github.com/dontellmama)

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

Author: [illusion](https://github.com/illusion0001)

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

## Texture Streaming Improvements

Author: [dontellmama](https://github.com/dontellmama)

May cause out of memory. Use at your own risk.

In file `...\end\content\paks\pakchunk0-ps4.pak`

<details>
<summary>Code (Click to Expand)</summary>

```ini
; https://github.com/illusion0001/illusion0001.github.io/commit/6d72ffa2a1fe389a3d614eb6756bea80b51439a4#commitcomment-51227749
; https://github.com/dontellmama

; Find:
; Under [PS4_Neo_4k DeviceProfile]
 r.Streaming.PoolSize=1350
; Under [PS4 DeviceProfile]
 r.Streaming.PoolSize=1300

; Replace:

r.Streaming.PoolSize=2400 ; 2600 will crash(CE-34787), 2000-2400 fine

; Find:

r.Streaming.MaxTempMemoryAllowed=35

; Replace:

r.Streaming.MaxTempMemoryAllowed=40

; Find:

MemoryMargin=5

; Replace:

MemoryMargin=10
```

</details>

## Optional Graphical Patches

Author: [dontellmama](https://github.com/dontellmama)

These will need to be added manually and cannot be done in hex editor.

More info about extracting and repacking pak [here](https://web.archive.org/web/20210424045205/https://gbatemp.net/threads/how-to-unpack-and-repack-unreal-engine-4-files.531784/).

Extract from `...\end\content\paks\pakchunk0-ps4.pak`

File to modify `...\Engine\Config\PS4\PS4DeviceProfiles.ini`

FF7R uses version 4.

<!-- <details>
<summary>Code (Click to Expand)</summary>

```ini
; Add in file ...\Engine\Config\PS4\PS4DeviceProfiles.ini
; use sg. prefix for presets or manual adjust, see BaseScalability.ini
; for more info.
+CVars=sg.AntiAliasingQuality=0 ; Hair and other Alpha to Coverage will flicker!
                                ; Do not use.
+CVars=sg.EffectsQuality=0      ; lower fx level
+CVars=sg.ViewDistanceQuality=0 ; pop in, improve performance slightly in cpu limited scene
+CVars=sg.PostProcessQuality=0  ; disable most post fx
```

</details> -->

<details>
<summary>Code (Click to Expand)</summary>

```ini
; all tested
; https://github.com/dontellmama

; Add in file ...\Engine\Config\PS4\PS4DeviceProfiles.ini

; disabled due to reported out of memory.
; --------------------------------------

; +CVars=r.Streaming.PoolSize=2000 ; Streaming Pool Size too large(for example, 2600) will crash.
                                   ; Cause PS4 total RAM too small, RAM and VRAM share 8G 
                                   ; (approximately 5G available for games)
; +CVars=r.Streaming.MaxTempMemoryAllowed=40
; +CVars=MemoryMargin=10
; +CVars=r.Streaming.PoolSize=2000
; +CVars=r.Streaming.MaxEffectiveScreenSize=0

; --------------------------------------

+CVars=r.MaxAnisotropy=16 ; AF 16X
+CVars=r.PostProcessAAQuality=3 ; default value 4 TAA too blur, value 3 balance more than other
+CVars=r.MotionBlurQuality=0 ; disable Motion Blur
+CVars=r.AmbientOcclusionMipLevelFactor=0.4 ; improve AO.
+CVars=r.AmbientOcclusionMaxQuality=100 ; improve AO
+CVars=r.AmbientOcclusionLevels=-1 ; improve AO
+CVars=r.AmbientOcclusionRadiusScale=1.0 ; improve AO
+CVars=r.DepthOfFieldQuality=2 ; DOF so far so good
+CVars=r.SceneColorFringeQuality=0 ; remove blur
+CVars=r.Tonemapper.GrainQuantization=0 ; remove grain
+CVars=r.Tonemapper.Quality=0 ; remove grain
+CVars=r.DetailMode=2 ; improve detail
+CVars=r.MaterialQualityLevel=1 ; improve material
```

</details>
