# AlphaForge V5 — 專案交接文件

**目的**：給 Claude Cowork（或下一位 AI 工程師）接手繼續優化用。

---

## 📁 檔案位置 & 部署

| 用途 | 路徑 |
|------|------|
| **本機開發檔（主檔）** | `C:\Users\Kenhsieh\Downloads\alphaforge-v5.html` |
| **部署 repo（Git 工作區）** | `C:\Users\Kenhsieh\Downloads\alphaforge-site\alphaforge-v5.html` |
| **GitHub repo** | `github.com/kenhsieh1211/alphaforge` |
| **GitHub Pages 公開 URL** | https://kenhsieh1211.github.io/alphaforge/alphaforge-v5.html |
| **Memory（Claude）** | `C:\Users\Kenhsieh\.claude\projects\C--Users-Kenhsieh\memory\project_compass.md` |

**部署流程**：
```bash
cp "C:/Users/Kenhsieh/Downloads/alphaforge-v5.html" "C:/Users/Kenhsieh/Downloads/alphaforge-site/alphaforge-v5.html"
cd "C:/Users/Kenhsieh/Downloads/alphaforge-site"
git add alphaforge-v5.html
git commit -m "..."
git push
```
GitHub Pages 約 30-60 秒生效。`alphaforge-site/.git/config` 內嵌 PAT（明文，user 已知此安全風險）。

---

## 🏗 技術 Stack

- **單一 HTML 檔**：~6500 行 / ~400KB
- **純 Vanilla JS**：無框架、無 build step
- **純 CSS 動畫**：無 SVG 動畫庫
- **無後端**：所有狀態存 localStorage
- **無編譯**：直接編輯 .html 即可

---

## 🌐 外部 API（全部已串接）

### 1. Anthropic Claude API
- **Key**：使用者各自設定（存 `localStorage.af_api_key`）
- **預設模型**：`claude-sonnet-4-5`（可切 `claude-haiku-4-5-20251001`）
- **用途**：AI 同仁子任務研究、CEO 統一匯報、CEO 決策審議、辦公室聊天回應、CEO 結語
- **直接呼叫**：`https://api.anthropic.com/v1/messages` + header `anthropic-dangerous-direct-browser-access:true`

### 2. Binance Public API（**無需 Key**）
- `GET /api/v3/ticker/24hr?symbols=[...]` — Top 10 即時行情
- `GET /api/v3/klines?symbol=XXX&interval=1d|4h&limit=N` — K 線
- 6 秒一次拉行情，CORS 允許瀏覽器直接打

### 3. Telegram Bot API（內建預設值）
- **Token**：`8629837615:AAF3qRgM8mrgVmN1RGoijSraEsGVOfUVvSA`
- **Chat ID**：`6256990697`
- **Email**：`kenhsieh1211@gmail.com`
- **用途**：
  - 推送 CEO 任務匯報、午夜日報、自動進場通知
  - 雲端同步（用 `sendDocument` + `setMyDescription` 當儲存指針）

> ⚠️ Token + Chat ID 寫死在 v5.html 程式碼中（變數 `DEFAULT_TG_TOKEN`、`DEFAULT_TG_CHATID`），public repo 上其他人看得到。可接受的安全風險（最多被惡意傳訊到亂猜的 chat id）。

---

## 💾 localStorage 鍵列表

| Key | 內容 |
|-----|------|
| `af_api_key` | Anthropic API key |
| `af_model` | 選定模型 |
| `af_tg_token` / `af_tg_chatid` | Telegram 設定（覆寫預設） |
| `af_positions` | 持倉陣列 |
| `af_meeting_tasks` | 會議任務（含 corrections, ceoDecision, assignees with status/report） |
| `af_sim_state` | `{initial, cash, realizedPnl, closedTrades[], navHistory[]}` |
| `af_office_chat` | 辦公室聊天歷史 |
| `af_learned_routes` | 自我升級的路由規則 |
| `af_strategies` | 策略庫（含 dsReview） |
| `af_watchlist` | 觀察列表 |
| `af_daily_reports` | 歷史日報（≤90 份） |
| `af_meeting_no` | 累計會議編號 |
| `af_auto_execute` | 任務自動執行（預設 true） |
| `af_auto_trade` | CEO 決策自動進場（預設 true） |
| `af_auto_sync` | 30 秒自動推雲端 |
| `af_last_daily_report` | 上次日報日期（YYYY-MM-DD） |
| `af_last_strategy_dev` | 上次策略開發日期 |
| `af_lab_last_decision` | 策略研究室最後一次決策 |
| `af_last_sync_push` | 上次推雲端時間戳 |

