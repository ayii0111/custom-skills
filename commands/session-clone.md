---
name: session-clone
description: 克隆當前 session 到另一個目錄中
argument-hint: [dir_path]
arguments: [dir_path]
disable-model-invocation: true
allowed-tools: Bash
---

先檢查 ~/.claude/projects 下是否有存在 $dir_path 相應的 slug 目錄
若沒有應建立 slug 格式相應的目錄
若有則從 ~/.claude/projects 中找出當前 session 的檔案/目錄，複製一份到 $dir_path 相應的 slug 目錄下
再將複製過去的 session 檔中的 cwd 欄位（注意：欄位名是 cwd，不是 pwd）全都改為 $dir_path

## 執行細節（避免繞路）

- session 檔是 **JSONL**，且原始格式是「緊湊」JSON（`:` 與 `,` 後面沒有空格）。取代 `cwd` 值時優先用純文字/正則取代（例如 `sed` 或字串替換），保留原始格式不變；**不要**用 Python `json.dumps` 重新序列化整份檔案，因為預設 `separators` 會加空格，造成格式與原檔不一致（且會讓後續用原格式 grep 驗證誤判失敗）。
  - 若必須用 json.dumps 重新序列化，務必帶 `separators=(',', ':')`。
- 驗證取代結果時，用 JSON parse 後檢查欄位值（例如逐行 `json.loads` 讀出 `cwd` 值），不要用「精確字串比對」的 grep（如 `"cwd":"..."`），因為空格、跳脫字元等格式差異會讓字串比對誤判為失敗。
