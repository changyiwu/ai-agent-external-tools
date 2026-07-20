# AGENT_SETUP：外部工具連接指南（給所有 AI Agent 讀）

> 出處：三師爸 Sense Bar《AI Agent 基本功 — 連接外部工具》（影片：https://youtu.be/-kIzGOf0bZA）
> 適用對象：任何 AI Agent，特別針對 Claude Code、ChatGPT 桌面版／Codex、Antigravity 2、OpenCode 校準。
> 使用方式：使用者說「讀這份檔案，幫我把 ○○ 接起來」後，依本指南完成訪談、選路、授權、驗收與記錄。
> 最後查證：2026-07-20。產品入口變動很快，實際操作前仍須確認官方文件。

---

## 你的任務

使用者多半是老師或不熟悉程式的初學者。你要：

1. **訪談**：確認要接什麼服務、需要讀取還是寫入。
2. **判路**：先找官方 Plugin／Connector，再考慮 MCP、CLI 或自建整合。
3. **執行**：每個授權動作先說明權限與風險，取得同意後才進行。
4. **驗收**：執行一次最小測試，確認工具真的可用。
5. **記錄**：留下連接方式、授權類型、設定位置、驗證結果與日期。

### 執行原則

- 一次只接一個服務，完成驗收後再處理下一個。
- 優先使用官方或產品內建方案；社群工具要標示來源、維護狀態與風險。
- 優先最小權限：能只讀就不開寫入，能限定專案就不開整個帳號。
- 介面與本指南不同時，以官方文件和目前實際畫面為準。
- 寫入、寄信、刪除、發作業、部署及修改正式資料前，必須再次確認。

---

## 第一步：訪談

依序確認：

1. **服務**：Google Drive、Obsidian、Firebase、Padlet、Supabase 等。
2. **動作**：只讀、建立、修改、刪除或管理權限。
3. **範圍**：單一檔案、單一資料夾、單一專案或整個帳號。
4. **Agent 與介面**：桌面版、網頁版、CLI、IDE 或 OpenCode。
5. **作業系統**：Windows、macOS 或 Linux。
6. **基礎工具**：依實際方案檢查 Node.js、Git、Python／uv 或官方 CLI；不要預設每個 MCP 都需要 Node.js。

---

## 第二步：判路

把連接拆成兩個獨立問題：

- **通道**：Plugin／Connector、MCP、CLI、直接操控畫面。
- **授權**：本機權限、既有登入狀態、OAuth、API Key／PAT、Service Account。

> 通道決定「怎麼接」，授權決定「以什麼身分、拿到多大權限」。API Key 並不一定全權，實際能力取決於服務端權限、scope、限制條件與安全規則。

依序判斷，符合就優先使用：

1. **產品內已有官方 Plugin／Connector 嗎？**
   - 先查看它實際提供的工具與讀寫權限。
   - 若已支援所需動作，直接使用並完成 OAuth 或產品授權。
   - 不要假設「連接器只能讀」；不同 Plugin／Connector 的能力不同。
2. **有官方或可信的 Remote MCP 嗎？**
   - 優先 Streamable HTTP＋OAuth。
   - 限定專案、工具群與唯讀模式（若服務提供）。
3. **有官方 CLI 嗎？**
   - 使用官方登入流程，例如 `gh auth login`、`firebase login`。
4. **需要 Google Workspace 自訂能力嗎？**
   - Gmail、Drive、Calendar、Classroom 個資不能只靠一般 API Key 存取。
   - 現成 Plugin／Connector 缺少必要動作，或需要自訂 scope、網域委派、伺服器自動化時，才評估 GCP＋OAuth／Service Account。
5. **以上都沒有嗎？**
   - 最後才使用 Browser／Computer Use，借用使用者已登入的狀態。
   - 這條路較慢、容易受介面改版影響，也需要更仔細確認高風險操作。

---

## 第三步：各 Agent 的目前入口

### 總覽（2026-07-20）

| 介面 | 優先入口 | 手動 MCP | 驗證方式 |
|------|----------|----------|----------|
| **Claude Code** | Anthropic Directory／claude.ai Connectors | `claude mcp add`、`.mcp.json`、`~/.claude.json` | `claude mcp list`、會話內 `/mcp` |
| **ChatGPT 桌面版／Codex** | Plugins；`Settings → MCP servers` | `codex mcp add`、`~/.codex/config.toml`、專案 `.codex/config.toml` | `/mcp`、`codex mcp list` |
| **ChatGPT 網頁版** | Work mode 的 **Plugins** | 使用 Plugin 內含的 Connector／Remote MCP；不讀本機 Codex 設定 | Plugins 頁與新對話中的可用工具 |
| **Antigravity 2** | `Settings → Customizations → Installed MCP Servers` | 全域 `~/.gemini/config/mcp_config.json`；專案 `.agents/mcp_config.json` | 已安裝清單、Refresh、CLI `/mcp` |
| **OpenCode** | 無內建目錄，直接設定 | 專案 `opencode.json`；全域 `~/.config/opencode/opencode.json` | `opencode mcp list`、實際呼叫工具 |

