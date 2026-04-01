# 知識庫檔案組織規則

- Scope: global
- Confidence: [固]
- Trigger: 知識庫組織, 檔案放哪, _AIDocs vs memory, 知識檔案位置, 新增知識, 建立文件, atom 放哪
- Last-used: 2026-04-01
- Confirmations: 1
- Related: doc-index-system, feedback-memory-path

## 知識

### 雙層定位
- [固] `_AIDocs/` = 專案知識庫，存放所有「以知識庫形態存在的細節檔案」（架構分析、邏輯索引、範圍區分、外部文件收錄等）
- [固] `memory/` = 索引層，存放 MEMORY.md 總索引 + 行為回饋 atom（feedback/偏好類）+ 跨專案全域 atom
- [固] `memory/MEMORY.md` 總索引必須包含「導向 _AIDocs/ 的入口」，讓 AI 能透過 _AIDocs/_INDEX.md 進階搜尋

### _AIDocs 子資料夾組織
- [固] 依知識類型建立子資料夾（如 ClaudeCodeInternals/, SourceTree/, Logics/, Zones/, WebDocs/ 等）
- [固] 每個子資料夾有自己的 _INDEX.md
- [固] 頂層 _AIDocs/_INDEX.md 統一索引所有子資料夾

### 判斷基準
- [固] 問自己：「這份內容是某個專案/主題的知識庫參考？」→ _AIDocs/
- [固] 問自己：「這份內容是跨專案的行為校正/偏好/工作流索引？」→ memory/

## 行動

- 新增知識庫類型的詳細內容 → 放 _AIDocs/ 對應子資料夾
- memory/ 下只保留：MEMORY.md、preferences、feedback_* 行為 atom、跨專案全域索引 atom
- MEMORY.md 加入 _AIDocs 導航條目，確保 AI 能自動找到知識庫
