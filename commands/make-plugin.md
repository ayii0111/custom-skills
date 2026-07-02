---
name: make-plugin
description: 把本 session 協作出的 CC 輔助功能 plugin 化到當前目錄
disable-model-invocation: true
---

我們這次 session 中，所協力架構出來輔助 claude code 的功能，請幫我進行 claude code 規格的 plugin 化，建構在當前目錄，方便我上傳到 github 分享給其他 cc 用戶

## 執行步驟

1. 讀取 session 內容，整理出這次協作出的 CC 輔助功能（hook、slash command、MCP server 等）
2. 在當前目錄建立合法的 CC plugin 結構：
   - `.claude-plugin/plugin.json`（含 `name`、`marketplace` 等欄位）
   - `.claude-plugin/marketplace.json`
   - hooks 腳本搬進 `.claude-plugin/hooks/` 並更新 hooks.json
   - hooks.json 中的 `"source"` 欄位必須使用 `"github"`，不可用 `"directory"`
   - **每支 hook 腳本開頭**（`#!/usr/bin/env bash` 之後）必須加上 debug 區塊（見下方 [Hook 腳本 Debug 樣板](#hook-script-debug-template)）
3. 建立 `install.sh`（見下方範本），`NAME` 與 `marketplace.name` 必須相同
4. 建立 `uninstall.sh`（見下方範本）
5. 確認 `install.sh` / `uninstall.sh` 都有執行權限（`chmod +x`）

## install.sh 範本

```bash
#!/usr/bin/env bash
set -e
NAME="<plugin-name>"          # ← 替換為實際名稱，需與 marketplace.name 相同
GITHUB_OWNER="ayii0111"

# 本機執行（./install.sh）：用本地路徑
if [ -f "$(dirname "$0")/.claude-plugin/plugin.json" ]; then
  DIR="$(cd "$(dirname "$0")" && pwd)"
  claude plugin marketplace add "$DIR"
else
  # curl | bash：用 GitHub source
  claude plugin marketplace add "${GITHUB_OWNER}/${NAME}"
fi

claude plugin install "${NAME}@${NAME}"
echo "✓ 安裝完成。在 CC 執行 /reload-plugins 或重啟 CC 套用。"
```

> **重要**：`NAME` 必須硬寫，不能靠 `$0` 推導——`curl | bash` 時 `$0` 是 `bash`，路徑偵測會失效。

## uninstall.sh 範本

```bash
#!/usr/bin/env bash
set -e
NAME="<plugin-name>"          # ← 替換為實際名稱

claude plugin uninstall "${NAME}@${NAME}"
claude plugin marketplace remove "$NAME"
echo "✓ 已移除 ${NAME}。重啟 CC 套用。"
```

---

## Hook 腳本 Debug 樣板 {#hook-script-debug-template}

每支 hook 腳本在 `#!/usr/bin/env bash` 之後、主邏輯之前，必須加上以下 debug 區塊：

```bash
exec 2>>"/tmp/claude-hook-$(date '+%Y-%m-%d').log"
echo "[$(date '+%H:%M:%S')] [腳本名] start" >&2
```

將 `[腳本名]` 替換為該腳本的實際名稱（例如 `pre-tool-use`、`post-tool-use`）。

### 範例

```bash
#!/usr/bin/env bash
exec 2>>"/tmp/claude-hook-$(date '+%Y-%m-%d').log"
echo "[$(date '+%H:%M:%S')] [pre-tool-use] start" >&2

# 主邏輯...
```

log 檔案會按日期分割，存放於 `/tmp/claude-hook-YYYY-MM-DD.log`，方便追蹤每日執行紀錄。

---

## 特殊情況：plugin 含 `/command` 技能時繞過 namespace

Plugin 系統會強制把技能命名為 `<plugin-name>:<skill-name>`，若使用者需要乾淨的 `/<command>` 入口（不帶 namespace），須採用以下策略：

### 架構原則

- **plugin 結構裡不建立 `commands/` 目錄與任何 `.md` 檔**，避免 plugin 系統自動登錄 `<plugin-name>:<skill-name>`
- Plugin 只放 `hooks/`（由 plugin 系統管理）
- Command 入口改由安裝/卸載腳本手動管理 `~/.claude/commands/<command-name>.md`

### Command 入口檔內容

手動寫入的 `~/.claude/commands/<command-name>.md` 只需：

```markdown
---
disable-model-invocation: true
---
```

不填 `description`；實際執行邏輯由 plugin 的 `UserPromptExpansion` hook 負責，這個檔案只是讓 CC 認出 `/<command-name>` 指令的入口。

### install.sh（含手動寫入 command 入口）

```bash
#!/usr/bin/env bash
set -e
NAME="<plugin-name>"
GITHUB_OWNER="ayii0111"
CMD="<command-name>"          # ← 替換為實際 command 名稱（通常與 NAME 相同）

# plugin 安裝（同上方標準流程）
if [ -f "$(dirname "$0")/.claude-plugin/plugin.json" ]; then
  DIR="$(cd "$(dirname "$0")" && pwd)"
  claude plugin marketplace add "$DIR"
else
  claude plugin marketplace add "${GITHUB_OWNER}/${NAME}"
fi
claude plugin install "${NAME}@${NAME}"

# 手動寫入 command 入口（繞過 namespace）
COMMANDS_DIR="${HOME}/.claude/commands"
mkdir -p "$COMMANDS_DIR"
cat > "${COMMANDS_DIR}/${CMD}.md" <<'EOF'
---
disable-model-invocation: true
---
EOF

echo "✓ 安裝完成。在 CC 執行 /reload-plugins 或重啟 CC 套用。"
```

### uninstall.sh（含移除 command 入口）

```bash
#!/usr/bin/env bash
set -e
NAME="<plugin-name>"
CMD="<command-name>"          # ← 替換為實際 command 名稱

claude plugin uninstall "${NAME}@${NAME}"
claude plugin marketplace remove "$NAME"

# 移除手動寫入的 command 入口
rm -f "${HOME}/.claude/commands/${CMD}.md"

echo "✓ 已移除 ${NAME}。重啟 CC 套用。"
```
