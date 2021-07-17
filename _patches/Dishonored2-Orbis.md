# Dishonored 2

## 60FPS

Author: [illusion](https://github.com/illusion0001)

In file `eboot.bin`

<details>
<summary>Code 1.05 (Click to Expand)</summary>

```
BE 01 00 00 00 E8 42 CC 9C 01

BE 00 00 00 00 E8 42 CC 9C 01
```

</details>

## Resolution Patch

Author: [illusion](https://github.com/illusion0001)

In file `eboot.bin`

<details>
<summary>Code 1.05 (Click to Expand)</summary>

```
# Call

C5 FA 10 1D 67 C4 65 03

E8 7F 09 64 FF EB 09 03

# Res adjustment code
# 67% of 1920x1080

55 48 89 E5 41 57 41 56 53 48 81 EC 28 02 00 00 4C 8B 3D C1 14 D5 02 48 8D 9D C0 FD FF FF 4C 8D B5 E0

C3 C7 05 11 BB 01 04 1F 85 2B 3F C5 FA 10 1D 09 BB 01 04 C5 FA 10 15 01 BB 01 04 C3 FF FF 4C 8D B5 E0

# Some notes
# I hardcoded horizontal scale to vertical
# to remove 
# C5 FA 10 15 01 BB 01 04
# C1 14 D5 02 48 8D 9D C3
# Neo will need adjustments.
# Line to change
# C7 05 11 BB 01 04 1F 85 2B 3F
# 0.67f
```

</details>
