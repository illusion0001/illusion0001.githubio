# Red Dead Redemption 2

## 60 FPS Unlock

[Demo](https://youtu.be/zSlrdwxP4yA)

CPU Limited. For use with Prospero.

Author: [illusion](https://github.com/illusion0001)

In file `eboot.bin`

<details>
<summary>Code 1.02 (Click to Expand)</summary>

```
85 C0 0F 44 F0 E8 89 A8 3F 01

BE 00 00 00 00 E8 89 A8 3F 01
```

</details>

<details>
<summary>Code 1.24 (Click to Expand)</summary>

```
85 C0 0F 44 F0 E8 F4 7A 47 01

BE 00 00 00 00 E8 F4 7A 47 01
```

</details>

## Resolution Patch

[Demo](https://youtu.be/zSlrdwxP4yA)

Author: [illusion](https://github.com/illusion0001)

In file `eboot.bin`

<details>
<summary>Code (Click to Expand)</summary>

```
# Base
30 31 32 33 34 35 36 37 38 39 3A 3B 3C 3D 3E 3F 80 07 00 00 38 04 00 00 3C 00 00 00 01 00 00 00
# 1920x1080 -> 1280x720
30 31 32 33 34 35 36 37 38 39 3A 3B 3C 3D 3E 3F 00 05 00 00 D0 02 00 00 3C 00 00 00 01 00 00 00

# Neo
30 31 32 33 34 35 36 37 38 39 3A 3B 3C 3D 3E 3F 80 07 00 00 38 04 00 00 3C 00 00 00 01 00 00 00 00 0F 00 00 70 08 00 00 3C 00 00 00 01 00 00 00
# 3840x2160 -> ????x????
# array of interest 00 0F 00 00 70 08 00 00
```

</details>
