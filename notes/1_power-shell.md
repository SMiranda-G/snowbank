## PowerShell Commands Explained

- `mkdir folder1, folder2, folder3` → Creates multiple folders at once. Works in **PowerShell** only (CMD fails).  
- `mkdir parent/child` → Creates nested folder directly (e.g., `.github/workflows`).
- `ni filename` → `ni` = `New-Item`. Defaults to `-ItemType File` (creates empty file).
  - Other `-ItemType` values: `Directory`, `SymbolicLink`, `HardLink`, `Junction`.
  - Explicit usage: `New-Item -ItemType Directory -Path myFolder`.