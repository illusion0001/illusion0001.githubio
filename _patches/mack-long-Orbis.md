# Crash Bandicoot N. Sane Trilogy

## 60FPS Unlock

Author: [illusion](https://github.com/illusion0001)

Uses double buffer causes hardlock to 30, 40 and 60FPS.

Not useful at the moment.

In file `eboot.bin`

<details>
<summary>Code 1.07 (Click to Expand)</summary>

```
# sceVideoOutSetFlipRate

E8 2D E3 D0 00

90 90 90 90 90
```

</details>
