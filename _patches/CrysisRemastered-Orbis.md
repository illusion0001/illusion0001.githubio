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

# write 0 to sys_maxfps
# this is needed because it will interfere with sceVideoOutSetFlipRate
# engine is still trying to cap to inproper 30 fps.

# call
8B 05 86 9D A8 01

48 E8 1B 32 B7 00

54 6F 67 67 6C 65 73 20 73 75 6E 6C 69 67 68 74 20

8B 05 65 6B F1 00 C6 05 2E 13 9B 01 00 C3 68 74 20

## notes:
# Kernel function will provide fixed rate of update time.
# 0x0 16.67ms -- 60hz
# 0x1 33.33ms -- 30hz fix bad frame pacing.
# 0x2 50.00ms -- 20hz
##
```

</details>
