# Life is Strange

## 60 FPS (CUSA03279)

Author: [illusion](https://github.com/illusion0001)

[Article](https://illusion0001.github.io/patches/2021/05/11/wif-dawn-60fps/)

[Example of CPU Limited Scenario](https://cdn.discordapp.com/attachments/650395105479360514/840144551825244180/20210507_191348_00757258.png)

In file `eboot.bin`

<details>
<summary>Code (Click to Expand)</summary>

```

# 60fps unlock

B0 01 C5 F9 5C 05 DF 37 7A 01 C5 F9 7C C0 C4 C1 7B 5C 07 C4 C1 7B 59 45 00 C5 FB 5C C1 C5 FA 10 0D 30 39 7A 01 C5 FB 5A C0 C5 F8 2E C8 77 AA

B0 00 C5 F9 5C 05 DF 37 7A 01 C5 F9 7C C0 C4 C1 7B 5C 07 C4 C1 7B 59 45 00 C5 FB 5C C1 C5 FA 10 0D 30 39 7A 01 C5 FB 5A C0 C5 F8 2E C8 90 90

# minor performance improvement

C5 FA 10 40 0C 31 C0 C5 FA

E8 05 B6 18 00 31 C0 C5 FA

55 48 89 E5 41 57 41 56 41 55 41 54 53 48 81 EC F8 06 00 00 48 8B 0D 15

C3 C7 40 0C 00 00 00 3E C5 FA 10 40 0C C6 40 1C 00 C3 00 00 48 8B 0D 15
```

</details>

## 60 FPS (CUSA01442)

Author: [illusion](https://github.com/illusion0001)

In file `eboot.bin`

<details>
<summary>Code 1.07 (Click to Expand)</summary>

```
# 60fps unlock

B0 01 C5 F8 2E C8 77 AA

B0 00 C5 F8 2E C8 90 90
```

</details>
