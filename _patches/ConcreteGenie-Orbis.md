# Concrete Genie

## 60 FPS

Author: [illusion](https://github.com/illusion0001)

Patch version: 1.1

- Added support for PS4 Pro.

In file `eboot.bin`

<details>
<summary>Code for 1.00 (Click to Expand)</summary>

```
# call
48 8B 05 EB A6 3F 03 C5 F0 57 C9 C5 FA 10 00 C5

48 8B 05 EB A6 3F 03 E8 0F EC 23 00 90 90 90 C5

# write floats
55 48 89 E5 48 8D 05 7D BC 1C 03 5D C3 90 90 90 55 48 89 E5 41 57 41 56 41 55 41 54 53 48 81 EC 08 02 00 00 48 8B 05 E5 C3 D0 02

# Replace with one of the following presets:

# base (720p) (66.667% of 1920x1080)

55 48 89 E5 48 8D 05 7D BC 1C 03 5D C3 90 90 90 C3 C7 00 55 55 85 42 EB 06 C7 00 00 00 70 42 C5 F0 57 C9 C5 FA 10 00 C3 00 C3 02

# Pro (neo) 1080p in 4k output or supersampling mode (50% of 3840x2160)

55 48 89 E5 48 8D 05 7D BC 1C 03 5D C3 90 90 90 C3 C7 00 00 00 48 42 EB 06 C7 00 00 00 70 42 C5 F0 57 C9 C5 FA 10 00 C3 00 C3 02

# Pro (neo) 900p in 1080p output (83.33% of 1920x1080)

55 48 89 E5 48 8D 05 7D BC 1C 03 5D C3 90 90 90 C3 C7 00 AA AA A6 42 EB 06 C7 00 00 00 70 42 C5 F0 57 C9 C5 FA 10 00 C3 00 C3 02

# call2
48 8B 05 69 35 1B 03 C5 F0 57 C9 C5 FA 10 00 C5

48 8B 05 69 35 1B 03 E8 C5 74 FE FF 90 90 90 C5
```

</details>

<details>
<summary>Code for 1.06 (Click to Expand)</summary>

```
# call
48 8B 05 EB 80 40 03 C5 F0 57 C9 C5 FA 10 00

48 8B 05 EB 80 40 03 E8 5F EC 23 00 90 90 90

# write floats

48 8D 05 2D 96 1D 03 5D C3 90 90 90 55 48 89 E5 41 57 41 56 41 55 41 54 53 48 81 EC 08 02 00 00 48 8B 05 25 B4 D1 02

# Replace with one of the following presets:

# base (720p) (66.667% of 1920x1080)

48 8D 05 2D 96 1D 03 5D C3 90 90 90 C3 C7 00 55 55 85 42 EB 08 C7 00 00 00 70 42 EB 00 C5 F0 57 C9 C5 FA 10 00 C3 02

# Pro (neo) 1080p in 4k output or supersampling mode (50% of 3840x2160)

48 8D 05 2D 96 1D 03 5D C3 90 90 90 C3 C7 00 00 00 48 42 EB 08 C7 00 00 00 70 42 EB 00 C5 F0 57 C9 C5 FA 10 00 C3 02

# Pro (neo) 900p in 1080p output (83.33% of 1920x1080)

48 8D 05 2D 96 1D 03 5D C3 90 90 90 C3 C7 00 AA AA A6 42 EB 08 C7 00 00 00 70 42 EB 00 C5 F0 57 C9 C5 FA 10 00 C3 02

# call2 

48 8B 05 19 0F 1C 03 C5 F0 57 C9 C5 FA 10 00

48 8B 05 19 0F 1C 03 E8 C5 74 FE FF 90 90 90
```

</details>

## Disable Motion Blur

Author: [illusion](https://github.com/illusion0001)

In file `eboot.bin`

<details>
<summary>Code for 1.00/1.06 (Click to Expand)</summary>

```
48 8B 0B 8B 41 38 F7 D0 66 A9 01 10 75 58

48 8B 0B 8B 41 38 F7 D0 66 A9 01 10 EB 58
```

</details>
