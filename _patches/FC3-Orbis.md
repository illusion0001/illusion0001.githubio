# Fry Cry 3

## 60 FPS Unlock

CPU Limited.

Author: [illusion](https://github.com/illusion0001)

In file `eboot.bin`

<details>
<summary>Code 1.02 (Click to Expand)</summary>

```
BE 01 00 00 00 E8 C1 A5 23 01

BE 00 00 00 00 E8 C1 A5 23 01
```

</details>

## Resolution Patch

Author: [illusion](https://github.com/illusion0001)

In file `eboot.bin`

<details>
<summary>Code 1.02 (Click to Expand)</summary>

```
# Base backbuffers
41 BE 80 07 00 00 BB 38 04 00 00

41 BE 00 05 00 00 BB D0 02 00 00

# Pro backbuffers
# experminent for yourself what works best.

# Height
48 89 18 E8 5A A9 23 01 83 F8 01 B8 00 0A 00 00

# Width
B8 A0 05 00 00 C4 C1 7A 2A C6 0F 44 D8 44 89 35 E7 2D
```

</details>
