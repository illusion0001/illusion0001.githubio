# Crysis: Remastered

## 30 FPS Limit (Proper Frame-Pacing)

Only use this when you really need to. It will cap Performance Mode on Neo to 30FPS.

Author: [illusion](https://github.com/illusion0001)

In file `eboot.bin`

<details>
<summary>Code for 1.00 (Click to Expand)</summary>

```
# code jump

31 F6 41 8B BF 28 01 00 00 E8 F8 A7 BC 00

EB 78 41 8B BF 28 01 00 00 E8 F8 A7 BC 00

# 0x1 to sceVideoOutSetFlipRate

E8 0E F3 FF FF EB 10 CD 41 E9 58 FF FF FF

E8 0E F3 FF FF EB 10 BE 01 00 00 00 EB 81

## notes:
# Kernel function will provide fixed rate of update time.
# 0x0 16.67ms -- 60hz
# 0x1 33.33ms -- 30hz fix bad frame pacing.
# 0x2 50.00ms -- 20hz
##
```

</details>
