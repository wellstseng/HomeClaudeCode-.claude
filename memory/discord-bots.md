# Discord Bot 帳號區分

- Scope: global
- Confidence: [固]
- Trigger: Discord bot, CatClaw bot, Claude Discord Plugin, bot ID, access.json, bot 帳號
- Last-used: 2026-03-25
- Confirmations: 3
- Related: decisions

## 知識

- [固] **Claude Discord Plugin bot**：由 Claude Code 的 Discord plugin 啟動，access.json 設定在 `~/.claude/channels/discord/`
- [固] **CatClaw bot**（ID: `1320597601506299985`）：Wells 自製，catclaw 系統啟動
- [固] 同一個伺服器有兩隻 bot，身份不同，不要混淆
- [固] access.json 的 `groups` key 必須用**頻道 ID**（channel snowflake），不是伺服器 ID（guild snowflake）。用錯了 bot 不會回應也不報錯
- [固] Claude Code session 關閉 = Discord plugin MCP server 斷線 = bot 離線。持久運作必須保持 session 開著

## 已啟用頻道

- DM：allowlist 含 Wells（`480042204346449920`）
- 群組頻道 `1485277764205547630`：requireMention=false

## 行動

- 在 Discord 頻道收到訊息時，確認自己是 Claude Plugin bot，不是 CatClaw bot
