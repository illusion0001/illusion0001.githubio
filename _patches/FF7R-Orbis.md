# Final Fantasy 7: Remake

## 60 FPS Unlock

[Blog Post](https://illusion0001.github.io/patches/2021/05/20/ff7r-end-60fps/)

In file `...\end\content\paks\pakchunk0-ps4.pak`

<details>
<summary>Code (Click to Expand)</summary>

```ini
; This file must be edited in hex editor,
; normal text editors will add more bytes
; and may cause game crashes.
; You may change 63 to 75% or higher for Pro consoles.
; There are multiple instances of the following lines,
; be sure to change all occurences.
; Neo uses 2880x1620 for highest bound?
; https://youtu.be/qyGl5C3Uwak?t=516

; For end users
; When replacing, only search for cvars
; i.e search for: rhi.SyncInterval=2
; Do not search for comments as they don't exist!

; Find:
rhi.SyncInterval=2 ; 30hz
; Res scale change may only be needed for Base Console, needs testing.
r.DynamicRes.MinScreenPercentage=83.3333333 ; 83% of target ir
r.DynamicRes.MaxScreenPercentage=100 ; 100% of target ir (1080p for base not sure for Neo)

; Replace:
rhi.SyncInterval=1 ; 60hz
; Res scale change may only be needed for Base Console, needs testing.
r.DynamicRes.MinScreenPercentage=50.0000000 ; 50% of target ir (540p for base)
r.DynamicRes.MaxScreenPercentage=63 ; 63% of target ir (720p for base)
```

</details>
