# 本專案目的
- 主要是為了，自研 skill 時，可以適應 skill 編寫靈感，避免編寫心流中斷的管理目錄

## skill 規格
---
name: xxx-xxx
description: 功能描述
argument-hint: [arg]
arguments: [arg]
allowed-tools: Read, Write, Edit, Grep, WebFetch, Bash
---

## 輔助工具
- 一鍵開啟 skill 編寫: 一鍵在 ide 編輯器打開 draf-skills 目錄
  - `alias ds='code ~/cc-projects/draf-skills'`
- 安裝指令: 應建立一個 terminal 指令，可以一鍵將當前編寫好 draf-skill 安裝到 cc 系統中

- 改名指令: 事後要對 cc 系統安裝的 skill 改名，也使用指令修改


### skill 有效觸發範本
- 兩者共享 1,536 字元上限
```
description: 該工具做可以做什麼
when_to_use: 在什麼情況下使用
```

**skill 觸發時機的 Debug**: ```你何時會使用 xxx skill?```



## 本機測試

```bash
claude --plugin-dir /Users/ayii/cc-projects/custom-skills
```

或先加入本機路徑作為 marketplace：

```
/plugin marketplace add /Users/ayii/cc-projects/custom-skills
/plugin install custom-skills
```
