# Ghost Of Tsushima

## 60 FPS Unlock

This isn't very useful right now because it only unlocks the framerate, nothing more.

CPU + GPU limited in normal gameplay, meaning you won't go above 32~fps without looking at the sky.

In file `eboot.bin`

<details>
<summary>Code 1.00 (Click to Expand)</summary>

```
39 05 CA B1 01 02 74 16 89 05 C2 B1 01 02

39 05 CA B1 01 02 EB 16 89 05 C2 B1 01 02
```

</details>

<details>
<summary>Code 1.12 (Click to Expand)</summary>

```
39 05 5A DA 18 02 74 16 89 05 52 DA 18 02

39 05 5A DA 18 02 EB 16 89 05 52 DA 18 02
```

</details>
