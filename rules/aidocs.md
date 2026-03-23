# _AIDocs 知識庫

## Session 啟動檢查

**每個 session 第一次互動前**，檢查專案根目錄是否有 `_AIDocs/`：

- **沒有** → 執行 `/init-project` skill
- **已有** → 啟用工作中維護規則

## 工作中維護規則

1. **開工前必讀**：先讀 `_AIDocs/_INDEX.md` 確認是否已有相關文件
2. **_AIDocs 內容分類閘門**：
   - **允許放入**：持久架構知識、函式級索引、設定對照、系統行為文件 — 實作完成後仍有長期參考價值的內容
   - **不允許放入**：規劃文件、進行中方案、階段性調查、TODO/roadmap、session 續接 prompt — 這些放 `memory/_staging/`
   - **判斷基準**：「實作全部完成後，這份文件的內容是否仍對未來開發者有參考價值？」是 → _AIDocs；否 → _staging
3. **修改 src/ 前必讀對應函式索引**：要修改 `src/{dir}/` 下的程式碼前，**必須先讀對應的 `Core-*-Functions.md`**。對照表：
   | src/ 目錄 | 讀取文件 |
   |-----------|---------|
   | acp/ | Core-ACP-Functions.md |
   | agents/ | Core-Agents-Functions.md |
   | auto-reply/, cron/, tts/, context-engine/ | Core-AutoReply-Functions.md |
   | browser/, media-understanding/, media/, link-understanding/, markdown/, canvas-host/ | Core-Media-Functions.md |
   | channels/, line/, whatsapp/, pairing/, sessions/ | Core-Channels-Functions.md |
   | cli/ | Core-CLI-Functions.md |
   | commands/ | Core-Commands-Functions.md |
   | config/ | Core-Config-Functions.md |
   | gateway/ | Core-Gateway-Functions.md |
   | hooks/ | Core-Hooks-Functions.md |
   | infra/ | Core-Infra-Functions.md |
   | plugin-sdk/, memory/, secrets/, security/ | Core-SDK-Functions.md |
   | plugins/ | Core-Plugins-Functions.md |
   | routing/ | Core-Routing-Functions.md |
   | shared/, utils/, logging/, process/, daemon/, terminal/, tui/, wizard/, node-host/, providers/, compat/ | Core-Utilities-Functions.md |
4. **禁止憑記憶修改程式碼**：不確定的架構事實必須查文件或原始碼。概念性問答不受此限。
5. **文件與原始碼同步**：讀取 `Core-*-Functions.md` 或其他 _AIDocs 文件後，若發現與實際原始碼有差異（函式已更名/移除/新增、簽名變更、檔案搬遷等），**立即更新該文件**，不需等到任務結束。差異修正同樣記入 `_CHANGELOG.md`。
6. **活文件更新**：修改核心結構、發現新認知、踩到陷阱 → 更新 `_AIDocs/*.md` + `_CHANGELOG.md`
7. **新增文件時**：同步更新 `_AIDocs/_INDEX.md`
