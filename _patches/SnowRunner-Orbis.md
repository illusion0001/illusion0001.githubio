# SnowRunner

## 60FPS Unlock

This game uses double buffer vsync.

Author: [illusion](https://github.com/illusion0001)

In file `eboot.bin`

<details>
<summary>Code 1.00 (Click to Expand)</summary>

```
44 8B 70 3C 44 3B B3 C4 50 04 00 74 1C

44 8B 70 3C 44 3B B3 C4 50 04 00 75 1C
```

</details>

<details>
<summary>Code 1.12 (Click to Expand)</summary>

```
48 8B 05 FE 16 D0 00 8B 58 44 41 3B 9C 24 EC 55 04 00 74 1E

48 8B 05 FE 16 D0 00 8B 58 44 41 3B 9C 24 EC 55 04 00 75 1E
```

</details>
