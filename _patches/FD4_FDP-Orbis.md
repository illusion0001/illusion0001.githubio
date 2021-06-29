# Dark Souls III: The Fire Fades Edition

## 60FPS Unlock

Author: [illusion](https://github.com/illusion0001)

Only useful for base console.

You must apply FPS Unlock patch to take effect on Base Console.

Pro owners do not need this patch.

In file `eboot.bin`

<details>
<summary>Code 1.03 (Click to Expand)</summary>

```
# VFR
74 1A E8 7C 82 2F 01

EB 0E E8 7C 82 2F 01

# Fliprate
74 1A E8 03 81 2F 01

EB 0E E8 03 81 2F 01
```

</details>

## 720p Resolution

Author: [illusion](https://github.com/illusion0001)

In file `eboot.bin`

<details>
<summary>Code 1.03 (Click to Expand)</summary>

```
# Replace all 3 occurrences.

80 07 00 00 38 04 00 00

00 05 00 00 D0 02 00 00
```

</details>
