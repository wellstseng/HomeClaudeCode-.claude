# Claude Code 全域設定 — 目錄結構

> 路徑：`C:\Users\holyl\.claude\`

```
~/.claude/
├── CLAUDE.md                    ← 全域工作流引擎指令（always-loaded）
├── settings.json                ← Hooks + 權限設定
├── .mcp.json                    ← MCP server 設定（3 個舊桌面自動化，均 disabled）
├── .gitignore                   ← Git 排除規則
│
├── commands/                    ← 自訂 Skills（/slash commands）
│   ├── consciousness-stream.md  ← /consciousness-stream 識流管線
│   ├── init-project.md          ← /init-project 知識庫初始化
│   └── talk-to-openclaw.md      ← /talk-to-openclaw Gateway WS 通訊
│
├── projects/                    ← 各專案的 auto-memory
│   ├── C--Users-holyl/
│   │   └── memory/              ← 2 atoms
│   ├── e--AI-Develop-consciousness-stream/
│   │   └── memory/              ← 1 atom (MEMORY.md)
│   ├── e--OpenClawWorkSpace/
│   │   └── memory/              ← 11 atoms（主力專案）
│   ├── e--OpenClawWorkSpace-------/
│   │   └── (備份用 workspace)
│   └── f--Mud-GM/
│       └── memory/              ← 1 atom (MEMORY.md)
│
├── _AIDocs/                     ← 知識庫（本目錄）
│
└── [系統目錄 — git 排除]
    ├── cache/                   ← 快取
    ├── debug/                   ← 除錯日誌
    ├── file-history/            ← 檔案歷史
    ├── downloads/               ← 下載
    ├── shell-snapshots/         ← Shell 快照
    ├── telemetry/               ← 遙測
    ├── todos/                   ← Todo 狀態
    ├── backups/                 ← 備份
    ├── plans/                   ← Plan mode 計畫檔
    ├── ide/                     ← IDE 整合
    └── plugins/                 ← 插件快取
```

## 關鍵數據

- **追蹤檔案**: 22 個（CLAUDE.md + settings + commands + memory atoms）
- **排除**: credentials、cache、session transcripts（.jsonl）、系統目錄
- **最大專案**: `e--OpenClawWorkSpace`（11 個 memory atoms）
