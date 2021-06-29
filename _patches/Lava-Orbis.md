# Crash Bandicoot 4: It's About Time

## Resolution Patch

Author: [illusion](https://github.com/illusion0001)

This game runs with an uncapped framerate on Orbis. Same can't be said for Durango though.

In file `eboot.bin`

<details>
<summary>Code 1.04 (Click to Expand)</summary>

```
48 8D 15 0D 08 BF 04 31 C9 3B 02 0F 95 C1 EB 7A

# 720p for base

31 C9 C7 04 8B 7F AA A6 42 48 E9 7B 00 00 00 90

# 900p for Pro

31 C9 C7 04 8B 0A 57 85 42 48 E9 7B 00 00 00 90
```

</details>
