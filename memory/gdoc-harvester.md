---
name: gdoc-harvester
description: Google Docs/Sheets 收割工具經驗 — Playwright + Chrome + cookie 同步 + aiohttp
type: project
---

## Google Docs/Sheets Harvester

**位置**: `~/.claude/tools/gdoc-harvester/`（技能本體，可上 GIT）
**Skill**: `/harvest`（`~/.claude/commands/harvest.md`）
**Runtime**: 使用者指定工作目錄（預設 `c:/tmp/harvester/`，含 browser-data + output，不進 GIT）

### 踩坑記錄

1. **Playwright Chromium 無法登入 Google** — Google 偵測自動化瀏覽器
   - 解法: `channel="chrome"` + `--disable-blink-features=AutomationControlled`

2. **Chrome profile lock 衝突** — 不能同時用同一個 profile
   - 解法: 複製 Chrome Default 的 Cookies 等關鍵檔到獨立目錄（需先關 Chrome）

3. **`context.request.get()` 不帶 browser cookies** — Playwright 設計限制

4. **`page.evaluate` + `fetch()` 被 CORS 擋** — Google export redirect 跨域

5. **export page session 不同步**
   - 最終解法: **aiohttp + browser cookies 同步**（`context.cookies()` → `aiohttp.CookieJar`）

6. **framenavigated race condition** — 同一 doc_id 多次觸發
   - 解法: `on_page_navigate` 在第一個 await 前 `visited.add(doc_id)` 佔位

7. **capture_doc/sheet 自殺 bug** — on_page_navigate 已加 visited，capture 又 `if in visited: return`
   - 解法: 移除 capture 的 early return，只保留冪等 `visited.add()`

8. **標題抓取失敗** — Google export HTML 的 `<title>` 常空或只有 doc_id
   - 解法: 多層 fallback: `<title>` → h1/h2/h3 → 第一段文字 → doc_id

9. **Sheets export 全部 401/400** — aiohttp cookie 同步可能不足
   - 狀態: **未解決**。所有 Sheet 都 401 或 400，Doc 部分也有 401（可能是不同帳號的文件）
   - 可能原因: Sheets API 需要的 auth token 不在一般 cookies 裡，或需要 SAPISIDHASH header

10. **GitLab 登入無法持續** — copytree 完整 profile 後仍無法登入，或關 tab 後被踢
    - 狀態: **未解決**。嘗試了 copytree 整個 Default profile（含 Local Storage、IndexedDB），仍失敗
    - 可能原因: CSRF token 時效、session cookie 綁定 IP/fingerprint、或 Playwright persistent context 的 storage 機制與原生 Chrome 不完全相容

11. **Google Slides 不支援** — 只偵測 document/d/ 和 spreadsheets/d/
    - 狀態: **未實作**。需加 presentation/d/ pattern + export（PDF/PPTX，無 HTML export）

### 架構

- Playwright Chrome (persistent context) → 使用者瀏覽
- `framenavigated` 偵測 Google Docs/Sheets URL
- `aiohttp` + browser cookies → 背景下載 export HTML/CSV
- `markdownify` + `BeautifulSoup` → Markdown 轉換 + 連結提取
- Dashboard (`http://127.0.0.1:8787`) → 即時進度（含摘要預覽）
- 結束後自動產生 `_INDEX.md` 總清單

### 安全設計

- 技能本體零硬編碼路徑、零公司 URL
- browser-data（含所有網站 cookies）存在 runtime 工作目錄，不進 git
- Skill 流程提醒使用者事後清理敏感資料

**Why:** 使用者要把散落在 Google Drive 的公司文件整理收割
**How to apply:** `/harvest` skill 使用或後續改進時參考
