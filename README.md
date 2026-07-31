# ai-agent-external-tools

> 三師爸 Sense Bar ・ [youtube.com/@sensebar](https://www.youtube.com/@sensebar)

`ai-agent-external-tools` 是一套協助老師與初學者理解、選擇並安全設定 AI Agent 外部工具的資料。內容涵蓋雲端硬碟、Classroom、筆記、資料庫與常見教學服務。

## 📺 教學影片

👉 **[連接外部工具：MCP 與連接器一張地圖講清楚](https://youtu.be/-kIzGOf0bZA)**

## 📦 這個資料夾有什麼

| 檔案 | 說明 |
|------|------|
| [`AGENT_SETUP_外部工具連接指南.md`](AGENT_SETUP_外部工具連接指南.md) | 給 AI Agent 讀的連接、授權、排錯與驗收流程 |
| [`基本功_外部工具連接方式總表.md`](基本功_外部工具連接方式總表.md) | AntiGravity／Codex 基本功中使用過的外部工具路線與目前入口 |
| [`agents.md`](agents.md) | 本專案的工作邊界、同步規則與外部引用規範 |

## 🤖 一句話讓 Agent 幫你接工具

把 [`AGENT_SETUP_外部工具連接指南.md`](AGENT_SETUP_外部工具連接指南.md) 交給 Claude Code、ChatGPT 桌面版／Codex、Antigravity 2 或 OpenCode，然後說：

> 「讀這份檔案，幫我把 Google Drive（或 Obsidian、Firebase、Padlet⋯）接起來。」

Agent 會依序協助你：確認需求 → 選擇通道 → 說明授權 → 完成設定 → 最小測試 → 記錄與撤銷方式。

## 🗺️ 核心選路方式

先問三個問題：

1. **產品裡有官方 Plugin／Connector 嗎？**
2. **它是否支援你需要的讀取或寫入動作？**
3. **能不能把權限限制在單一專案、唯讀或必要 scope？**

典型路線如下，但不是硬性規則：

| 需求 | 優先路線 | 需要完整自訂時 |
|------|----------|----------------|
| Google Workspace 個資 | 官方 Plugin／Connector＋OAuth | 現成工具缺少必要動作或 scope 時，才評估 GCP＋OAuth／Service Account |
| 其他雲端服務 | 官方 Remote MCP／Plugin＋OAuth | 特殊情境才使用 API Key、PAT 或自建 MCP |
| 本機筆記與檔案 | 限定資料夾的本機權限／可信 MCP | 明確限制可讀寫路徑 |
| 沒有 API 或 MCP 的服務 | Browser／Computer Use | 最後手段；每個高風險操作都要再次確認 |

底層仍可分成兩條軸：

- **通道**：Plugin／Connector、MCP、CLI、直接操控畫面。
- **授權**：本機權限、既有登入、OAuth、API Key／PAT、Service Account。

> 通道決定「怎麼接」，授權決定「以什麼身分、拿到多大權限」。API Key 並不一定全權，實際能力取決於服務限制、scope 與安全規則。

## 👩‍🏫 老師接上以後能做什麼

| 服務 | 立刻有感的用法 |
|------|----------------|
| Google Drive | 讀教材、整理講義、產出簡報 |
| Classroom | 發作業、抓學生名單、整理繳交狀態 |
| Obsidian | 更新第二大腦、整理踩坑筆記 |
| Firebase／Supabase | 幫課堂互動遊戲記分、存測試紀錄 |
| Padlet | 用官方 Public API 管理既有看板資料與貼文；建立課程牆則使用產品介面或經審查的第三方 MCP |

## 🚀 新手起手式

| 你是哪種老師 | 先接這個 | 為什麼 |
|-------------|---------|--------|
| 完全新手 | **Google Drive Plugin／Connector** | 授權流程清楚、教材使用情境多 |
| 想建知識庫 | **Obsidian** | 本機控制、可限制在單一 vault |
| 想快速整理教材 | **Gemini Notebook（原 NotebookLM）** | 能把教材整理成摘要、簡報或音訊；社群 `nlm` 使用未公開內部 API 與瀏覽器 session，僅適合個人實驗 |

> **口訣**：一次只接一個；先完成最小測試，再開寫入與更多權限。

## 🔐 安全重點

- 能限定專案就不要開整個帳號；能只讀就不要先給寫入。
- Token、PAT、OAuth client secret、Service Account private key 不得放進 repo。
- 只有「僅限 Firebase 服務」的 Firebase App client key 可作為公開識別資訊；資料安全要靠 Security Rules 與 App Check。Gemini Developer API key 與 Service Account private key 都是高度敏感秘密。
- 資料庫優先使用開發專案、唯讀模式與去識別化資料。
- 不用的授權要撤銷，並留下撤銷方式與最後驗證日期。

## 🎞️ 既有素材與查證來源

- 各服務路線與教學素材：[`基本功_外部工具連接方式總表.md`](基本功_外部工具連接方式總表.md)
- 具體安裝、排錯與官方來源：[`AGENT_SETUP_外部工具連接指南.md`](AGENT_SETUP_外部工具連接指南.md)

> 文件最後查證：2026-08-01。介面與連接器能力會變動，實際操作前以官方文件和目前工具清單為準。

---

> 歡迎自由下載、分享、改作。製作：三師爸 Sense Bar
