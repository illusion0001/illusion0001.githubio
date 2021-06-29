# Uncharted 4

To apply patch:

PS4: Open decrypted executable (eboot.bin) in hexeditor and search for first line and replace with second line. Repeat the same for remaining codes.

## Dev Menu

Author: [illusion](https://github.com/illusion0001)

In file `eboot.bin`

<details>
<summary>Code (Click to Expand)</summary>

```
1.00
F6 C2 01 75 34 84 C9 74 30

F6 C2 01 75 34 84 C9 75 30
```

</details>

## Custom Resolution

Author: [illusion](https://github.com/illusion0001)

[Article](https://illusion0001.github.io/patches/2021/04/16/u4-60fps-dream/)

**WARNING**: This patch breaks chapter title cards and will be displayed incorrectly.

[Example](https://cdn.discordapp.com/attachments/650395105479360514/832654624665763850/20210417_011904_00093533.png)

In file `eboot.bin`

<details>
<summary>Code only for 1.00 (Disc Ver.) (Click to Expand)</summary>

```
# framelock 0 (60fps unlock)

C7 83 E4 2F 00 00 01 00 00 00

C7 83 E4 2F 00 00 00 00 00 00

# triple buffering

C7 80 50 0C 00 00 00 00 00 00

C7 80 50 0C 00 00 01 00 00 00

# res hack

48 BA 40 06 00 00 84 03 00 00

48 BA C0 03 00 00 1C 02 00 00 # customize your resolution setting here (960x540)
                              # you may change string "Switch on/off 900p" to match your resolution change

48 BA 00 05 00 00 D0 02 00 00 # (1280x720) for Neo
```

</details>

In file `eboot.bin`

<details>
<summary>How to use (Click to Expand)</summary>

Install custom resolution and dev menu patch above

Open Dev Menu with L3 Touchpad Right, Go into Display -> Frame Settings

Change the following settings:

Enable Switch on/off 900p (light blue -> light red)

(Optional) Disable FPS in msgcon to hide fps statistics.

</details>
