# 趙雲菠蘿傳 — 開發交接文件

> 此文件供新 Claude session 接手繼續開發。
> 最後更新：2026-05-04 · 主檔 v0.7.x · 38 個 task 已完成

## 🎯 專案速覽

**遊戲類型**：三國題材 Pineapple Open-Face Chinese Poker（菠蘿開放式拉米）卡牌戰鬥
**主玩法**：趙雲（蜀）vs 曹軍 10 關 BOSS，撲克牌 + 三道布陣（前鋒/中軍/後陣）+ 神兵遺物
**線上版**：https://kenhsieh1211.github.io/alphaforge/zhaoyun.html
**Repo**：kenhsieh1211/alphaforge（main branch，public）

---

## 🏗️ 技術架構

### 主檔結構（單檔 HTML）
- `D:\.Claude\.Pineapple\zhaoyun_pineapple_v06.html`（~8400 行 / 956 KB）
- 部署到 GitHub Pages 後檔名為 `zhaoyun.html`

### 渲染雙軌制
1. **Canvas（720×1280 解析度）**：戰鬥畫面、結算、過場、武將立繪
   - 純 JS / 純 vanilla（無框架）
   - phase machine 驅動：intro / menu / placing / battle / reward / char_transition / gameover / victory
   - PHASE_RENDERERS dispatcher 表（line ~3303）
2. **React DOM 主選單**（蓋在 canvas 上的 overlay）
   - React 18 + Babel Standalone（CDN）
   - inline `<script type="text/babel">` 區塊（在 v06.html 末尾）
   - 元件：MenuApp / PrimaryBtn / ListBtn / IconBtn / RulesModal / SettingsModal / HeroesModal / RulesModal
   - 設計 tokens 物件 `TT`（color / font / ease）

### 通訊：vanilla JS ↔ React
- `let game = {}` (script-scoped) **必須** `window.game = game` 暴露給 React
- React 用 `useEffect setInterval` 每 200-300ms poll `window.game.phase`
- React 觸發 `window.startGame()` / `window.loadGame()` / `window.toggleAudioMute()` 等

### 關鍵全域變數
- `game.phase` — 'menu' / 'placing' / 'battle' / 'reward' / 'char_transition' / 'gameover' / 'victory'
- `game.rows` / `game.enemyRows` — { front, middle, back } card arrays
- `game.hand` / `game.discards` — 手牌 / 棄牌
- `game.relics` — 已取得遺物 array
- `game._qi` — 氣值 0-10
- `game._skillMode` / `game._skillMode2` — 龍膽換位 / 七進七出 mode flags
- `_ASSET_IMGS` — 外部載入的圖（key = 武將名 / asset id）
- `_PORTRAIT_IMGS` — base64 嵌入立繪（fallback）
- `_RELIC_SHEET_IMG` + `RELIC_SHEET_MAP` — 神兵 sprite sheet（8×6 grid，cell=256）

### 關鍵函數位置（行數有漂移，用 grep 校正）
- `initGame()` — line ~895
- `startGame()` — line ~1280
- `chooseReward()` — line ~2580（含 saveProgress hook）
- `resolveBattle()` — line ~1990 / `showNextBattleStep()`
- `drawBattlefield()` — line ~3795
- `drawResults()` — line ~5750（已拆 4 函數：_computeResultsContext / _drawResultsBg / _drawResultsHeader / _drawResultsRows + sections 4-6 inline）
- `drawCard()` — line ~6700
- `handleClick()` — line ~7470（已拆 8 個 sub-handler：handleGlobalUiClick / handlePhaseScreenClick / handleBattleResultClick / handleLogPanelClick / handlePlacingButtonClick / handleHandCardClick / handleSkillModeClick / handleRowZoneClick + handleSkill2DiscardClick）
- React MenuApp + HEROES const — line ~7975（在 babel script 區塊內）

---

## 📁 資料夾結構