> 加入或修改工具後，依產品提示執行 Reload、Restart、Restart extension 或開新對話。不要把「所有 Agent 都一定要完全重啟」當成通則。

### Claude Code

遠端 HTTP 是雲端服務的優先方式：

```bash
claude mcp add --transport http <名稱> <MCP網址>
```

本地 stdio server：

```bash
claude mcp add --transport stdio <名稱> -- npx -y <MCP套件名>
```

Windows 若出現 `npx` 無法啟動或 `spawn` 類錯誤，再改用相容性寫法：

```powershell
claude mcp add --transport stdio <名稱> -- cmd /c npx -y <MCP套件名>
```

- 預設 local scope 只供目前專案與目前使用者使用。
- 要把設定分享進版本控制，使用 `--scope project`，會寫入 `.mcp.json`。
- 要跨專案個人使用，使用 `--scope user`。
- OAuth 可在 `/mcp` 或支援版本的 `claude mcp login <名稱>` 完成。
- claude.ai Connectors 只有在 Claude Code 使用 claude.ai 訂閱登入時才會載入。
- Plugin 提供的 MCP 若需重新載入，可使用 `/reload-plugins`；部分 MCP 支援動態工具更新。

### ChatGPT 桌面版／Codex／網頁版

#### ChatGPT 桌面版與 Codex

1. 開啟 `Settings → MCP servers`。
2. 選擇 **Add server**。
3. 選擇 STDIO 或 Streamable HTTP，填入命令或 URL。
4. 儲存後依畫面選擇 **Restart**；OAuth server 再按 **Authenticate**。

CLI 也可設定：

```bash
codex mcp add <名稱> -- npx -y <MCP套件名>
codex mcp list
codex mcp login <名稱>
```

或編輯設定：

```toml
[mcp_servers.<名稱>]
command = "npx"
args = ["-y", "<MCP套件名>"]
```

- ChatGPT 桌面版、Codex CLI 與 IDE extension 在同一個 Codex host 上共用 MCP 設定。
- 專案專用設定可放在受信任專案的 `.codex/config.toml`。

#### ChatGPT 網頁版

- 切換到 **Work mode**，從 **Plugins** 安裝需要的 Plugin。
- Plugin 可包含 Skills、Connectors 與 Remote MCP 工具。
- 網頁版不讀本機 `~/.codex/config.toml`，也沒有本機 Codex 的 `/mcp` 指令選單。
- 安裝 Plugin 後通常要開新對話，才會載入新工具。

### Antigravity 2

- 到 `Settings → Customizations → Installed MCP Servers`，選 **Add MCP** 進入 MCP Store。
- 全域設定：`~/.gemini/config/mcp_config.json`。
- 專案設定：`.agents/mcp_config.json`。
- Remote MCP 使用 `serverUrl`；舊的 `url`、`httpUrl` 不適用於這份設定格式。

```json
{
  "mcpServers": {
    "本地例子": {
      "command": "npx",
      "args": ["-y", "<MCP套件名>"]
    },
    "遠端例子": {
      "serverUrl": "https://<MCP網址>",
      "headers": {
        "Authorization": "Bearer <token>"
      }
    }
  }
}
```

- JSON 不能寫註解或多餘逗號。
- 優先使用 OAuth／Store 的安全欄位；若必須填 token，確認設定檔不會被分享或提交。

### OpenCode

```json
{
  "$schema": "https://opencode.ai/config.json",
  "mcp": {
    "本地例子": {
      "type": "local",
      "command": ["npx", "-y", "<MCP套件名>"],
      "enabled": true
    },
    "遠端例子": {
      "type": "remote",
      "url": "https://<MCP網址>",
      "enabled": true
    }
  }
}
```

- Remote MCP 支援 OAuth；可用 `opencode mcp auth <名稱>` 登入。
- 用 `opencode mcp list` 檢查狀態、`opencode mcp debug <名稱>` 排錯。
- 每個 MCP 的工具說明都會增加上下文，只啟用目前 Agent 需要的工具。

---

## 常見服務快查表

