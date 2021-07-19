# Prey (2017)

## 60 FPS Unlock

Author: [illusion](https://github.com/illusion0001)

In file `PreyDll.prx`
    
<details>
<summary>Code 1.10 (Click to Expand)</summary>

```
48 8B 45 90 8B 7B 70 8B 70 48

48 8B 45 90 8B 7B 70 31 F6 90
```

</details>

## Resolution Patch

Author: [illusion](https://github.com/illusion0001)

In file `PreyDll.prx`

<details>
<summary>Code 1.10 (Click to Expand)</summary>

```
# call

48 C7 83 48 5D 00 00 00 00 00 00 8B 83 58 5E 00 00

48 C7 83 48 5D 00 00 00 00 00 00 E8 FF 6E E7 FF 90

# main code

55 48 89 E5 41 57 41 56 41 54 53 48 83 EC 20 4C 8B 3D AA 67 81 01 48 89 FB 49 8B 07 48 89 45 D8 48 8B 03 FF

# For Base (960x540)

C3 C7 83 58 5E 00 00 C0 03 00 00 C7 83 5C 5E 00 00 1C 02 00 00 8B 83 58 5E 00 00 C3 48 89 45 D8 48 8B 03 FF

# For Neo (1600x900)

C3 C7 83 58 5E 00 00 40 06 00 00 C7 83 5C 5E 00 00 84 03 00 00 8B 83 58 5E 00 00 C3 48 89 45 D8 48 8B 03 FF
```

</details>
