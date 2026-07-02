# user-skills

測試用 Claude Code plugin，`skills/` 與 `commands/` 目錄目前為空，可自行新增。

## 安裝

```
/plugin marketplace add ayii0111/custom-skills
/plugin install user-skills
```

## 移除

```
/plugin uninstall user-skills
```

若要連 marketplace 來源也移除：

```
/plugin marketplace remove custom-skills
```

## 本機測試

```bash
claude --plugin-dir /Users/ayii/cc-projects/custom-skills
```

或先加入本機路徑作為 marketplace：

```
/plugin marketplace add /Users/ayii/cc-projects/custom-skills
/plugin install user-skills
```
