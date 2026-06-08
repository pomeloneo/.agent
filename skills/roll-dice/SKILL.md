---
name: roll-dice
description: 使用真随机数掷骰子。当被要求掷骰子（d6、d20 等）、投骰子或生成随机骰子结果时使用。
---

To roll a die, use the following command that generates a random number from 1
to the given number of sides:

```bash
shuf -i 1-<sides> -n 1
```

```powershell
Get-Random -Minimum 1 -Maximum (<sides> + 1)
```

Replace `<sides>` with the number of sides on the die (e.g., 6 for a standard
die, 20 for a d20).