```
.Pineapple/
├── zhaoyun_pineapple_v06.html     ⭐ 主檔
├── README.md                       
├── _deploy.js                      舊部署腳本（token hardcoded，已警告 user）
│
├── assets/
│   ├── logo.png                    主選單 logo (376 KB)
│   ├── zhaoyun_hero.jpg            主選單立繪 (117 KB)
│   ├── shenbing_icons/             47 個 256×256 神兵 icon 個別檔（備用）
│   └── ui/                         其他 UI sprite（很少用）
│
├── characters/
│   ├── zhaoyun_s1.jpg ~ s5.jpg     5 個趙雲階段 (s1=最乾淨/s5=滿身傷)
│   └── 01-10_*.jpg                 10 個 BOSS 立繪
│
├── relic_sheet.png                 神兵 sprite sheet 2048×1536（8×6 cells，每 256×256），1.19 MB
├── battle_result_bg.png            戰鬥結算背景
├── victory_bg.png                  勝利畫面背景
│
├── docs/
│   ├── TODO.md                     待辦清單
│   ├── BALANCE_AUDIT.md            v0.6 平衡度檢視報告
│   ├── UI_SCREENS_INVENTORY.md     12 UI 畫面索引
│   └── HANDOVER.md                 ⭐ 本檔
│
├── _backups/                       約 25+ 個版本備份 .html
│   └── dev_tools/                  舊開發腳本
│
├── _trash/                         126 MB 廢檔（card/ + 雜檔），用戶可手動 rm
└── .ui_update/                     UI v2 設計源檔（React JSX 參考用）
```

---

## 🛠️ 開發流程

### 本機 server
```powershell
cd D:\.Claude\.Pineapple
$l=[System.Net.HttpListener]::new();$l.Prefixes.Add('http://localhost:8000/');$l.Start();while($l.IsListening){$c=$l.GetContext();$p=Join-Path (Get-Location) ([Uri]::UnescapeDataString($c.Request.Url.LocalPath.TrimStart('/')));if(Test-Path $p -PathType Leaf){$ext=[IO.Path]::GetExtension($p).ToLower();$mime=@{'.html'='text/html';'.js'='application/javascript';'.css'='text/css';'.png'='image/png';'.jpg'='image/jpeg';'.json'='application/json'}[$ext];if($mime){$c.Response.ContentType=$mime};$b=[IO.File]::ReadAllBytes($p);$c.Response.OutputStream.Write($b,0,$b.Length)}else{$c.Response.StatusCode=404};$c.Response.Close()}
```
然後開 `http://localhost:8000/zhaoyun_pineapple_v06.html`

### 部署（GitHub Pages）
- 用 Python urllib + GitHub Contents API（範例見 _backups/dev_tools 或 commit history）
- Token 已暴露在 _deploy.js — **新 session 第一件事建議讓 user revoke**
- 推檔流程：GET sha → PUT base64 content
- Build 約 30-60 秒會在 https://kenhsieh1211.github.io/alphaforge/zhaoyun.html 生效

---

## ⚠️ 重要陷阱（踩過的坑）

### 1. TDZ 致命 bug（最嚴重）
- `let CONFIG = {...}`、`let game = {}` 等 top-level 宣告在未到達該行時觸碰會拋 ReferenceError
- **症狀**：console 看不到錯誤，但 game 物件是空的、整個 script 沒跑完
- **教訓**：移動 `_loadExternalAssets()` 等呼叫位置時，必須在所有依賴 const 宣告之後

### 2. window.game 同步
- `let game = {}` 不會自動掛 window
- initGame() 重新 `game = {...}` 後，必須 `window.game = game`
- **位置**：line 898-899（初始）+ initGame() 結尾

### 3. 沙盒 fs 限制
- bash 內 `rm` 失敗（"Operation not permitted"），但 `mv` 成功
- 整理檔案要用 `mv` 到 `_trash/` 替代刪除
- 用 Python 寫主檔時 read 整個文件 OK，但 grep 含 PORTRAIT_B64 base64 行會超 token，要用 `--head_limit` 或 Python 處理

### 4. Chrome MCP 不穩
- 行動裝置 viewport 變化、tab 隱藏時，rAF throttled
- screenshot 工具偶爾 CDP error，要 fallback 用 pixel sampling 或 fetch HTML
- file:// URL 不能跑 React + Babel（CORS）—— 必須走 http server

