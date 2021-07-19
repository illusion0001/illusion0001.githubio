# The Outer Worlds

## 60 FPS Unlock + Resolution Patch (CUSA13689)

Author: [illusion](https://github.com/illusion0001)

In file `eboot.bin`

<details>
<summary>Code 1.00 (Click to Expand)</summary>

```
# call 1

31 C9 45 8B 3C 8F C4 C1 6B 2A C7

31 C9 67 67 E8 8E 90 CE 01 90 90

# call 2

0F 95 C1 C5 FA 10 04 8B C5 FA 10 4D EC

0F 95 C1 E8 15 26 81 00 C5 FA 10 4D EC

# main code
# 720p target for base

55 48 89 E5 41 57 41 56 41 55 41 54 53 48 81 EC 78 03 00 00 48 8B 05 4D A0 81 02 48 89 95 98 FC FF FF 45 89 C6 41

C7 04 8B 00 00 86 42 C5 FA 10 04 8B EB 11 41 C7 04 8F 01 00 00 00 45 8B 3C 8F C4 C1 6B 2A C7 C3 FF FF 45 89 C6 41

# 900p target for PS4 Pro, untested.
# apply code above first then search for this.

C7 04 8B 00 00 86 42

C7 04 8B 00 00 7C 42
```

</details>
