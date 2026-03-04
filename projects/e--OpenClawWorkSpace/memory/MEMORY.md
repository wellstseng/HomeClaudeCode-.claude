# OpenClaw 速查（Atom Index）

> 工作目錄: `E:\OpenClawWorkSpace` | 配置: `E:\OpenClawWorkSpace\.openclaw\openclaw.json` | 版本: 2026.3.4
> GitHub: `holylight1979/OpenClaw-AtomicMemory`（Public）| `holylight1979/OpenClawPanel`（Public, MIT）
> 原子記憶規格: `.openclaw/workspace/skills/atomic-memory/SKILL.md`（核心）+ `SKILL-reference.md`（完整參考）

## 使用者偏好 (holylight)

- 輕量極簡、反對過度綁定、高可讀性
- Windows 10 Pro 電腦、Claude Max + ChatGPT 訂閱
- 自建 _AIDocs 工作流 + 原子記憶系統 (ai-kb-framework 作者)
- 回應語言：繁體中文（技術術語可英文）
- 重視隱私合規：敏感資料分級、禁止長期記憶金融/身分/醫療資料
- [固] 修改後直接記錄（_CHANGELOG + MEMORY + git），不用先問

## 架構摘要

- **LLM**: OpenAI Codex OAuth (gpt-5.3-codex)
- **安全**: group:fs 已解禁、exec.security=full（2026-03-01）、fs.workspaceOnly: true、workspace 已擴大至 `E:\OpenClawWorkSpace`（2026-03-03）
- **平台**: Discord + LINE（ngrok webhook）+ Claude Code Bridge
- **Gateway**: port 18789（WS+webhook）、18791（Browser control, auth=token）；需 `OPENCLAW_CONFIG_PATH` env var；sandbox off
- **[固] controlUi**: `enabled: true, basePath: "/ui"`（2026-03-04 修復）。預設 basePath `/` 的 SPA catch-all 會攔截 plugin webhook routes（LINE 404）+ `/health` 回 HTML。設 `basePath: "/ui"` 隔離。Dashboard URL: `http://127.0.0.1:18789/ui#token=...`。詳見 `memory/gateway-controlui-routing.md`
- **[固] 版本**: 2026.3.2（2026-03-04 從 2026.3.1 升級，`npm i -g openclaw@2026.3.2`）。備份: `E:\OpenClawWorkSpace - 0304`
- **[固] ngrok 指向 18789 + traffic policy 注入 Bearer token**：Gateway 對所有 plugin HTTP route（含 `/line/webhook`）強制 Gateway auth（`shouldEnforceGatewayAuthForPluginPath`），LINE 只帶 `X-Line-Signature` 不帶 Bearer → 需 ngrok 注入。Policy 檔: `.openclaw/ngrok-policy.yml`
- **[固] ngrok 啟動指令**: `ngrok http 18789 --traffic-policy-file=".openclaw/ngrok-policy.yml"`
- **[固] 雙 config 問題**: Gateway 預設讀 `~\.openclaw\openclaw.json`（精簡版），完整版在 `E:\OpenClawWorkSpace\.openclaw\openclaw.json`。啟動時必須設 `OPENCLAW_CONFIG_PATH`，否則 channels 不載入
- **[固] 啟動 Gateway**: 用 `E:\OpenClawWorkSpace\.openclaw\start-gateway.bat`（已包含 env var）
- **[固] auth-profiles.json**: `~\.openclaw\agents\main\agent\auth-profiles.json` 必須存在（OpenAI Codex OAuth token）。Gateway 啟動時載入，新增/修改後需重啟 Gateway
- **ngrok**: WinGet 安裝、LINE webhook `/line/webhook`
- **Bridge**: port 3847、default token、config 讀 `OPENCLAW_CONFIG`
- **OpenClawPanel**: .NET 9 WinForms、控制 Gateway/Bridge/ngrok 啟停、已設定本機路徑（獨立 git repo `holylight1979/OpenClawPanel`）
- **[固] Gateway health check**: `/health` 回 JSON `{"ok":true}`。Panel 用此偵測存活
- **[固] OpenClawPanel v2 功能**: 個別服務 Start/Stop、Active Ports 掃描、per-port Kill、ngrok 含 traffic-policy-file、🌐 Dashboard 按鈕（開 `/ui#token=...`，動態偵測 Gateway port）。重複按鈕已合併（v2 移除舊 Dashboard 按鈕）
- **[固] Panel Gateway 錯誤顯示（2026-03-04）**: `TryStartGateway()` 捕獲 stderr，失敗時 MessageBox 顯示完整錯誤。Health check 15s polling（冷啟動需 7~11s）。StartAll 共用同一方法。Release: `OpenClawPanel\bin\Publish\`
- **[固] Gateway port 18792 = CDP port**: Gateway Internal，每次啟動都會開，非殘留。Browser control (18791) 的 cdpPort 指向它
- **OpenClawDesktop**: .NET 9 Console MCP Server（桌面自動化），取代 3 個舊 MCP（computer-control-mcp/MCPControl/desktop-automation）
- **[固] Discord 策略（2026-03-04 修正）**: groupPolicy="open" + dmPolicy="open" + allowFrom=["*"] + allowBots=true。streaming="off" + blockStreaming=**false**。dm: { policy:"allowlist", allowFrom:["*"], groupEnabled:true }。configWrites=true。**禁止** blockStreaming:true + streaming:"off" 組合（= deadlock）。user-level config 不可有 channels.discord（merge 干擾）
- **Discord 通知頻道**: `1476967208461664378`
- **Session**: per-peer + identityLinks
- **記憶**: 原子記憶 V2.1（向量 RAG + 本地 LLM 分類 + 衝突仲裁 + GC），V1 確定性匹配作為降級後備
- **Preprocessor**: `preprocessor/index.js`（18 模組，含 A20/A21/A22），確定性+本地LLM 演算法，產出 `preprocessor-reports/`
- **memory-retriever V2**: workspace hook，`message:before` 時 Gate(A15)→雙通道(A4+A16)→Fusion(A17)→注入 context → A20 行為觀察
  - Gate: `intent-classifier.js`（確定性快篩 + qwen3:1.7b 五大分類）
  - A4: `trigger-matcher.js`（確定性匹配）| A16: `vector-indexer.js`（LanceDB 向量搜尋）
  - Fusion: `fusion-ranker.js`（0.35×keyword + 0.35×vector + 0.15×recency + 0.15×intent + related - superseded）
  - 降級: Ollama 不可用→自動回退純 A4（V1 行為）
- **本地 LLM**: Ollama + qwen3:1.7b（分類）+ qwen3-embedding:0.6b（嵌入）+ qwen3:0.6b（reranker），~1.8GB VRAM
- **向量索引**: LanceDB (`@lancedb/lancedb` v0.26.2), `vector-store/cc-atoms.lance/`, 15 atoms indexed
- **五大分類**: `taxonomy/categories.json`（人事時地物 + 30 子分類），可自動演化（A18）
- **[固] V2 實作注意**: qwen3:1.7b 是 thinking model，需 `/no_think` tag（system + user 雙重放置）+ `num_predict: 400` + fallback 讀 `message.thinking` 欄位；LanceDB 不接受 null 欄位需用空字串；Node.js v24 會誤判 JSDoc 為 TypeScript 語法；Windows 無 `/dev/stdin` 需用 temp file
- **[固] V2 atom**: `atoms/global/memory-v2-architecture.md`（V2 架構+五大分類+設計哲學）
- **[固] V2 測試腳本**: `scripts/test-e2e-v2.js`(E2E) / `test-degradation.js`(降級) / `test-classifier.js`(分類器) / `run-full-index.js`(全量索引)
- **[固] Ollama 路徑**: `/c/Users/holyl/AppData/Local/Programs/Ollama/ollama.exe`（不在 bash PATH）
- **[固] workspace git repo**: `.openclaw/workspace/` 有獨立 git repo（branch: master），主 repo `.gitignore` 排除 `.openclaw/*`
- **[觀] 2026.3.1 升級注意**: `memorySearch` schema 精簡（移除 mode/maxAtoms/minScore 等）、plugins 需 `openclaw plugins enable` 明確啟用、LINE `groupAllowFrom` 新增（取代嵌套 `groups.*.allowFrom`）、`discord-reader`/`computer-use`/`claude-bridge` plugins 已移除
- **安全**: exec.security=full（2026-03-01）、elevated.enabled=false、fs.workspaceOnly=true、workspace=`E:\OpenClawWorkSpace`（原 `.openclaw\workspace`，2026-03-03 擴大以涵蓋 `平台工作空間/`）
- **SKILL.md**: 拆分為核心 153 行（always-loaded）+ SKILL-reference.md 464 行（on-demand）
- **人員辨識**: `atoms/_identity-map.md`（高速身份映射）→ `atoms/persons/{role}/{alias}/{facet}/`（語意化路徑）
- **事件系統**: `atoms/events/`（多人共享記憶，_active.md 索引）
- **人員權限**: owner（全域）/ user（自身）/ OpenClaw（執行者）
- **atom-context-injector**: workspace hook，agent:bootstrap 時自動注入 atom 到 USER.md（WHO-based）+ 跨 session 行為配對 pending
- **[固] Behavioral Observer（2026-03-04）**: 靜默行為觀察系統，三模組架構
  - A20 `behavioral-observer.js`: 每訊息確定性特徵提取（字數/句長/問句比/指令比/追問比/emoji），rolling avg 到 `session-profiles/{hash}.json`
  - A21 `session-profiler.js`: 每 10 則 Ollama qwen3:0.6b 合成 trait 標籤 + 信心度計算（session 60% + 跨 session 40%）
  - A22 `behavior-matcher.js`: 跨 session snapshot 比對 + 補完 candidate-matcher J3（語言風格 20% + 互動模式 15%）
  - **閾值**: 已知+有歷史 0.80 / 已知無歷史 0.85 / 未識別 0.90
  - **浮出**: 達標才問一次，否認後 +0.15 閾值繼續觀察
  - **9 trait 規則**: concise/verbose/precise/vague/directive/inquisitive/perfectionist/emotional/neutral_tone

## Atom 載入（按需讀取）

> 先讀 Trigger 判斷相關性，相關才完整載入。

| Atom | Trigger |
|------|---------|
| `decisions.md` | 修改 OpenClaw 配置、安裝升級、安全策略討論、平台整合問題、ngrok 啟動、Discord 設定、LINE 設定 |
| `pitfalls.md` | 遇到錯誤訊息、異常行為、啟動失敗、設定不生效、沒有回應、連不上、Discord bot、webhook 失敗 |
| `bridge.md` | 涉及 Bridge 服務、computer-use、Claude Code 整合、LINE→Claude 操作、port 3847、inbox |
| `openclawdesktop.md` | 涉及桌面自動化 MCP、截圖、UI Automation、SendInput、DPI 問題 |
| `desktop-workflow.md` | 需要操作畫面、切換視窗、監測其他 session、GUI 自動化任務 |
| `openclaw-config-intelligence.md` | 修改 openclaw.json、新增 channel/group、設定不生效除錯、理解參數依賴關係 |
| `memory-v2-architecture.md` | 記憶系統、向量搜尋、分類器、V2、memory-retriever、Ollama、LanceDB、五大分類、人事時地物、Gate、Fusion |
| `consciousness-stream.md` | 識流、意識流、透過識流進行、八識、轉識成智、高風險任務、跨系統任務、需要全面分析 |
| `discord-channel-config.md` | Discord 設定、修改 Discord config、Discord guild、Discord DM、Discord streaming、Discord actions、voice、threads |
| `gateway-controlui-routing.md` | controlUi 路由衝突、Dashboard Not Found、405、SPA catch-all、basePath、升級 2026.3.2 |
| `gateway-chat-send-routing.md` | chat.send 行為、LINE 推送、cross-context messaging、session routing、deliverOutboundPayloads |
| `memory-v2-cc.md` | Claude Code 記憶 V2、memory-v2、UserPromptSubmit hook、CC atom 自動檢索、向量搜尋 CC、開發者分類 |

## 雙向通訊（Claude Code ⟷ OpenClaw）

- **[固] Claude Code → OpenClaw**: `/talk-to-openclaw` skill + `gateway-chat.js`
  - Gateway WebSocket `ws://127.0.0.1:18789/ws`
  - 認證: client.id=`"webchat"`, minProtocol=3, Origin header 必要
  - 方法: `connect` (auth) → `chat.send` (sessionKey + idempotencyKey)
  - 預設 session: `agent:main:direct:holylight`（LINE DM session）
  - 串流事件: `event:"agent"` + `stream:"assistant"` + `data.delta`（token）、`event:"chat"` + `state:"final"`（完成）
  - **[觀] chat.send 限制**: `OriginatingChannel` 設為 `"internal"`，回應只回 WebSocket client，不會自動推到 LINE。詳見 `memory/gateway-chat-send-routing.md`
  - 備用: Bridge `POST /message/to-openclaw` → `.openclaw/workspace/claude-messages.jsonl`
- **[固] OpenClaw → Claude Code**: Bridge `POST /message/to-claude` → `.claude/inbox.jsonl`
  - PreToolUse hook 自動讀取（`inbox-check.js`，hookSpecificOutput + additionalContext）
  - OpenClaw 說明: `.openclaw/workspace/skills/notify-claude-code.md`
- **[觀] 桌面控制通訊**: Bridge computer-use 可操作 Chrome 但 VS Code 搶焦點，不可靠
- **[固] LINE 直接推送**: `line-push.js` — 繞過 OpenClaw，直接呼叫 LINE Push API。用於 Claude Code 需要確保訊息到達 LINE 時
- **腳本位置**: `.claude/scripts/` (gateway-chat.js, inbox-check.js, inbox-watcher.js, inbox-write.js, send-to-openclaw.js, line-push.js)
- **[固] 備份目錄**: `E:\OpenClawWorkSpace - 複製`（含完整 .openclaw 設定 + auth-profiles.json，災難復原用）

## [固] Claude Code 記憶 V2.1（Memory V2.1）

- **位置**: `.claude/scripts/memory-v2/`（21 檔案，含 v2.1 新增 8 個）
- **Hook**: `UserPromptSubmit` → `index.js`（每次送出訊息時自動觸發）
- **管線**: Gate（intent-classifier）→ A4 keyword match + A16 vector search → Fusion merge → 注入 additionalContext
- **來源**: CC memory atoms + OpenClaw workspace atoms（雙來源索引）
- **分類**: 開發者五分類 tech/arch/ops/flow/domain（非人事時地物）
- **降級**: Ollama 不可用 → 純 A4 keyword 匹配；全掛 → exit 0（MEMORY.md 照常）
- **共用**: Ollama + qwen3-embedding:0.6b + qwen3:1.7b + LanceDB（與 OpenClaw 共用）
- **[固] v2.1 Sprint 1（2026-03-04）**: 核心治理 — 4 個模組
  - `atom-parser.js`: 新增 11 欄位（memory_id, type, ttl, supersedes, related_to, tags, category, audit_log 等），向下相容
  - `write-gate.js`（新增）: 寫入閘門 — 長度門檻(<20字拒絕) + cosine>0.85 去重合併 + staging(useCount<3)
  - `fusion-ranker.js`: 新公式 0.35×keyword + 0.35×vector + 0.15×recency + 0.15×intent + relatedBonus - supersededPenalty
  - `privacy-delete.js`（新增）: deleteAtom + deleteByUser + 向量清理 + 引用傳播清理 + audit log（memory-audit.jsonl）
- **[固] v2.1 Sprint 2（2026-03-04）**: 衝突與關聯 — 2 個新模組 + 2 修改
  - `conflict-resolver.js`（新增）: 衝突偵測（cosine+keyword overlap 雙門檻）+ 3 層仲裁（newer time→higher confidence→keep both）+ applySupersede 寫入
  - `heartbeat-gc.js`（新增）: 定期治理 — [臨]30d歸檔、[觀]90d列review、[固]180d列audit、superseded 30d歸檔、staging過期。dryRun/exec 模式
  - `fusion-ranker.js`: 從 config.json 讀取 decay 覆蓋值
  - `config.json`: 新增 conflict、decay、gc 配置區塊
- **[固] v2.1 atom schema 新欄位**: Memory-ID, Type(procedural/semantic/episodic/working), TTL, Expires-at, Supersedes, Related-to, Tags, Category, Last-verified-at, Use-count, 演化日誌 section
- **[固] v2.1 recency decay**: 按 type 差異化半衰期 — procedural=365d, semantic=60d, episodic=14d, working=1d（config.decay 可覆蓋）
- **[固] v2.1 intent-aware**: Gate 的 intent 結果影響 fusion 排序（debug→偏好 pitfalls, configure→偏好 decisions）
- **[固] v2.1 衝突仲裁**: 規則引擎（不用 LLM）— Rule1: newer last_verified_at、Rule2: higher confidence、Rule3: keep both + markConflict
- **[固] v2.1 GC 治理**: heartbeat-gc.js — 掃描 atom 目錄 + staging，歸檔到 archive/ 子目錄，支援 dryRun。向量索引同步清理。CLI: `scripts/run-gc.js`（--exec/--report）+ `run-gc.bat`。Windows Scheduled Task `OpenClaw-AtomGC`（每週一 04:00）
- **[固] v2.1 Sprint 3（2026-03-04）**: 品質與觀測 — 3 個新模組 + 2 修改
  - `test-precision.js`（新增）: 20 query-atom 標注測試集 + Precision@3/Recall@3/NDCG@3 評估（offline/full 模式）
  - `vector-indexer.js`: 新增 `filterByPrivacy()` — sensitive atom 只在 DM+userId match 時回傳
  - `audit-viewer.js`（新增）: 審計日誌查詢 API（按 action/user/date/atomId 過濾）+ summary + JSON/文字報告
  - `reranker.js`（新增）: 條件式 Ollama LLM reranker（預設 disabled，qwen3:0.6b，VRAM 可行）
  - `config.json`: 新增 `reranker` 配置區塊
- **[固] v2.1 privacy filter**: public=放行、private=userId match scope、sensitive=isDM+userId match。無 session context 時向下相容全部放行
- **[固] v2.1 reranker**: **已啟用**（`config.reranker.enabled: true`）。qwen3:0.6b + `/no_think` tag，0-10 relevance scoring，weight=0.3 混合 fusionScore。~0.4GB VRAM
- **[固] v2.1 Precision 品質**: Offline(A4)=P@3 15% → Full(A4+Vector+Reranker)=P@3 **45%**, R@3 **85%**, NDCG@3 **82.5%**, 零命中 **0/20**
- **[固] v2.1 測試**: `scripts/test-v2.1.js`（191 tests, Sprint 1+2+3 全通過）+ `scripts/test-precision.js`（品質評估）+ `scripts/test-e2e.js`（原 v2 E2E）
- **[固] v2.1 intent classifier 穩定化**: `/no_think` 放 system + user 雙重、content 空時 fallback 讀 `message.thinking`、`parseClassification` 從 thinking 文字提取 JSON。Empty response: 2/20 → 0/20
- **v2.1 進度**: Sprint 1+2+3 + Tuning Pass + 收尾全部完成（2026-03-04）
- **[固] Windows \r\n**: atom-parser + vector-indexer 已修正 `split(/\r?\n/)`
- **[固] stdin 讀取**: 用 `fs.readFileSync(0, 'utf8')` 避免競態
- **文件**: `.claude/scripts/memory-v2/README.md` + `INSTALL-FOR-OTHER-AI.md`
- **Atom 詳情**: `memory/memory-v2-cc.md`
- **規劃文件**: `_AIDocs/atomic-memory-v2.1-plan.md`（含 Sprint 2/3 路線圖）

## 識流工作流（Consciousness Stream）

- **Skill**: `/consciousness-stream` — 九層管線（觸→五識→六識→七識→八識→光明心→轉智→執行→薰習）
- **觸發**: 使用者說「透過識流進行…」
- **完整研究**: `E:\AI-Develop\consciousness-stream\`（獨立專案）
- **哲學基底**: 唯識學 + 密宗光明心 + 他空見
