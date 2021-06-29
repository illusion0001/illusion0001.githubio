# Hellblade: Senua's Sacrifice

## Resolution Patch

Author: [illusion](https://github.com/illusion0001)

[Framerate analysis](https://youtu.be/DO8zPsX_ahE)

This game already runs with uncapped framerate.

Only useful for Base Console.

In file `eboot.bin`

<details>
<summary>Code 1.02 (Click to Expand)</summary>

```
# all code below must be applied

# allow screenpercentage to be used
8B 05 77 87 20 03 83 38 01 75 0D

8B 05 77 87 20 03 83 38 01 74 0D

# call
48 8B 05 4C 7D 03 03

67 67 E8 7D 0A 22 00

# main code
E8 0C E1 49 01 90 90 90 90 90 90 90 90 90 90 90 90 55 48 89 E5 41 57 41 56 41 54 53 48 89 D3 45 89

E8 0C E1 49 01 90 90 90 90 90 90 90 90 90 90 90 90 C3 48 8B 05 C8 72 E1 02 C7 00 00 00 86 42 C3 89

# Presets:

# 540p target
C7 00 00 00 48 42 C3 89

# 720p target // default, search for this, then replace with others
C7 00 00 00 86 42 C3 89

# 900p target
C7 00 F6 A8 A6 42 C3 89

# 50.0f  = 00 00 48 42
# 67.0f  = 00 00 86 42
# 83.33f = F6 A8 A6 42
```

</details>

## Skip Startup Videos

Author: [illusion](https://github.com/illusion0001)

[Demo](https://cdn.discordapp.com/attachments/650395105479360514/858528041557557278/SkipMovieDemo.mp4)

Curiously enough, it makes no difference in loading times.

In file `eboot.bin`

<details>
<summary>Code 1.02 (Click to Expand)</summary>

```
48 83 BF A0 00 00 00 00 0F 85 87 06 00 00

48 83 BF A0 00 00 00 00 48 E9 87 06 00 00
```

</details>

## Disable Motion Blur

In file `eboot.bin`

<details>
<summary>Code 1.02 (Click to Expand)</summary>

```
66 A9 01 10 75 40

66 A9 01 10 EB 40
```

</details>
