# The Witcher 3

## 60 FPS Unlock

[Article]

Author: [illusion](https://github.com/illusion0001)

In file `eboot.bin`

<details>
<summary>Code for 1.62 (Click to Expand)</summary>

```
B9 80 07 00 00 41 B8 38 04 00 00

B9 00 05 00 00 41 B8 D0 02 00 00
```

</details>

## Resolution Patch

Currently for Base Console, Pro Backbuffer is included but untested.

[Article]

Author: [illusion](https://github.com/illusion0001)

In file `eboot.bin`

<details>
<summary>Code for 1.62 (Click to Expand)</summary>

```
# Base
# targeting 1280x720
B9 80 07 00 00 41 B8 38 04 00 00

B9 00 05 00 00 41 B8 D0 02 00 00

# Neo
B9 80 07 00 00 41 B8 70 08 00 00

# targeting ??x??
```

</details>

## Display Framerate Stats

Author: [illusion](https://github.com/illusion0001)

[Article]

In file `eboot.bin`

<details>
<summary>Code for 1.62 (Click to Expand)</summary>

```
49 89 DC 0F 84 33 01 00 00 C4 C1 7A 10

49 89 DC 0F 85 33 01 00 00 C4 C1 7A 10
```

</details>