# The Order 1886

## 60 FPS Unlock

Author: [illusion](https://github.com/illusion0001)

[Article](https://illusion0001.github.io/patches/2021/06/27/oodle-res-framerate-patches/)

In file `eboot.bin`

<details>
<summary>Code 1.02 (Click to Expand)</summary>

```
CF 01 00 8B 1D 3A 8B D4 00

CF 01 00 C7 C3 01 00 00 00
```

</details>

## Resolution Patch

Author: [illusion](https://github.com/illusion0001)

[Article](https://illusion0001.github.io/patches/2021/06/27/oodle-res-framerate-patches/)

In file `eboot.bin`

<details>
<summary>Code 1.02 (Click to Expand)</summary>

```
# launch param -use720p
# "Use 720p resolution in fullscreen mode"
# SDK limits not letting us go below 720p.

FC FF FF A8 80 74 41

FC FF FF A8 80 75 41

# For 16:9 to work; 1080p will cause visual issues
# 900p or below is a must.
# 720p doesn't need additional changes.
# 1280x720 -> 1600x900

75 41 C7 85 84 FC FF FF 00 05 00 00 C7 85 88 FC FF FF D0 02 00 00

75 41 C7 85 84 FC FF FF 40 06 00 00 C7 85 88 FC FF FF 84 03 00 00

# launch param -use4k
# "Use 4K resolution in fullscreen mode"
# SDK limits might come back to haunt us.

EB 41 F6 C4 02 74 46

EB 41 F6 C4 02 75 46
```

</details>

## 16:9 Aspect Ratio (Full Screen)

Author: [illusion](https://github.com/illusion0001)

[Article](https://illusion0001.github.io/patches/2021/06/27/oodle-res-framerate-patches/)

In file `eboot.bin`

<details>
<summary>Code 1.02 (Click to Expand)</summary>

```
# Native 1080p will cause visual issues, 900p or below must be used.
# 720p doesn't need additional changes.

C7 85 18 FC FF FF 9A 99 19 40

C7 85 18 FC FF FF 39 8E E3 3F
```

</details>
