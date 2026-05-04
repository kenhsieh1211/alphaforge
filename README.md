# 趙雲菠蘿傳 v0.7

> 三國題材 Pineapple Open-Face Chinese Poker 卡牌戰鬥遊戲。
> 線上版：https://kenhsieh1211.github.io/alphaforge/zhaoyun.html

## 📁 資料夾結構

```
.Pineapple/
├── zhaoyun_pineapple_v06.html     ⭐ 主檔（v0.7 / 8652 行 / 968 KB）
├── _deploy.js                      部署腳本（推上 GitHub Pages）
├── README.md                       本檔
│
├── assets/                         遊戲使用的圖片資產
│   ├── logo.png                    主選單 logo（376 KB / 800×533）
│   ├── zhaoyun_hero.jpg            主選單立繪（117 KB / 720×720）
│   └── ui/                         其他 UI sprite
│
├── characters/                     11 位武將立繪（25 MB）
│   ├── 00_ZhaoYun_MainHero.png    主角
│   └── 01-10_*_*.png               10 位 BOSS
│
├── battle_result_bg.png            戰鬥結算背景（drawResults 用）
├── victory_bg.png                  勝利畫面背景（drawVictory 用）
├── relic_sheet.png                 神兵 sprite sheet 8×6（drawRelic 用）
│
├── docs/                           專案文件
│   ├── TODO.md                     待辦清單
│   ├── BALANCE_AUDIT.md            v0.6 平衡度檢視報告
│   └── UI_SCREENS_INVENTORY.md     12 個 UI 畫面索引
│
├── _backups/                       17 個版本備份（每個里程碑可回退）
│   ├── zhaoyun_pineapple_v06_before_*.html
│   ├── zhaoyun_pineapple_v05_10.html
│   └── dev_tools/                  舊開發工具腳本
│
├── _trash/                         🗑️ 確定不再用的檔案（可手動刪除節省空間）
│   ├── card/                       54 張舊撲克牌圖（121 MB，drawCard 已純 canvas 不需要）
│   ├── logo.png.png                舊 logo 重複檔
│   ├── logo.png.png.before_compress.bak  原始未壓縮 logo（2.93 MB）
│   ├── chatgpt_old_ref.png         舊 ChatGPT 參考圖
│   ├── screenshot.png              舊截圖
│   └── ui_screenshots/             空資料夾
│
├── .ui_update/                     UI v2 設計源檔（React JSX 參考用）
└── .claude/                        Claude Code 設定
```

## 🎮 開玩

**線上版**：https://kenhsieh1211.github.io/alphaforge/zhaoyun.html

**本地開發**：開 PowerShell 在 .Pineapple 跑：
```powershell
$l=[System.Net.HttpListener]::new();$l.Prefixes.Add('http://localhost:8000/');$l.Start();while($l.IsListening){$c=$l.GetContext();$p=Join-Path (Get-Location) ([Uri]::UnescapeDataString($c.Request.Url.LocalPath.TrimStart('/')));if(Test-Path $p -PathType Leaf){$ext=[IO.Path]::GetExtension($p).ToLower();$mime=@{'.html'='text/html';'.js'='application/javascript';'.css'='text/css';'.png'='image/png';'.jpg'='image/jpeg';'.json'='application/json'}[$ext];if($mime){$c.Response.ContentType=$mime};$b=[IO.File]::ReadAllBytes($p);$c.Response.OutputStream.Write($b,0,$b.Length)}else{$c.Response.StatusCode=404};$c.Response.Close()}
```
然後開 `http://localhost:8000/zhaoyun_pineapple_v06.html`

## 🚀 部署

```bash
node _deploy.js
```

**⚠️ 重要**：`_deploy.js` 內 GitHub token 已 hardcode 且外露過。請：
1. 去 https://github.com/settings/tokens revoke 舊 token
2. 改用環境變數：`const TOKEN = process.env.GH_TOKEN;`
3. 部署：`$env:GH_TOKEN='ghp_新token'; node _deploy.js`

## 🗑️ 清理建議

`_trash/` 資料夾佔 126 MB，可整個手動刪除：
```powershell
Remove-Item -Recurse -Force D:\.Claude\.Pineapple\_trash
```

刪掉後 `.Pineapple` 從 218 MB 降到 ~92 MB。

## 📋 版本歷史

- **v0.6** (UI v2)：React DOM 主選單 + 手機框 + 趙雲立繪 + 武將圖鑑（11 位）+ 繼續征途存檔
- **v0.7** (Balance)：七進七出 5 氣 / 龍膽破軍 ×2.5 / 張遼 ×1.5 / 中墩 Royalty /2 / 夏侯惇 95 HP / 不死鳥 +12 / 紫色遺物 weight 0.5

詳見 `docs/TODO.md` 完成項目列表。