### 5. drawXxx 大規模 refactor 風險
- 之前一次 cleanup 寫錯 Python：`return content[:start] + ''` 而非 `+ content[end:]`，把 8400 行誤刪
- **教訓**：批次 string replacement 加 safety check：
  - 單一變動 > 300 行警告
  - 檔案大小縮 > 50% 異常停手
  - 從 GitHub 重 fetch 是 last resort

---

## 📊 目前狀態（v0.7.x）

### 已完成的 38 個 milestone（精選）
- T6 phase dispatcher / T2 戰鬥特效隔離 / T3 handleClick 拆分
- T5 5 個 canvas primitives (vGrad / fillRoundRectVGrad / textCentered / drawDoubleBorder)
- TODO-5 drawResults 拆 4 子函數 / TODO-6a 移除 transition 死碼 (-188 行)
- 平衡度 v0.7：七進七出 5 氣 / 龍膽破軍 ×2.5 / 中墩 Royalty /2 / 不死鳥 +12 等
- UI v2：React 主選單 + 手機框 + 趙雲立繪 + 武將圖鑑（11 武將+冷笑話+全屏 zoom）+ 繼續征途 save/load + 設定 modal（音量 slider）
- 神兵 v2：47 個新 icon 整合（256×256 grid）+ 1:1 語意映射
- 棄牌堆橫向堆疊 + 七進七出改棄牌交換
- 立繪 v2：5 個趙雲階段 + 10 BOSS（37 MB → 1.9 MB JPG）
- Mobile fix：JS-driven scale + 100dvh + haptic 觸覺回饋
- Dead code -539 行（log/discard/matchLog/drawBattleBackground/silhouettes 等）

### 撤銷的（誠實紀錄）
- T1 截圖 fallback（不是 bug，mock schema 錯）
- T8 AI memo（baseline 0.01ms 不痛）
- T6b 復活水墨過場（rollback：跟 char_transition 串連 9 秒太長）
- TODO-E logo cross-fade（React `<img>` 取代後失效）

---

## 🎯 下一步建議（依優先序）

### 🟢 P0 安全先做
1. **_deploy.js 改環境變數**：`process.env.GH_TOKEN`，token 不要 hardcode
2. **手動 revoke 舊 token**：https://github.com/settings/tokens（user 自己處理）
3. **驗證遊戲完整流程**：跑一遍 1→10 關，確認所有 v0.7.x 改動都沒破

### 🟡 P1 內容向
1. **第 11+ 關 / DLC**：通關曹操後新內容
2. **新技能**：替代龍膽換位 / 七進七出
3. **新模式**：Endless / Daily Challenge / Boss Rush
4. **武將圖鑑加阿斗 / 諸葛亮 / 關羽 / 張飛**（需 portrait）
5. **平衡度二輪檢視**：實玩過 v0.7 後再次調整

### 🟡 P1 程式向
1. **戰鬥畫面套新 design tokens**：目前還是舊朱紅金線，跟主選單黑鐵金線不統一
2. **textCentered 推廣**：60+ 處 fillText 可改用 helper（drawCharTransition / drawGameOver / drawVictory 是好目標）
3. **enemyDiscards 顯示**：Pineapple 規則「棄牌全部公開」但敵方棄牌沒在 UI 顯示
4. **暫停按鈕改進**：右上角「↩ 主選單」按鈕已有，可加更多 in-game 操作（音效、設定）

### 🔴 P2 大重構（需 user 同意才動）
1. **TODO-7 game state 物件分群**：137 屬性 → game.battle.* / .player.* 等（風險高，要動 200+ reference）
2. **TODO-8 canvas → 部分 DOM 混合**：HP bar / 按鈕 / 計時器改 DOM 可省 70% rerender
3. **TODO-9 拆檔 + build script**：8400 行單檔 → 多檔（dev）+ build concat（deploy）

---

## 🤝 與 user 協作風格筆記