---

## 🎭 8 位 AI Agent + Skills

每位有完整 5 段 system prompt：【人格】+【思考框架/Skill】+【輸出格式】+【協作關係】+【禁忌】

| Agent | Skill 框架 | 角色 |
|-------|-----------|------|
| **CEO** | Synthesis & Decision | 總指揮官，最後整合決策 |
| **MA** | Three-Scenario Macro | 總經分析（Fed、CPI、三情境機率） |
| **QR** | Hypothesis → Backtest → Sharpe | 量化研究員（因子、IC、Sharpe） |
| **TC** | Pattern → Volume → Entry/Exit | 技術線型 + 籌碼 |
| **CX** | Cross-Domain Hypothesis → Statistical Test | 混沌信號獵人（瘋狂跨域假說） |
| **DS** | Validation & Statistical Rigor | 統計嚴謹性（p 值、過擬合、placebo） |
| **RM** | Kelly → VaR → Exposure Cap | 風險管理（單標 ≤5%、單市場 ≤30%） |
| **EX** | Order Routing & Execution Plan | 執行交易員（拆單、滑點、流動性） |

每位有 **~160 條獨特對話台詞**（在 `agentBubbles` 物件），總計約 1,280 條。

---

## 🧩 主要模組（程式碼區塊在原始檔的相對位置）

### Section 1: HEAD + CSS（行 1-450）
- 完整視覺主題：terracotta + gold + cream + brown 復古配色
- Ghibli 風格 CSS 角色（會議室）：`.gh-char` + 8 種 `.gh-XX` 變體
- RWD 4 段 breakpoints：1100 / 900 / 600 / 380px

### Section 2: HTML 結構（行 450-1300）
- Header（含 🛎 開會 / ⚙ API KEY 按鈕）
- 4 個主要 Page（office / positions / dashboard / lab）
- 多個 Modals（API key / Telegram / 編輯任務 / 任務詳情 / 學習記憶 / 日報歷史 / 會議確認 / 重啟確認）
- 8 位吉普力角色（HTML 結構 + class）

### Section 3: JavaScript 核心（行 1500-6500）

**3.1 Agent Systems**（行 ~4945-5200）
- `agentSystems` 物件：8 位 AI 的完整 system prompts
- `agentBubbles`：1280+ 條台詞池
- `agentMeta`：每位 agent 的顏色、全名

**3.2 Binance API**（行 ~3700-4100）
- `BINANCE_TOP10`、`fetchBinanceTop10()`、`fetchKlines()`
- `buildMarketContext(taskText)` — 抽出標的 → 拉 K 線 → 組成完整 context 餵給 AI
- `extractSymbols()`、`calcKlineStats()`

**3.3 模擬投資 + Sim State**（行 ~4470-4650）
- `simState` 結構：`{initial, cash, realizedPnl, closedTrades[], navHistory[]}`
- `calcSimSnapshot()` — 即時計算 NAV 含未實現
- `simOpenPosition()`、`simClosePosition()`
- `takeNavSnapshot()` — 每分鐘檢查跨日，記錄歷史
- `updateSimDashboard()` + `updateFullDashboard()` — 5 秒一次刷新所有 Dashboard 數字 / 曲線 / 三市場 / 交易紀錄
- `restartSimulationWithMission()` — 徹底重啟 + 派 CEO 中長期加密貨幣任務

**3.4 會議室 + 任務系統**（行 ~3900-4400）
- `confirmMeeting()` / `enterMeeting()` / `closeMeeting()`
- `selectMeetingChar(name)` — 點角色顯氣泡 + 簡報
- `genBriefing(agent)` — 動態生成各角色簡報（含 Binance 即時數據）
- `assignTask()` — 新建任務，CEO 拆子任務
- `ceoBreakdownSubtask(who, text)` — 各角色客製化子任務描述
- `simulateAgentDone(taskId, who)` — 單一 agent 執行子任務（API 呼叫含 marketCtx）
- `autoExecuteTask(taskId)` — 依 `AGENT_EXEC_ORDER` 順序執行
- `generateCEOSummary(taskId)` — CEO 統合
- `editTask` / `saveTaskEdit` / `expandTask` / `renderTaskDetail` / `forceResetTask`

