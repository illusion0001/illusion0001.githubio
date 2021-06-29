# Ghost Of Tsushima

## 60 FPS Unlock

Author: [illusion](https://github.com/illusion0001)

This isn't very useful right now because it only unlocks the framerate, nothing more.

GPU limited in normal gameplay, meaning you won't go above 32~fps without looking at the sky.

**Must be used with [resolution patch](#resolution-patch)**

In file `eboot.bin`

<details>
<summary>Code 1.00 (Click to Expand)</summary>

```
39 05 CA B1 01 02 74 16 89 05 C2 B1 01 02

39 05 CA B1 01 02 EB 16 89 05 C2 B1 01 02
```

</details>

<details>
<summary>Code 1.12 (Click to Expand)</summary>

```
39 05 5A DA 18 02 74 16 89 05 52 DA 18 02

39 05 5A DA 18 02 EB 16 89 05 52 DA 18 02
```

</details>

## Resolution Patch

Author: [illusion](https://github.com/illusion0001)

This isn't very useful right now because it breaks occlusion on Base Console. (See notes)

Pro works (reported by tester.)

**Must be used with [framerate patch](#60-fps-unlock)**!

In file `eboot.bin`

<details>
<summary>Code 1.00 (Base Only) (Click to Expand)</summary>

```
0F 84 7C 00 00 00 48 B8 00 0F 00 00 70 08 00 00 48 B9 80 0C 00 00 08 07 00 00 48 89 05 D7 7D 2F 01 48 89 0D DC 7D 2F 01 C6 05 0E B5 02 02 01

0F 85 7C 00 00 00 48 B8 80 07 00 00 38 04 00 00 48 B9 C0 03 00 00 1C 02 00 00 48 89 05 D7 7D 2F 01 48 89 0D DC 7D 2F 01 C6 05 0E B5 02 02 00

# some notes:
# jnz to use pro path on base, (this doesn't fix culling)
# https://cdn.discordapp.com/attachments/842452358968508446/851187186011340871/0_42.mp4
# using pro path allows for full control over buffer render targets.
# as base use rcx for both front and back, this can be separated of course
# but would reqiure extra code
### set front buffer to 1920x1080
# 48 B8 00 0F 00 00 70 08 00 00
# 48 B8 80 07 00 00 38 04 00 00
###
### set back buffer to 960x540
# 48 B9 80 0C 00 00 08 07 00 00
# 48 B9 C0 03 00 00 1C 02 00 00
###
### disable temporal upscale
## otherwise you'll get this:
# https://cdn.discordapp.com/attachments/650395105479360514/854487869535551508/20210616_090846_00496201.png
# https://cdn.discordapp.com/attachments/650395105479360514/854487883100192788/20210616_090923_00844421.png
# C6 05 0E B5 02 02 01
# C6 05 0E B5 02 02 00
###
```

</details>

<details>
<summary>Code 1.00/1.12 (Pro Only) (Click to Expand)</summary>

```
# set back buffer to 1600x900
48 B9 80 0C 00 00 08 07 00 00

48 B9 40 06 00 00 84 03 00 00
```

</details>

## Disable Temporal Upscale

**Must be used with [resolution patch](#resolution-patch)**

In file `eboot.bin`

<details>
<summary>Code 1.00 (Click to Expand)</summary>

```
C6 05 0E B5 02 02 01

C6 05 0E B5 02 02 00
```

</details>

<details>
<summary>Code 1.12 (Click to Expand)</summary>

```
C6 05 98 D3 19 02 01

C6 05 98 D3 19 02 00
```

</details>
