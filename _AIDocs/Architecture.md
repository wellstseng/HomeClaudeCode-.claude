# Claude Code 全域設定 — 核心架構

## Hooks 系統

三個 hook，定義在 `settings.json`：

| Hook | 觸發時機 | 腳本 | 用途 |
|------|---------|------|------|
| `UserPromptSubmit` | 使用者送出訊息前 | `memory-v2/index.js` | 自動記憶檢索（Memory V2） |
| `PreToolUse` | 每次工具呼叫前 | `inbox-check.js` | 讀取 OpenClaw → Claude Code 的 inbox 訊息 |
| `Stop` | Session 結束時 | inline Node.js | 透過 Bridge 發 Discord 通知 |

**注意**: Memory V2 腳本在 `E:\OpenClawWorkSpace\.claude\scripts\memory-v2\`（絕對路徑），但 hook 是全域的，任何專案都會觸發。

## Skills（/Slash Commands）

| Skill | 檔案 | 用途 |
|-------|------|------|
| `/consciousness-stream` | `commands/consciousness-stream.md` | 九層識流管線（佛教八識） |
| `/init-project` | `commands/init-project.md` | 專案知識庫（_AIDocs）初始化 |
| `/talk-to-openclaw` | `commands/talk-to-openclaw.md` | 透過 Gateway WS 與 OpenClaw 對話 |

## Memory 系統

### 分層架構

1. **MEMORY.md**（always-loaded）: 每個專案的 auto-memory 索引 + 高頻事實，200 行上限
2. **Atom 檔案**（按需載入）: 分類知識，由 Trigger 欄位決定何時載入
3. **Memory V2**（自動檢索）: UserPromptSubmit hook 透過向量 + 關鍵詞搜尋，自動注入最相關的 atoms

### Memory V2 管線

```
使用者訊息 → UserPromptSubmit hook
  ├─ Gate: 確定性快篩 + LLM 分類（跳過問候）
  ├─ A4: trigger-matcher（keyword 匹配）
  ├─ A16: vector-indexer（LanceDB 向量搜尋）
  └─ Fusion: 加權合併 → top-3 atoms → additionalContext
```

降級: Ollama 不可用 → 純 A4 | 全掛 → exit 0（MEMORY.md 照常）

### 各專案記憶量

| 專案 | Atoms 數 | 說明 |
|------|---------|------|
| OpenClawWorkSpace | 11 | 主力專案（OpenClaw + Claude Code 整合） |
| C--Users-holyl | 2 | 使用者 home 目錄相關 |
| consciousness-stream | 1 | 識流研究 |
| Mud-GM | 1 | MUD 遊戲 |

## MCP Servers

`.mcp.json` 定義 3 個 MCP server，**全部已停用**（被 OpenClawDesktop 取代）：
- `computer-control-mcp` (Python)
- `MCPControl`
- `desktop-automation`

## 權限設定

`settings.json` 的 `permissions.allow` 列表：
- Bash: powershell, python, ls, wc, git, gh
- Read: C:\Users\**, E:\OpenClawWorkSpace\**
- 特定 OpenClaw 管理指令
