# Uncharted: The Lost Legacy

To apply patch:

PS4: Open decrypted executable (eboot.bin) in hexeditor and search for first line and replace with second line. Repeat the same for remaining codes.

## 720p + 60 FPS Unlock

Author: [illusion](https://github.com/illusion0001)

Note: You'll not be able to set resolution lower than 720p. Possibly SDK Limits. 

Patch is untested on PS4 Pro HW, likely needs adjustments due to separate buffers setup.

In file `eboot.bin`

<details>
<summary>Code for 1.00 (Disc Ver.) (Click to Expand)</summary>

```
# framelock 0 (60fps unlock)

C7 83 64 30 00 00 01 00 00 00

C7 83 64 30 00 00 00 00 00 00

# triple buffering

C7 05 9A 86 C8 02 00 00 00 00

C7 05 9A 86 C8 02 01 00 00 00

# main buffer

83 3D 01 CC C4 02 00 BB 38 04 00 00 BF 80 07 00 00

83 3D 01 CC C4 02 00 BB D0 02 00 00 BF 00 05 00 00

# back buffer

49 BF 80 07 00 00 38 04 00 00

49 BF 00 05 00 00 D0 02 00 00

# front buffer

80 07 00 00 38 04 00 00 40

00 05 00 00 D0 02 00 00 40

```

</details>
