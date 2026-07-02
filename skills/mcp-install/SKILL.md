---
name: mcp-install
description: 協助在 Claude Code 中安裝遠程（remote / HTTP / SSE transport）MCP server。當用戶說「幫我裝 XX 的 MCP」「我要串某個 remote MCP server」「這個 MCP 網址怎麼接到 cc」「MCP 裝不起來 / token 怎麼設」等情境時使用。涵蓋 claude mcp add 指令、transport 選擇、scope、header/OAuth 驗證與安裝驗證。
---

# 安裝遠程 MCP server

## 分工原則
- **過程靜默**：組指令、執行 `claude mcp add`、跑驗證這些你自己做的事，不要逐步播報、不要解釋內部選擇（transport/scope 怎麼選的理由不必講）。安裝過程保持畫面乾淨。
- **只在需要用戶動手時才開口**，且一次講清楚：
  - 要用戶去某網址申請 token / 授權 → 給出網址與逐步操作流程。
  - 只有用戶能跑的指令 → 整理成可直接複製貼上的一行。
- 用戶多半看不懂內部細節，對用戶只說「你要做什麼」，不說「我做了什麼」。

## 決策流程
1. **判斷 transport**：網址型遠程 server 用 `http`（新式）或 `sse`（舊式）。不確定先試 `http`，失敗再 `sse`。
2. **判斷驗證方式**：
   - 靜態 token → 用 `--header "Authorization: Bearer <TOKEN>"`。
   - OAuth → 安裝時不帶 header，裝完在對話輸入 `/mcp` 完成授權。
3. **選 scope**（`-s`，預設 `local`）：
   - `local`：只在當前專案、僅自己可見（預設）。
   - `project`：寫入專案 `.mcp.json`，隨 repo 分享給團隊。
   - `user`：跨所有專案，個人全域可用。
4. **執行安裝**，再**驗證**。

## 指令範本
```bash
# HTTP（最常見）
claude mcp add --transport http <name> <url>

# SSE
claude mcp add --transport sse <name> <url>

# 帶靜態 token
claude mcp add --transport http <name> <url> --header "Authorization: Bearer <TOKEN>"

# 指定 scope（如要團隊共用）
claude mcp add --transport http <name> <url> -s project
```

## 驗證安裝
```bash
claude mcp list          # 看是否列出且狀態正常
claude mcp get <name>    # 看單一 server 細節
```
OAuth server：在對話輸入 `/mcp`，依提示完成瀏覽器授權。

## 常見問題
- **OAuth 報 `does not support dynamic client registration`**：該 server 不支援 cc 的自動 OAuth（GitHub remote MCP 即如此）。改走 token：請用戶產 PAT，再用 `--header "Authorization: Bearer <PAT>"` 重裝。
  - GitHub：URL 用 `https://api.githubcopilot.com/mcp/`，token 在 https://github.com/settings/personal-access-tokens/new 產（依需求勾 Contents / Administration / Issues / Pull requests 等權限）。
- **401 / 驗證失敗**：token 過期或 header 格式錯，確認是 `Authorization: Bearer xxx`。
- **連不上 / connection refused**：transport 選錯，`http` 與 `sse` 互換再試。

> token 屬機密：提醒用戶用完到 GitHub 設定頁 revoke 重產；可用環境變數避免明文寫進 `.claude.json`。

移除：`claude mcp remove <name>`

> 注意：指令旗標以當前 cc 版本 `claude mcp add --help` 為準。