**3.5 自我升級 / 學習路由**（行 ~3950-4150）
- `pushbackPatterns` — regex 偵測 AI 推回語句
- `detectPushback(report)` — 抽出建議的 agent 名稱（含 RA→QR、PM→CEO 對應）
- `autoCorrectIfNeeded()`、`manualCorrect()` — 自動 / 手動校正
- `learnedRoutes`（localStorage）— 永久記憶
- `extractTaskPattern()` — 從任務抽 pattern key
- `suggestAssignees()` — 含 isResearch / isExecution 細分判斷

**3.6 CEO 決策審議**（行 ~4180-4300）
- `ceoDeliberateDecision(summary, taskText)` — 結構化 JSON 輸出
- `calcPortfolioStatus()` — 把投組狀態餵給 CEO
- `approveCeoTrade()` / `rejectCeoTrade()` / `triggerCeoDeliberate()`
- `autoExecuteTrades(taskId)` — **自動進場**（檢查現金 + 單標 ≤10%）

**3.7 Telegram 通知 + 雲端同步**（行 ~4350-4500）
- `sendTelegram(text, opts)` — Markdown 自動降級 + 4096 字切片
- `buildTelegramBody(t)` — 任務匯報格式化
- `sendTaskEmail` (function 名稱保留 email 但實際走 TG)
- **雲端同步**：`pushStateToCloud()` / `pullStateFromCloud()`
  - 把 state 上傳為 JSON document
  - file_id 寫到 `setMyDescription`（全域可讀）
  - 任何裝置 token 對 → 同樣讀得到

**3.8 CEO 午夜日報**（行 ~4600-4800）
- `scheduleMidnightReport()` — 計算到 00:00 倒數
- `generateDailyReportContent()` — 含 Binance 行情 / 持倉 / PnL / 任務狀態 / CEO 結語
- `getCEODailyNote()` — 用 Claude API 生成個人化結語
- `viewDailyReports` / `exportDailyReports`
- 啟動 catch-up：00:00-08:00 之間若今日尚未發送 → 立刻補發

**3.9 策略開發流程**（行 ~4900-5050）
- `qrIdeaTemplates` — 15 個範本（QR/TC/CX/MA/EX 各自開發）
- `runDailyStrategyDevelopment()` — 每天 09:00 自動觸發 QR 提案 → DS 評鑑
- `dsReviewTemplate()` — 依 Sharpe + 樣本數給不同評語

**3.10 時間軸**（行 ~5100-5250）
- `buildTimelineEvents()` — 彙整所有來源（任務、子任務、決策、交易、日報、策略、校正）
- `renderTimeline()` — 按日期分組 + 7 種 filter

**3.11 即時聊天**（行 ~5350-5500）
- `chatHistory` 持久化
- `sendChatMessage()` — 智慧路由 1-3 個 agent 回應
- `pickResponders()` — 關鍵字 + @ 指定
- 70% 機率走 API、30% 範本

---

## 🎬 邏輯執行順序

`AGENT_EXEC_ORDER = ['MA','QR','TC','CX','DS','RM','EX','CEO']`

**派任務 → 自動執行的完整流程**：
1. CEO 智慧分配子任務（依關鍵字 + 學習記憶）
2. 排序成上述邏輯順序
3. 依序執行（每位 1.2s 間隔）
4. 偵測 pushback → 自動校正 → 改派 → 重跑
5. CEO 統合（generateCEOSummary）
6. CEO 深思熟慮（ceoDeliberateDecision → 結構化 JSON）
7. 若 GO / PARTIAL → autoExecuteTrades 自動進場
8. 推 Telegram

---

## 🐛 已知問題 / 待優化方向

### Already Working
- ✅ 全部 API 串通（Claude / Binance / Telegram）
- ✅ 自我升級 + 學習記憶
- ✅ 跨裝置同步
- ✅ 自動進場 + 部位管理
- ✅ 手機版 RWD

### 可繼續優化
1. **股票數據**：TSMC / SPY 仍是 Math.random walk simulation。免費 API 選項：Yahoo Finance 非官方 API、Alpha Vantage（需 key, 25/day）
2. **Sharpe 需 ≥10 天 navHistory** 才能算（現在會顯示「需 ≥10 日資料」）
3. **AI 簡報的 markdown 沒解析** — 顯示原始 markdown 文字（用 `pre-wrap`）。可加 `marked.js` 或自製 markdown→HTML 簡單轉換
4. **任務數量上限**：目前無刪舊邏輯，meetingTasks 會持續成長。可加 7 天前 completed 任務自動歸檔
5. **平倉觸發條件**：目前只支援手動平倉（`closePos(id)`）。可加自動停損（價格觸發 sl%）+ 自動停利
6. **多空策略**：目前 CEO 決策 JSON 有 direction 多/空，但所有 prompts 偏向多單 mindset
7. **PWA**：可加 `manifest.json` + service worker 支援離線

