# The Order 1886

## 60 FPS Unlock

This isn't very useful right now because it only unlocks the framerate, nothing more.

GPU limited in normal gameplay.

In file `eboot.bin`

<details>
<summary>Code 1.02 (Click to Expand)</summary>

```
CF 01 00 8B 1D 3A 8B D4 00

CF 01 00 C7 C3 01 00 00 00
```

</details>

## Resolution Patch (Non Working)

Borked! Won't boot below pre defined res.

Only for developers.

1920x1080 only.

In file `eboot.bin`

<details>
<summary>Code 1.02 (Click to Expand)</summary>

```
41 8B 84 24 4C 01 00 00 41 8B 8C 24 50 01 00 00 48

# 1920x1080 // default, game will boot.

48 C7 C0 80 07 00 00 48 C7 C1 38 04 00 00 90 90 48

# 1600x900 // game won't boot.

48 C7 C0 40 06 00 00 48 C7 C1 84 03 00 00 90 90 48
```

</details>
