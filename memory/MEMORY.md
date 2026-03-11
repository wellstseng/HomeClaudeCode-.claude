# Atom Index — Global

> Session 啟動時先讀此索引。比對 Trigger → Read 對應 atom。

| Atom | Path | Trigger | Confidence |
|------|------|---------|------------|
| preferences | memory/preferences.md | 偏好, 風格, 習慣, style, preference, 語言, 回應, 執P, 執驗上P, 上GIT | [固] |
| decisions | memory/decisions.md | 全域決策, 工具, 工作流, workflow, guardian, hooks, MCP, 記憶系統 | [固] |
| excel-tools | memory/excel-tools.md | Excel, xls, xlsx, 讀取, 試算表, spreadsheet, openpyxl, xlrd | [固] |
| spec | memory/SPEC_Atomic_Memory_System.md | SPEC, 規格, atom格式, 記憶規範, memory spec | [固] |
| workflow-rules | memory/workflow-rules.md | svn, svn-update, 版本控制, 同步, vcs | [固] |
| hardware | memory/hardware.md | 硬體, 電腦, 升級, hardware, PC, GPU, CPU, 顯卡, 記憶體, RAM, 主機板 | [觀] |
| self-iteration | memory/self-iteration.md | 自我迭代, self-iteration, 演進原則, 理論背書, 設計哲學 | [固] |
| failures | memory/failures.md | 失敗, 錯誤, debug, 踩坑, pitfall, crash, 重試, retry, workaround | [觀] |
| toolchain | memory/toolchain.md | 工具, 環境, 指令, command, path, 路徑, bash, git, python, npm, ollama | [固] |
| v2.9-spec | memory/v3-design-spec.md | V2.9, V3, 設計, 檢索強化, project-alias, ACT-R, multi-hop, blind-spot | [固] |
| v3-research | memory/v3-research-insights.md | 研究, 認知科學, 佛學, 唯識, ACT-R, spreading activation | [觀] |
| unity-yaml | memory/unity/unity-yaml.md | Unity YAML, fileID, GUID, PrefabInstance, .prefab, .meta, 型別ID, 序列化, Missing Script | [固] |

---

## 高頻事實

- 使用者: holyl | Windows 10 Pro | 回應語言: 繁體中文
- [固] 原子記憶 V2.10：回應捕獲 + 跨 Session 鞏固 + 自我迭代 + Wisdom Engine + 檢索強化(V2.9) + Read Tracking + VCS Query Capture + _staging 暫存區(V2.10)
- [固] Vector Service @ localhost:3849 | Dashboard @ localhost:3848
- [固] Ollama models: qwen3-embedding:0.6b + qwen3:1.7b
- [固] 雙 LLM：Claude Code（雲端決策）+ Ollama qwen3（本地語意）
- [固] Vector DB: ChromaDB（非 LanceDB，因 i7-3770 不支援 AVX2）
- [固] Excel: `~/.claude/tools/read-excel.py`（Python3 + openpyxl + xlrd）
- [固] SVN 專案修改前必問 svn update（每 session 一次）| Skill: /svn-update
- [觀] Wisdom Engine: causal graph + reflection + situation classifier
