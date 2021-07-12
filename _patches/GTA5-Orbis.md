# Grand Theft Auto V

## Unlock FPS (CUSA00411)

Completely unlocks Framerate.

[Video](https://media.discordapp.net/attachments/474327933700276234/864138722719039548/0_45.mp4)

Author: [illusion](https://github.com/illusion0001)

In file `eboot.bin`

<details>
<summary>Code (Click to Expand)</summary>

```
BF 02 00 00 00 31 F6 E8 87 67 CB 00

BF 01 00 00 00 31 F6 E8 87 67 CB 00

# do not apply
# this will introduce screen tear when param is loaded with 0
# 2 will become 20hz
# 1 becomes 30hz
# 0 is unlocked
# code will skip sceVideoOutSetFlipRate
# 0F 44 F7 8B 3D B8 6D A4 01 E9 93 52 A6 00
# 0F 45 F7 8B 3D B8 6D A4 01 E9 93 52 A6 00
```

</details>
