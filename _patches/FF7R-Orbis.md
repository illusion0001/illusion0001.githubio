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

## Texture Streaming Improvements

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

Additional Patches by [@dontellmama](https://github.com/dontellmama)

[Source](https://github.com/dontellmama/Improve-Final-Fantasy-VII-Remake/blob/d9ac423934da27aff97aa512df47906016b8c3d3/README.md).

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

[TextureStreaming]
r.Streaming.PoolSize=2000 ; Streaming Pool Size too large(for example, 2600) will crash. Cause PS4 total RAM too small, RAM and VRAM share 8G (approximately 5G available for games)
r.Streaming.MaxTempMemoryAllowed=40
MemoryMargin=10

; Add in file ...\Engine\Config\PS4\PS4DeviceProfiles.ini

+CVars=r.Streaming.MipBias=0
+CVars=r.MaxAnisotropy=16 ; AF 16X
+CVars=r.Streaming.PoolSize=2000
+CVars=r.Streaming.MaxEffectiveScreenSize=0

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
