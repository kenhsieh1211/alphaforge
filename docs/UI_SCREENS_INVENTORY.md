# 趙雲菠蘿傳 v0.6 — UI 畫面索引

> 主檔：`D:\.Claude\.Pineapple\zhaoyun_pineapple_v06.html`
> 截圖時間：2026-05-03（透過 localhost HTTP server + Chrome MCP 自動抓取）
> 截圖檔：保留在本對話中（Chrome MCP 截圖 ID 列於下表）；如需離線存檔請從對話中右鍵另存

## 畫面狀態總覽

遊戲核心是一個 phase machine：`game.phase ∈ {intro, menu, char_transition, transition, placing, battle, reward, gameover, victory}`，加上三個子模式 `_skillMode / inFantasyland / showLog` 疊加在 `placing` 之上。

| # | 畫面名 | game.phase / 子模式 | 觸發條件 | 主要 draw 函數 | 行數 | 截圖 ID |
|---|---|---|---|---|---|---|
| 01 | 開場動畫 | `intro` | 載入頁面後自動播放 | `drawIntro()` | 3103 | ss_2060m4m3g |
| 02 | 主選單 | `menu` | 從 `intro` 進入後（或手動切換） | `drawMenu()` | 3558 | ss_9096138tb |
| 03 | 敵將出場 | `char_transition` | 選完獎勵後、進下一關前 | `drawCharTransition()` | 3245 | ss_4131kae8v |
| 04 | 關卡過場 | `transition` | 切換新關卡的中介動畫 | `drawStageTransition()` | 3377 | ss_9892bq2x1 |
| 05 | 排牌中（標準） | `placing` | 主玩法狀態 | `drawBattlefield()` + `drawHand()` | 3905 / 7212 | ss_5004vwhj0 |
| 06 | 龍膽換位模式 | `placing` + `_skillMode=true` | 按下「龍膽換位」按鈕（消 4 氣） | `drawBattlefield()`（按鈕標籤改變） | 1817–1832 邏輯 | ss_0196z5y85 |
| 07 | 夢幻國度 | `placing` + `inFantasyland=true` | 前墩 ≥ QQ 觸發，下一場滿手 13 張自由排 | `drawBattlefield()` + `drawFLConfirm()` | 4447 | ss_5302e9y8h |
| 08 | 戰報面板 | `placing` + `showLog=true` | 點右上角「牌局」按鈕呼出 | `drawLogPanel()` | 6620 | ss_5219g8nai |
| 09 | 戰鬥結算 | `battle` | 5 回合排牌完，自動觸發 `resolveBattle()` | `drawResults()` | 6154 | ss_18809o3ay |
| 10 | 戰後繳獲（選遺物） | `reward` | 戰勝後 | `drawReward()` | 7376 | ss_9490l8f9i |
| 11 | 七進七出（勝利） | `victory` | 通關全 10 關 | `drawVictory()` | 3814 | ss_9653g52i2 |
| 12 | 陣亡（失敗） | `gameover` | HP 歸零 | `drawGameOver()` | 3752 | ss_468585z8j |

## 每個畫面的關鍵元素

### 01 · 開場動畫（intro）
趙雲立繪背景 + 金色標題「趙雲菠蘿傳」+ 底部提示「點一下進入遊戲」。7 秒後自動進 `startGame()`，邏輯在 line 3236。

### 02 · 主選單（menu）
頂部小字「蜀漢‧趙子龍」、大標題「長坂坡」、副標「趙雲菠蘿傳 v0.6」、出戰按鈕、底下整捲卷軸寫「戰場規則」（對戰 / 鬼牌 / 5 回合 / 夢幻國度 / 三道規則 / 傷害公式 / 操作）。資訊密度高、教學一次給足。

### 03 · 敵將出場（char_transition）
頂部關卡標籤「— 第 1 關 —」、敵將立繪（曹軍小兵）、姓名、台詞「前方有人！給我拿下！」。

### 04 · 關卡過場（transition）
動畫狀態，可能在切換到下一關時先顯示前一個 BOSS 的回顧或下一關 BOSS 的預告。截圖時抓到的是夏侯惇（第 2 關 BOSS）的局部立繪，畫面有撕裂感、可能是動畫中段。

