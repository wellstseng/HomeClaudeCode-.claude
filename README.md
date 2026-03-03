# Claude Code Custom Extensions

> Claude Code hooks + MCP + Atomic Memory = 自動化工作流監督 & 跨 session 知識管理

---

## Overview

這是一套 Claude Code 的自訂擴充系統，核心解決兩個問題：

1. **工作流監督** — Claude 容易忘記同步（git commit、更新文件），這套系統自動追蹤修改、提醒同步、阻止未完成就結束
2. **跨 session 記憶** — Claude 每次新對話都是白紙一張，原子記憶系統讓知識在 sessions 之間延續

**技術架構**：
- **Hooks**（Python）— 6 個生命週期事件的統一處理器
- **MCP Server**（Node.js）— JSON-RPC stdio + HTTP Dashboard
- **Atomic Memory** — 兩層（全域/專案）觸發式知識注入

---

## 7-Phase Workflow Lifecycle

每個 Claude Code session 的完整生命週期：

```
┌──────────────────────────────────────────────────────────────────────┐
│                                                                      │
│  Phase 1: BOOT（啟動初始化）                                          │
│  Trigger: SessionStart hook                                          │
│  ├─ 新 session → 建立 state, 解析全域+專案 MEMORY.md atom index       │
│  └─ resume/compact → 恢復 state, 清空 injected_atoms, 注入摘要       │
│                                                                      │
├──────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Phase 2: RECALL（記憶召回）                                          │
│  Trigger: UserPromptSubmit hook（每輪）                               │
│  ├─ 比對使用者 prompt vs atom Trigger 關鍵詞（case-insensitive）      │
│  ├─ 命中 → 在 token budget 內載入 atom 全文或摘要                     │
│  ├─ 自動更新被載入 atom 的 Last-used 日期                             │
│  └─ 未命中 → 不載入（省 token）                                       │
│                                                                      │
├──────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Phase 3: TRACK（修改追蹤）                                           │
│  Trigger: PostToolUse hook（Edit|Write）                              │
│  ├─ 靜默記錄 file_path + tool + timestamp                            │
│  └─ 設定 sync_pending = true                                         │
│                                                                      │
├──────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Phase 4: REMIND（同步提醒）                                          │
│  Trigger: UserPromptSubmit hook（週期性）                              │
│  ├─ 每 N 輪提醒一次未同步修改（max_reminders 上限）                    │
│  └─ 偵測 sync 關鍵詞時顯示完整 sync context                          │
│                                                                      │
├──────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Phase 5: COMPACT（壓縮保護）                                         │
│  Trigger: PreCompact hook                                             │
│  ├─ 快照 state（timestamp）                                          │
│  └─ Resume 時由 Phase 1 恢復 context + 重新注入 atoms                 │
│                                                                      │
├──────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Phase 6: GATE（結束閘門）                                            │
│  Trigger: Stop hook                                                   │
│  ├─ 修改 ≥ min_files_to_block → BLOCK（最多 N 次）                   │
│  ├─ phase=done/muted → ALLOW                                         │
│  └─ 阻止 N 次後強制放行（anti-loop）                                  │
│                                                                      │
├──────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Phase 7: SYNC（同步執行）                                            │
│  Trigger: 手動（Claude + 使用者確認）                                 │
│  ├─ 更新 _AIDocs/_CHANGELOG.md                                       │
│  ├─ 更新 atom 檔（知識段落 + Last-used）                              │
│  ├─ workflow_signal("sync_completed") → 清空 queue + phase=done       │
│  └─ git commit + push                                                │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
```

---

## Features

### Workflow Guardian
- **自動追蹤修改** — Edit/Write 操作靜默記錄，不干擾工作
- **Stop 閘門** — 有未同步修改時阻止 Claude 結束，防止遺忘
- **Anti-loop 保護** — 最多阻止 N 次後強制放行，不會卡死
- **Mute 靜音** — 不想被打擾時可以靜音提醒
- **Dashboard** — `http://127.0.0.1:3848` 即時監控所有 sessions
- **多實例 Heartbeat** — 多個 VS Code 視窗時自動接管 Dashboard port
- **Session ID Prefix Match** — 用截短 ID（前 8 碼）即可操作

### Atomic Memory
- **兩層架構** — 全域 atoms（跨專案）+ 專案 atoms（專案綁定）
- **觸發式載入** — Prompt 關鍵詞命中才注入，省 token
- **Token Budget** — 根據 prompt 複雜度自動調整載入量（4.5~15KB）
- **三級分類** — `[固]` 確認長期有效、`[觀]` 可能演化、`[臨]` 單次決策
- **Last-used 自動刷新** — Atom 被載入時自動更新使用日期
- **Compact 恢復** — Context 壓縮後自動重新注入 atoms

---

## Quick Start

詳細安裝指南見 [Install-forAI.md](Install-forAI.md)（為 AI 設計的安裝手冊）。

### 核心元件

```
~/.claude/
├── CLAUDE.md                      # 工作流引擎指令（Claude 自動載入）
├── hooks/
│   └── workflow-guardian.py        # 6 事件 hook 處理器
├── tools/
│   └── workflow-guardian-mcp/
│       └── server.js              # MCP server + HTTP Dashboard
├── workflow/
│   ├── config.json                # Guardian 設定（門檻值、關鍵詞）
│   └── state-*.json               # Session state（自動產生，不 commit）
├── memory/
│   ├── MEMORY.md                  # 全域 Atom Index
│   ├── preferences.md             # 使用者偏好 atom
│   ├── decisions.md               # 全域決策 atom
│   └── SPEC_Atomic_Memory_System.md  # 原子記憶規格書
├── commands/
│   └── init-project.md            # /init-project 自訂指令
├── _AIDocs/
│   ├── _INDEX.md                  # 文件索引
│   ├── Architecture.md            # 系統架構詳述
│   └── _CHANGELOG.md              # 變更記錄
└── settings.json                  # hooks 註冊 + 權限
```

### MCP Tools

| Tool | Description |
|------|-------------|
| `workflow_status` | 查詢 session 狀態（省略 session_id → 列出全部） |
| `workflow_signal` | 發送信號：sync_started / sync_completed / reset / mute |
| `memory_queue_add` | 新增知識到 pending queue |
| `memory_queue_flush` | 標記 queue 已寫入 atom |

---

## MCP Transport Format

Claude Code v2.x 使用 **JSONL** 傳輸格式（非 LSP Content-Length header）：

```
{"jsonrpc":"2.0","method":"initialize","params":{...}}\n
{"jsonrpc":"2.0","id":1,"result":{...}}\n
```

- protocolVersion: `2025-11-25`
- 自寫 MCP server 務必遵循此格式，否則 30 秒超時 failed

---

## Configuration

`~/.claude/workflow/config.json`:

```json
{
  "enabled": true,
  "dashboard_port": 3848,
  "stop_gate_max_blocks": 2,
  "min_files_to_block": 2,
  "remind_after_turns": 3,
  "max_reminders": 3,
  "sync_keywords": ["同步", "sync", "commit", "提交", "結束", "收工"]
}
```

---

## License

Personal use. Not published as a package.