### Edge Cases to Verify
- Telegram 訊息 Markdown 偶爾會 escape 失敗（已加 fallback）
- API key 錯誤時，部分流程可能卡住（autoExecutingTaskId 鎖）
- iPhone Safari 上 sendDocument 可能因 FormData 處理略有差異

---

## 🔧 偵錯指引

### 任務卡住
- 看任務卡片標題列，**>5 分鐘 in-progress** 會出現紅色 🔄 強制重置按鈕
- F12 Console 看 `[manualCorrect]`、`[simulateAgentDone] API failed`

### 價格不對 / 卡住
- F12 看 `binanceData` 物件，每 6 秒應該更新
- 側邊欄「Binance Top 10」狀態指示：🟢 LIVE / 🔴 連線失敗

### Telegram 沒收到
- 開 📨 TG 設定 → 🧪 測試傳訊（先呼叫 getMe 驗 token，再 sendMessage）
- 常見錯誤：`chat not found`（需先傳 hi 給 bot）

### 雲端同步不同步
- 兩台裝置必須用**同一組 Token**（不同 token = 不同雲端）
- 推送一次後 bot description 才會有 file_id
- 拉取需要 description 存在

---

## 📋 重要按鈕 / 快捷

| 按鈕 | 位置 | 動作 |
|------|------|------|
| 🛎 開會 | Header | 進入會議室 |
| ⚙ API KEY | Header | 設定 Anthropic key |
| 🚀 重啟模擬 | 辦公室工具列 | 徹底重置 + 派 CEO 中長期加密任務 |
| ☁ 推送 | 辦公室工具列 | 推送狀態到雲端 |
| 📥 拉取 | 辦公室工具列 | 從雲端覆蓋本機 |
| 📨 日報 | 辦公室工具列 | 立即觸發 CEO 日報 |
| 📨 TG | 會議室任務區 | Telegram 設定（含自動交易/自動同步開關） |
| 🧠 學習 | 會議室任務區 | 查看學習記憶 |
| 🔍 | 任務卡片 | 全屏放大檢視 |
| ✏️ | 任務卡片 | 編輯任務 |
| 🔄 | 任務卡片（卡住時才出現） | 強制重置 |

---

## 💡 給接手者的建議

1. **不要重寫**，這份單檔已經很完整。優化方向應該是**補強與擴充**。
2. **優先做的 3 件事**：
   - 加自動停損 / 停利（plumbing 已準備好，sl 欄位都在 positions）
   - 加股市真實 API（TSMC / SPY）— 用 Yahoo Finance 非官方 endpoint 即可
   - 加 markdown 渲染 — 簡報內容會更好看
3. **慢的點**：simulateAgentDone 一次抓 K 線 + 呼叫 Claude API，約 5-15 秒/位 agent。7 位同仁 = 1-2 分鐘。可考慮並行而非串行。
4. **歷史可保留**：localStorage `af_*` 都跨會議延續。重啟模擬只清交易相關，不清聊天 / 學習記憶。

---

## 🗂 commit 歷史精華

```
b8dd9db  Add restart simulation + CEO mid-long-term crypto investment mission
55e4433  Auto-execute trades + cross-device cloud sync via Telegram
d01ea8c  Dashboard now fully dynamic
b4e9c9e  Reset: actually clears positions + tasks + dashboard trade rows
59c7b00  Reset truly clears all hardcoded numbers in 倉位管理 + sidebar pipeline
06f3d4e  Auto-execute on edit + logical agent order
06f3d4e  Self-correcting CEO routing (pushback detection + learning)
ec984c2  Deploy AlphaForge V5 (initial)
```

---

## 📞 持續對話的 Anchor

如果在 Cowork 開新會話接續，可以直接說：

> 「我有一個 AlphaForge V5 投顧模擬系統的單檔 HTML，部署在 https://kenhsieh1211.github.io/alphaforge/alphaforge-v5.html。完整 handover 在 `C:\Users\Kenhsieh\Downloads\alphaforge-v5-HANDOVER.md`，主檔在 `C:\Users\Kenhsieh\Downloads\alphaforge-v5.html`。請讀完這兩個檔案後接手繼續優化。」

或者直接複製本檔內容貼到 Cowork 對話框。

---

_Generated 2026-04-28 by Claude · AlphaForge V5 final state_
