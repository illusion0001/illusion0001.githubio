# Lichdom: Battlemage

## 30 FPS Limit (Proper Frame-Pacing)

Author: [illusion](https://github.com/illusion0001)

In file `eboot.bin`

<details>
<summary>Code (Click to Expand)</summary>

```
# code jump

31 F6 E8 B1 5D 1E 01

EB 5D E8 B1 5D 1E 01

# 0x1 to sceVideoOutSetFlipRate

E8 73 4C 1E 01 90 66 90 B8 FF FF FF FF C3 66 2E 0F 1F 84 00 00 00 00 00

E8 73 4C 1E 01 90 66 90 B8 FF FF FF FF C3 C3 BE 01 00 00 00 EB 9C 00 00

## notes:
# Kernel function will provide fixed rate of update time.
# 0x0 16.67ms -- 60hz
# 0x1 33.33ms -- 30hz fix bad frame pacing.
# 0x2 50.00ms -- 20hz
##
```

</details>

## Resolution Patch

Author: [illusion](https://github.com/illusion0001)

In file `eboot.bin`

<details>
<summary>Code (Click to Expand)</summary>

```
BE 40 06 00 00 48 8B BB 08 7B 00 00 48 8B 07 FF 50 48 48 8B BB 10 7B 00 00 BE 84 03 00 00

# 1280x720
# CE has internal scaler meaning it can go below 1280x720
# Going above 1600x900 is probably possible.

BE 00 05 00 00 48 8B BB 08 7B 00 00 48 8B 07 FF 50 48 48 8B BB 10 7B 00 00 BE D0 02 00 00
```

</details>

<!-- 
# dev note:
# when increasing beyond 1600x900
# defines must be changed to match other backbuffer!
# does it even work? 
-->
