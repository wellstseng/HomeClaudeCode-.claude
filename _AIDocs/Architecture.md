# Claude Code 全域設定 — 核心架構

## Hooks 系統（V2.3）

7 個 hook 事件，定義在 `settings.json`，全部由 `workflow-guardian.py` 處理：

| Hook | 觸發時機 | 腳本 | 用途 |
|------|---------|------|------|
| `SessionStart` | Session 開始 | workflow-guardian.py | 初始化 session state |
| `UserPromptSubmit` | 使用者送出訊息 | workflow-guardian.py | RECALL 記憶檢索 + intent 分類 |
| `PreToolUse` | 工具呼叫前 | inbox-check.js | OpenClaw → CC inbox 訊息 |
| `PostToolUse` | Edit/Write 後 | workflow-guardian.py | 追蹤修改檔案 |
| `PreCompact` | Context 壓縮前 | workflow-guardian.py | 提醒同步 |
| `Stop` | 對話結束前 | workflow-guardian.py | 閘門：未同步則阻止結束 |
| `SessionEnd` | Session 結束 | workflow-guardian.py | Episodic atom 生成 + 清理 |

## Skills（/Slash Commands）

| Skill | 檔案 | 用途 |
|-------|------|------|
| `/consciousness-stream` | `commands/consciousness-stream.md` | 九層識流管線 |
| `/init-project` | `commands/init-project.md` | 專案知識庫（_AIDocs）初始化 |
| `/talk-to-openclaw` | `commands/talk-to-openclaw.md` | 透過 Gateway WS 與 OpenClaw 對話 |

## 記憶系統（原子記憶 V2.3）

### 雙 LLM 架構

| 角色 | 引擎 | 職責 |
|------|------|------|
| 雲端 LLM | Claude Code | 記憶演進決策、分類判斷、晉升/淘汰 |
| 本地 LLM | Ollama qwen3 | embedding、query rewrite、re-ranking、intent 分類 |

### 資料層

1. **MEMORY.md**（always-loaded）: Atom 索引 + 高頻事實
2. **Atom 檔案**（按需載入）: 由 Trigger 欄位 + 向量搜尋發現
3. **Vector DB**: ChromaDB（`memory/_vectordb/`），345 chunks, 18 atoms

### 記憶檢索管線

```
使用者訊息 → UserPromptSubmit hook (workflow-guardian.py)
  ├─ Intent 分類 (Ollama qwen3:1.7b)
  ├─ MEMORY.md Trigger 匹配 (keyword)
  ├─ Vector Search (ChromaDB + qwen3-embedding:0.6b)
  └─ Ranked Merge → top atoms → additionalContext
```

降級: Ollama 不可用 → 純 keyword | Vector Service 掛 → graceful fallback

### 索引來源（4 層）

| Layer | 路徑 | Atoms |
|-------|------|-------|
| global | `~/.claude/memory/` | 2 (preferences, decisions) |
| project:C--Users-holyl | `projects/C--Users-holyl/memory/` | 1 |
| project:e--OpenClawWorkSpace | `projects/e--OpenClawWorkSpace/memory/` | 10 |
| extra:openclaw | `E:/OpenClawWorkSpace/.openclaw/workspace/atoms/` | 5 |

### 工具鏈

| 工具 | 路徑 | 用途 |
|------|------|------|
| rag-engine.py | `tools/rag-engine.py` | CLI: search/index/status/health |
| memory-write-gate.py | `tools/memory-write-gate.py` | 寫入品質閘門 + 去重 |
| memory-audit.py | `tools/memory-audit.py` | 格式驗證、過期、晉升建議 |
| memory-conflict-detector.py | `tools/memory-conflict-detector.py` | 矛盾偵測 |
| memory-vector-service/ | `tools/memory-vector-service/` | HTTP 服務 (port 3849) |

## MCP Servers

| Server | 傳輸 | 用途 |
|--------|------|------|
| workflow-guardian | stdio (Node.js) | session 管理 + Dashboard (port 3848) |
| computer-control-mcp | stdio (Python) | 桌面自動化（停用中） |
| MCPControl | stdio | 桌面自動化（停用中） |
| desktop-automation | stdio | 桌面自動化（停用中） |

## 權限設定

`settings.json` 的 `permissions.allow` 列表：
- Bash: powershell, python, ls, wc, git, gh, ollama, curl
- Read: C:\Users\**, E:\OpenClawWorkSpace\**
- MCP: workflow-guardian (workflow_signal, workflow_status)
- 特定 OpenClaw 管理指令