| 服務 | 優先通道 | 授權與注意事項 |
|------|----------|----------------|
| **Google Drive／Gmail／Calendar** | 官方 Plugin／Connector | 通常走 OAuth；先看實際工具是否支援所需讀寫動作。一般 API Key 不能存取使用者個資 |
| **Google Classroom** | 官方／可信整合；必要時自建 OAuth | 寫入與學生資料權限要特別確認；學校 Workspace 管理政策可能限制授權 |
| **Obsidian** | 本機檔案權限或可信 MCP | 不需雲端 token；把可讀寫範圍限制在指定 vault |
| **Gemini Notebook（原 NotebookLM）** | 產品介面；必要時使用社群 `nlm` | `nlm` 不是 Google 官方 CLI，使用瀏覽器登入與既有 session，產品改版時可能失效 |
| **GitHub** | Plugin／Connector 或 `gh` CLI | OAuth；CLI 使用 `gh auth login`，確認 repo 與寫入範圍 |
| **Padlet** | 可信 MCP／API | API token；不要寫入 repo，先在測試看板驗證 |
| **Firebase CLI** | 官方 CLI | `firebase login` 透過 Google 帳號授權；不是服務帳號金鑰 |
| **Firebase Web App** | 官方 SDK／設定物件 | Web config 與 Firebase API key 是公開識別資訊；資料安全依靠 Security Rules 與 App Check |
| **Firebase Admin／伺服器** | Admin SDK／ADC／Service Account | Service Account JSON private key 是高度敏感憑證，只能放在受控環境 |
| **Supabase** | 官方 Hosted MCP | 預設使用 OAuth，不再要求 PAT；優先指定 `project_ref`、`read_only=true`，不要連正式資料 |
| **Notion 等 SaaS** | 官方 Remote MCP／OAuth | 優先官方整合；只有不支援 OAuth 或 CI 等特殊情境才考慮長效 token |

---

## 常見卡關排除

| 症狀 | 先檢查什麼 |
|------|------------|
| 加入 MCP 後看不到工具 | 先看 server 狀態與錯誤；依產品執行 Refresh、Reload、Restart、Restart extension 或開新對話 |
| Claude Code 在 Windows 無法啟動 npx server | 確認 Node.js／PATH；若為 spawn 問題，再改用 `cmd /c npx ...` |
| 出現 `npx`／`npm` 找不到 | 安裝 Node.js LTS，重開終端機後再次驗證 PATH |
| JSON 設定沒有作用 | 檢查註解、多餘逗號、欄位名稱與 Windows 路徑跳脫 |
| ChatGPT 網頁找不到本機 MCP | 網頁版不讀本機 Codex 設定；改在 Work mode 安裝 Plugin，或改用桌面版／Codex host |
| 遠端 MCP 回傳 401／403 | 完成或重跑 OAuth；檢查 token、scope、帳號與 workspace 管理政策 |
| `nlm` 登入失效 | 重新登入並確認社群工具版本；若產品介面已改版，回到 Gemini Notebook 官方介面 |
| Agent 工具太多、答非所問 | 停用不需要的 MCP／Plugin，或依專案、Agent、工具群縮小範圍 |

---

## 安全守則

1. **最小權限**：只開需要的 scope、專案與工具；能只讀就不給寫入。
2. **分清憑證類型**：API Key／PAT 通常是長效憑證，但權限並非一律全開；OAuth 可撤銷且能限制 scope；Service Account 是機器身分。
3. **秘密不進 repo**：token、PAT、Service Account private key、OAuth client secret 放在環境變數、系統憑證庫或受控設定檔。
4. **Firebase 例外要講清楚**：Firebase Web API key 可公開，但仍要限制適用 API，並正確設定 Security Rules／App Check；Service Account private key 絕不可公開。
5. **學生資料去識別化**：只使用必要欄位；測試資料不要放真實姓名、信箱或成績。
6. **不明 MCP 不安裝**：優先官方或可審查、持續維護的來源；外部內容仍可能造成 prompt injection。
7. **資料庫不直接接正式環境**：優先開發專案、唯讀模式、專案限定與測試分支。
8. **高風險操作再確認**：寄信、發作業、刪除、部署、修改權限與正式資料前再次取得同意。

---

## 驗收與記錄

1. 先測一個最小讀取，例如列出一個資料夾或查詢一張測試表。
2. 寫入功能先使用測試資料夾、測試看板或開發資料庫。
3. 確認 Agent 實際看得到預期工具，不能只看「已安裝」。
4. 在 `AGENTS.md`、專案筆記或當次研習紀錄寫下：服務、通道、授權、scope、設定位置、驗證方式、日期與撤銷方式。

---

## 官方與主要來源

- OpenAI MCP：https://learn.chatgpt.com/docs/extend/mcp
- OpenAI Plugins：https://learn.chatgpt.com/docs/plugins
- Claude Code MCP：https://code.claude.com/docs/en/mcp
- Google Antigravity MCP：https://antigravity.google/docs/mcp
- OpenCode MCP：https://opencode.ai/docs/mcp-servers/
- Firebase API keys：https://firebase.google.com/docs/projects/api-keys
- Firebase Admin SDK：https://firebase.google.com/docs/admin/setup
- Firebase CLI：https://firebase.google.com/docs/cli
- Supabase MCP：https://supabase.com/docs/guides/ai-tools/mcp
- Gemini Notebook 更名公告：https://blog.google/innovation-and-ai/products/gemini-notebook/notebooklm-gemini-notebook/
- 社群 `nlm` 專案：https://github.com/jacob-bd/notebooklm-mcp-cli

> 版本：v3（2026-07-20）。已更新 ChatGPT／Codex MCP 與 Plugins 入口、Supabase OAuth、Firebase 憑證分類、Gemini Notebook 名稱、各產品重新載入方式及官方來源。
