# Batman: Arkham Knight

## 60 FPS Unlock

Author: [illusion](https://github.com/illusion0001)

In file `eboot.bin`

<details>
<summary>Code for 1.14 (Click to Expand)</summary>

```
# 60fps
BE 01 00 00 00 E8 28 97 BF 01

BE 00 00 00 00 E8 28 97 BF 01

# prevent game from speedup above 30fps
0B 88 54 05 00 00 0F 84 A9 00 00 00

0B 88 54 05 00 00 0F 85 A9 00 00 00
```

</details>
