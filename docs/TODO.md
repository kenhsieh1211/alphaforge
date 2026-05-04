# 趙雲菠蘿傳 v0.6/0.7 — 進度與待辦

> 最後更新：2026-05-04
> v0.6: code 結構整理 + UI v2 整合
> v0.7: 平衡度調整（內容階段）

## ✅ 已完成（24 項任務，倒序）

### v0.7 (Balance + Polish)
- ✅ **設定 modal 強化** — 存檔資訊顯示 + 清除按鈕 + About 區塊
- ✅ **平衡度建議實作** — 9 項數值調整（七進七出 5氣/55HP、龍膽破軍 ×2.5、張遼 ×1.5、中墩 Royalty /2、夏侯惇 95HP、不死鳥 +12HP、前墩三條 Royalty +13、紫色遺物 weight ×1.67）

### v0.6 整理階段
- ✅ TODO-A 平衡度檢視（9288 字 BALANCE_AUDIT.md）
- ✅ TODO-5 drawResults 拆 4 函數（_computeResultsContext + Bg + Header + Rows + orchestrator）
- ✅ TODO-G char_transition 縮時 4.0s→2.5s
- ✅ TODO-F drawStageIndicator 函數本體刪除 (-216 行)
- ✅ 移除戰場背景雜亂元素（騎兵/船艦/火炬/塵埃 全清）
- ✅ UI v2 Milestone 2: 繼續征途 save/load + 武將圖鑑 modal（11 武將文案）
- ✅ 主選單加趙雲立繪（720×720 / mask gradient / 117 KB JPG）
- ✅ 修 _loadExternalAssets TDZ 致命 bug（之前一直存在的隱性錯誤）
- ✅ UI v2 Milestone 1: DOM 主選單（React + Babel CDN + 手機框 + 4 ListBtn）
- ✅ Logo 載入修正 v1+v2（壓 376KB / 雙 URL fallback / cache-bust）
- ✅ 緊急修 3 bug：T6b rollback / foul 順序 / 隱藏進度條
- ✅ TODO-6a 移除 transition 死碼 (-188 行)
- ✅ TODO-4 drawScrollBar 漸層 refactor
- ✅ T5 5 primitives + drawMiniCard / drawCard / drawScrollButton refactor
- ✅ T3 handleClick 197 行 → 主 dispatcher + 8 sub-handler
- ✅ T2 戰鬥殘留特效隔離
- ✅ T6 phase if-else → PHASE_RENDERERS dispatcher

### 撤銷的（誠實紀錄）
- ❌ T1 截圖 fallback bug（不存在，是測試 mock schema 錯）
- ❌ T8 AI memoization（baseline 0.01ms，不痛）
- ❌ T6b 復活水墨過場（rollback：跟 char_transition 串連 9 秒太長）
- ❌ TODO-E cross-fade 書法字 → logo（React `<img>` 取代 canvas fallback 後失效）

---

## 🟢 P0 · 順手快做（< 30 分鐘）

### TODO-2. 註解微錯
- 已修：line 號漂移處改成「定義於 X 上方」不寫死

### TODO-3. textCentered 推廣（部分完成）
- T5 已建 helper，drawMiniCard 用了
- 全檔 168 處 fillText 中 60+ 可改，但收益每處只 1-2 行
- 建議：只在「函數內 fillText ≥ 5 且配色簡單」處做

## 🟡 P1 · 中等工程

### TODO-5b. drawResults sections 4-6 拆分（部分完成）
- 1-3 已抽 (Bg/Header/Rows)
- 4-6 共享 _extraY 仍 inline
- 想做更徹底可改 _extraY 為 return 值傳遞

### 戰鬥畫面套新 design tokens
- 目前戰鬥/結算畫面還是舊朱紅金線
- 套新黑鐵金線 + 鋼藍 tokens 統一風格

### char_transition 升級
- 也用新 design tokens
- 加跳過按鈕（目前要點兩次才跳過）

## 🔴 P2 · 大重構（半天以上）

### TODO-7. game state 物件分群
- 137 屬性 → game.battle.* / .player.* / .enemy.* / .relics.* / .ui.* / .flags.* / .audio.*
- 風險高，要動 200+ reference

### TODO-8. canvas → 部分 DOM 混合
- 固定 UI（按鈕、HP bar、計時器）改 DOM 可省 70% rerender

### TODO-9. 拆檔 + build script
- 8600+ 行單檔 → 多檔 + build concat

## 🌱 內容向

### TODO-B. 內容擴充
- 第 11+ 關（後傳 / DLC）
- 新技能（替代「七進七出」）
- 新模式（Endless / Daily / Boss Rush）

### TODO-D. WebP / 進一步壓縮 logo
- logo.png 376 KB → WebP 可降到 ~200 KB
- 趙雲立繪 zhaoyun_hero.jpg 117 KB OK

### 武將圖鑑強化
- 11 武將 → 加更多細節（生卒年、技能、BOSS 戰技巧）
- 加阿斗 / 諸葛亮 / 關羽 / 張飛 等蜀漢相關武將（需 portrait）

### 平衡度二輪檢視
- 在玩家實際玩過 v0.7 後再次檢視
- 收集死亡關卡分布、遺物選擇分布資料

---

## 備份檔對照

| 檔案 | 內容 |
|---|---|
| zhaoyun_pineapple_v06.html | **當前主檔**（含所有 ✅ 改動 + Balance v0.7） |
| zhaoyun_pineapple_v06_before_balance.html | 平衡修正前 |
| zhaoyun_pineapple_v06_before_TODO_F_G_5.html | 拆函數前 |
| zhaoyun_pineapple_v06_before_cleanbg.html | 清戰場背景前 |
| zhaoyun_pineapple_v06_before_milestone2.html | 繼續征途+武將圖鑑前 |
| zhaoyun_pineapple_v06_before_zhaoyun_portrait.html | 加趙雲立繪前 |
| zhaoyun_pineapple_v06_before_phoneframe.html | 改手機框前 |
| zhaoyun_pineapple_v06_before_logofix2.html | logo fix v2 前 |
| zhaoyun_pineapple_v06_before_emergency_fix.html | 3 bug 修正前 |
| zhaoyun_pineapple_v06_before_logofix.html | logo fix 前 |
| zhaoyun_pineapple_v06_before_T6b.html | T6b 之前（**有完整水墨過場**） |
| zhaoyun_pineapple_v06_before_T5.html | T5 之前 |
| zhaoyun_pineapple_v06_before_T3.html | T3 之前 |
| zhaoyun_pineapple_v06_before_T1T2T6T8.html | 全部優化之前（**最初狀態**） |
| zhaoyun_pineapple_v06_before_ui_v2.html | UI v2 整合前 |
| zhaoyun_pineapple_v06_before_phoneframe.html | phone frame 改造前 |
| logo.png.png.before_compress.bak | 2.93 MB 原始 logo |

## 相關文件

| 檔案 | 用途 |
|---|---|
| `BALANCE_AUDIT.md` | v0.6 平衡度檢視報告（9 大主題分析） |
| `UI_SCREENS_INVENTORY.md` | 12 個 UI 畫面索引 + 截圖描述 |
| `TODO.md` | 本檔（待辦清單） |
