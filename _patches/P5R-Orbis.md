# Persona 5: Royal

## 60 FPS

Author: [illusion](https://github.com/illusion0001), [ZEROx](https://github.com/Xcedf)

Patch may accelerate some game effects by 2x.

Game will softlock at centain places with 60FPS Patch enabled. Be sure to frequently save and switch to 30FPS with X + DPad Down to progress further.

In file `eboot.bin`

<details>
<summary>Code for 1.01 (Click to Expand)</summary>

```
# all codes must be applied for patch to work

# write to flag // hack

E8 A2 FE 19 00 5D B0 01 C3 90 90 90 90 90 90 90 90 90

E8 A2 FE 19 00 5D B0 01 C3 C6 05 7F A5 4F 02 3C EB 56

48 8D 7A 14 0F 45 C8 89 4A 2C

48 8D 7A 14 EB A5 90 90 90 90

# call

48 8D 0D 1D 81 4F 02 8B 71 2C E8 45 9E 00 00

67 67 E8 CD 81 05 00 90 90 90 E8 45 9E 00 00

###########################################

# Buttons id (missing dpad):
# int | button
# 2 L3 
# 4 R3
# 2048 R1
# 512 R2
# 1024 L1 
# 256 L2
# 16384 X
# 32768 Square
# 4096 Triangle
# 8192 Circle
# 1048576 Touchpad

###########################################

# switch framerate mode with X + DPad Down

C3 E8 60 03 5F 01 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 55 48 89 E5 41 57 41 56 53 48 83 EC 28 4C 8B 3D A4 25 B6 01 49 89 FE 48 89 F3 48 8D 55 D0 48 8D 4D C0 49 8B 07 48 89 45 E0 48 8B BE C8 00 00 00 48 8D 76 08 48 8B 07 FF 50 10 48 8D 05 D3 D8 E9 01 C5 FA 10 45 D0 C5 FA 10 10 C5 FA 5C C2 C4 E3 79 04 DA 00 C5 EA 58 55 C0 C5 FA 11 45 D0 C5 FB 10

C3 E8 60 03 5F 01 48 8D 0D 49 FF 49 02 EB 08 90 90 90 90 90 90 90 C3 81 3D DD D8 1C 02 40 40 00 00 75 33 81 3D 99 F9 49 02 40 40 00 00 74 35 80 3D 4C FF 49 02 3C 75 05 41 B0 1E EB 03 41 B0 3C 44 88 05 3B FF 49 02 44 88 05 80 F9 49 02 41 B8 40 40 00 00 EB 07 44 8B 05 A1 D8 1C 02 44 89 05 62 F9 49 02 44 8A 05 63 F9 49 02 44 88 05 10 FF 49 02 8B 71 2C C3 10
```

</details>