User 的偏好（重要）：
1. **直接動手不要囉唆**：「請繼續」「直接執行吧」「你先排任務吧」 = 不要再問細節，自己決定優先序
2. **單檔交付**：不要拆檔（除非 P2 有計畫）
3. **頻繁 push 部署**：每個改動都要推 GitHub Pages 上線（不要等批次）
4. **每個改動要備份**：`cp ... _backups/zhaoyun_pineapple_v06_before_X.html`
5. **報告要短**：少廢話、表格 + 重點，不要寫長篇 markdown
6. **挑戰問題本質**：「真正要解決什麼問題？對玩家什麼價值？」
7. **承認不確定 / 撤銷錯誤**：T1 / T8 / T6b 都被誠實 rollback 過

User 不喜歡的：
- 沒重點的長篇分析
- 重複問已答過的問題
- 改完不上線（user 一定會在手機驗收）
- 邊做邊偷懶不留備份

---

## 🔧 常用 Python 模板

### Patch HTML 主檔
```python
path = '/sessions/.../zhaoyun_pineapple_v06.html'
with open(path) as f: c = f.read()

old = """要找的字串"""
new = """要替換的字串"""
assert old in c, "marker not found"
c = c.replace(old, new, 1)

with open(path, 'w') as f: f.write(c)
```

### 部署到 GitHub
```python
import os, base64, json
from urllib import request, error
TOKEN = '<env var or 重新生 token>'
REPO = 'kenhsieh1211/alphaforge'

def api(method, path, body=None):
    url = f'https://api.github.com/repos/{REPO}/{path}'
    headers = {'Authorization': f'Bearer {TOKEN}', 'Accept': 'application/vnd.github+json', 'User-Agent': 'claude-deploy'}
    if body:
        headers['Content-Type'] = 'application/json'; body = json.dumps(body).encode('utf-8')
    req = request.Request(url, data=body, method=method, headers=headers)
    try:
        with request.urlopen(req, timeout=60) as r: return r.status, json.loads(r.read() or b'{}')
    except error.HTTPError as e:
        try: return e.code, json.loads(e.read() or b'{}')
        except: return e.code, {}

sha = api('GET', 'contents/zhaoyun.html')[1].get('sha')
with open(LOCAL_PATH, 'rb') as f:
    content = base64.b64encode(f.read()).decode()
payload = {'message': COMMIT_MSG, 'content': content, 'branch': 'main', 'sha': sha}
code, body = api('PUT', 'contents/zhaoyun.html', payload)
print(f"Push: HTTP {code}, commit: {body.get('commit',{}).get('sha','?')[:7]}")
```

### 安全的函數移除（修正版）
```python
import re
def remove_function(content, fn_name):
    pat = rf'function {fn_name}\([^)]*\) \{{'
    m = re.search(pat, content)
    if not m: return content, 0, False
    start = m.start()
    i = m.end()
    depth = 1
    while i < len(content) and depth > 0:
        ch = content[i]
        if ch == '{': depth += 1
        elif ch == '}': depth -= 1
        i += 1
    if depth != 0: return content, 0, False
    end = i
    if end < len(content) and content[end] == '\n':
        end += 1
    block = content[start:end]
    n_lines = block.count('\n')
    # 重要：保留 start 之前 + end 之後
    new_content = content[:start] + content[end:]
    # Safety: 單函數 >300 行 / 檔案縮 >50% 跳過
    if n_lines > 300:
        return content, 0, False
    if len(new_content) < len(content) * 0.5:
        return content, 0, False
    return new_content, n_lines, True
```

---

## 📞 快速確認 checklist（新 session 第一件事）

```bash
# 1. 確認檔案還在
ls /sessions/eloquent-dreamy-archimedes/mnt/.Pineapple/zhaoyun_pineapple_v06.html

# 2. 看 git status / 最新 commit
# (用 Python urllib 打 GitHub API)

# 3. 看 TODO.md 哪些待辦
cat /sessions/eloquent-dreamy-archimedes/mnt/.Pineapple/docs/TODO.md

# 4. 看主檔行數（目前 ~8400 line）
wc -l /sessions/eloquent-dreamy-archimedes/mnt/.Pineapple/zhaoyun_pineapple_v06.html
```

---

**結語**：v0.7.x 已是穩定可玩、視覺上接近完成版。下一階段重心應該是 **內容擴充 + 平衡度二輪 + 戰鬥畫面 design token 統一**。重大重構（TODO-7/8/9）建議等 user 主動提才動。

祝接手順利 🐉
