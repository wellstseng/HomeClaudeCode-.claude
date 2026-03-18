# 變更記錄

> 保留最近 ~8 筆。舊條目移至 `_CHANGELOG_ARCHIVE.md`。

---

| 日期 | 變更 | 涉及檔案 |
|------|------|---------|
| 2026-03-18 | **`/atom-debug` skill**：新增注入/萃取 debug log 開關。開啟後記錄每次 additionalContext 注入內容、episodic atom 萃取、extract-worker LLM 萃取結果至 `~/.claude/Logs/atom-debug.log`。ERROR log 不受開關控制，含 stack trace | `commands/atom-debug.md`, `hooks/workflow-guardian.py`, `hooks/extract-worker.py`, `workflow/config.json` |
| 2026-03-17 | **V2.12 精確修正計畫**：新增 `/fix-escalation` skill（6 Agent 會議制）+ Guardian hook FixEscalation 信號注入（retry≥2）+ `rules/session-management.md` 精確修正升級段落 + `feedback_fix_escalation.md` atom | `commands/fix-escalation.md`, `hooks/workflow-guardian.py`, `rules/session-management.md`, `memory/feedback_fix_escalation.md`, `memory/MEMORY.md`, `README.md`, `Install-forAI.md` |
| 2026-03-13 | **自檢修復 7 項**：fix silence_accuracy 跨 process 失效（改讀 state）、統一 over_engineering 寫入路徑（消除雙寫競爭）、刪除逐輪萃取死代碼 ~65 行、config per_turn_enabled→false、MEMORY.md failures [觀]→[固]、reflection_metrics 清殘留+重置、toolchain ChromaDB→LanceDB | `hooks/wisdom_engine.py`, `hooks/workflow-guardian.py`, `workflow/config.json`, `memory/MEMORY.md`, `memory/wisdom/reflection_metrics.json`, `memory/toolchain.md` |
| 2026-03-13 | **對外文件更新 Dual-Backend**：README 補充 Dual-Backend 架構+三階段退避+靜態停用旗標；Install 補 rdchat 設定步驟+移除內部 URL+移除過時 extract-worker；Architecture 補 Dual-Backend+Long DIE+ollama_client 工具 | `README.md`, `Install-forAI.md`, `_AIDocs/Architecture.md` |
| 2026-03-13 | **/read-project**: 新增 DocIndex-System.md（76 檔系統索引）+ doc-index-system atom | `_AIDocs/DocIndex-System.md`, `memory/doc-index-system.md`, `memory/MEMORY.md` |
| 2026-03-13 | **選擇性 cherry-pick**：從來源 V2.10 合併 `/continue` skill + `/resume` staging 安全網 + `BOOTSTRAP.md` + `workflow-rules.md` 補回大型計畫/GIT/同步判斷段落 | `commands/continue.md`, `commands/resume.md`, `BOOTSTRAP.md`, `memory/workflow-rules.md` |
| 2026-03-13 | **Atom 整理**：SPEC(950行)/self-iteration/v3-design-spec/v3-research 移至 `memory/_reference/`，MEMORY.md 索引 13→8 筆，新增「參考文件」區塊（開發記憶系統時手動讀取） | `memory/MEMORY.md`, `memory/_reference/*` |
| 2026-03-13 | **README Token/延遲表校正**：CLAUDE.md Token 2500-3500→1500-2500、MEMORY.md Token 50-80→200-350、Prompt 延遲 300-600→200-500ms（V2.11 移除逐輪萃取）、總 Overhead 3000-5000→2000-5500 | `README.md` |
| 2026-03-13 | **README/Install 文件補完 V2.11**：版本號更新、新增「初步建議使用方式」區塊、Skills 表格上移並補 `/harvest` `/upgrade`、架構樹補 `rules/`、Install 補安裝後使用指引 | `README.md`, `Install-forAI.md` |
_(舊條目已移至 `_CHANGELOG_ARCHIVE.md`。最近移入：2026-03-13 V2.11 全面升級)_