### 05 · 排牌中 · 標準（placing）
**核心玩法畫面**。布局：
- 上半：敵方立繪（曹軍小兵）+ 三道牌位（前鋒/中軍/後陣，5 槽）
- 中央分隔：本回合台詞「將 5 張全部擺入三道」+ 牌局/音效按鈕
- 下半：玩家立繪（常山趙子龍）+ 三道牌位
- 底部：HP / 氣值 / 連擊條 + 手牌（手中 5 張：鬼/5♠/5♥/7♦/4♦）+ 三技能按鈕（龍膽換位 / 七進七出 / 重置）+ 神兵 ►

### 06 · 龍膽換位模式（_skillMode）
與 05 完全相同，差別只在底部按鈕：「龍膽換位」變成「**選第一張**」（提示玩家點一張要交換的已排牌）。視覺差異很微妙——從可用性角度看可能要強化模式進入的視覺回饋（全螢幕邊框 highlight、暗化非可選元素之類）。

### 07 · 夢幻國度（inFantasyland）
與 05 差異：
- 中央 banner 變紫色，加上 ★ 符號
- 底部按鈕變成「全部重置 / 花色排序 / 點數排序」（不是技能按鈕）
- 進入此模式時會一次發 13 張，自由排三道，無回合制

### 08 · 戰報面板（showLog）
半透明黑色 overlay 蓋住主畫面，標題「— 牌局紀錄 —」。當前狀態顯示「尚無紀錄」（剛開局所以無歷史）。實戰中會列出每一場的勝負與牌型對比。

### 09 · 戰鬥結算（battle）
**佈局清晰**：
- 頂部「戰鬥結算」標題 + 副題「我軍潰退，請整軍再戰」
- 三排對比框（紅=敗 / 藍=平 / 藍=平），中央顯示勝負字（敗/平/勝）+ 我方傷害數字
- 底部「— 本場戰果 —」面板：我方承傷 / 對手他媒幻 / 敗方損傷
- 「再戰」按鈕
- 浮動 -32 飄字效果

> **注意**：截圖中卡片顯示為 placeholder 字符（"unfair/affected"）是因為我用 mock 資料觸發的 resolveBattle，rank 格式不完全匹配資產載入器；實戰中會正常顯示。

### 10 · 戰後繳獲（reward）
**最漂亮的畫面之一**。
- 標題「戰後繳獲 — 神兵現世 —」
- 三張紅色卡牌排列：每張上方圓形遺物圖示、名稱（紅字）、效果說明（白字）、底部紅色標籤按鈕
- 範例：長坂魂（陣亡反擊）/ 阿斗護符（不死鳥）/ 一騎當千（掃三軍）
- 底部小字「本次只能選擇一項，選定後不可更改」

### 11 · 七進七出（victory）
- 趙雲騎馬背景圖（黃昏/烈日色調）
- 金色大字「七進七出」
- 副標「懷抱阿斗，殺出長坂坡！擊潰 10 路敵將！」+「子龍一身都是膽也」引言
- 中段引言框「董卓：『以後初一十五不吃素。』董卓後來確實沒有再吃過素。」（這個引言看起來是彩蛋／冷知識，跟趙雲關連弱，可能是值得檢視的 UX 細節）
- 「再戰一場」按鈕
- 浮動光點裝飾

### 12 · 陣亡（gameover）
- 暗紅色背景 + 紅色光點
- 大字「陣亡」
- 副標「撐到了第 1 關 — 曹軍小兵」+「阿斗未能救出，英魂將化為下局力量」
- 「再戰一場」按鈕

## 技術觀察（給後續優化任務參考）

1. **canvas 固定 720×1280**：垂直手機比例，桌面瀏覽器有大量 dead space。如果要做桌面適配可考慮 letterbox 或設計橫向 layout。
2. **phase 切換是字串比對**：line 3049–3057 的 if-else 鏈用 \`game.phase === 'X'\` 一路比下去——重構時可以考慮 phase → drawFn 的 dispatch table。
3. **子模式 = 多布林旗標**：\`_skillMode / inFantasyland / showLog / mustPlace / aiThinking\` 互相覆蓋，沒有單一 source of truth；UI 邏輯也散落在各個 draw 函數中（每個 draw 函數都自己檢查布林）。重構時可考慮整合成 mode enum。
4. **截圖中卡片 placeholder text**：說明資產載入有 fallback 行為但 fallback 內容（"unfair/und/affected"）看起來像 dummy debug 文字，可能是某個 PORTRAIT_B64 / 卡牌 fallback 邏輯的 bug。
5. **char_transition 與 transition 都是過場**：兩個 phase 功能重疊，可能可以合併。
