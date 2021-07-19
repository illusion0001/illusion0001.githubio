# Gravity Rush 2

## 60 FPS Unlock (NON-WORKING) (WORK-IN-PROGRESS)

```
Help wanted! Find a way to disable internal frame limter.
Pull Request is always welcome.

1. Internal Frame Limter caps out at 32 FPS.

2. sceVideoOutSetFlipRate 0x1 for new frame every 33.33ms.
   They likely used kernel function for proper frame-pacing.
```

Author: [illusion](https://github.com/illusion0001)

In file `eboot.bin`

<details>
<summary>Code 1.00 (Click to Expand)</summary>

```
BE 01 00 00 00 E8 86 F2 F6 00

BE 00 00 00 00 E8 86 F2 F6 00
```

</details>
