# 騎士氣象台 — 台北公路車天氣 PWA

## 專案資訊
- **永久網址**：https://chenchunkuang0111-lang.github.io/cycling-weather-taipei/
- **分享頁**：https://chenchunkuang0111-lang.github.io/cycling-weather-taipei/add-to-home.html
- **Repo**：https://github.com/chenchunkuang0111-lang/cycling-weather-taipei（**必須維持 public**，見已知坑 1）
- **部署**：GitHub Pages，source = `main` 分支 / `/`（root）
- **檔案結構**：純靜態 HTML，無 build step。`index.html` / `add-to-home.html` / `boundary.json` / `README.md` / `docs/`

## 已完成功能

| # | 任務 | 後端 | 前端 | 驗證 | Commit |
|---|---|---|---|---|---|
| 1 | 三日逐時預報 + 雷達 + 攝影機 + 破紀錄指數 + PWA | Open-Meteo / NCDR / livecam.tw | index.html | 線上可開 | （歷史） |
| 2 | 加入主畫面引導頁 | — | add-to-home.html | 線上可開 | b0a1ccc |
| 3 | 新增三條路線（銅鑼圈獅潭、106 福隆、太平山） | 路線設定 | index.html | 顯示正常 | 811ce45 |
| 4 | 修正「銅麅圈 → 銅鑼圈」錯字 | — | index.html | 文字正確 | 167421a |
| 5 | **修復 GitHub Pages 404** — repo 改 public 後重啟 Pages | gh API | — | 4 個關鍵網址 HTTP 200 | （本次 commit） |

## 已知坑

### 1. Repo 必須維持 public（不能改回 private）
- **原因**：免費 GitHub 方案的 GitHub Pages **不支援 private repo**。改回 private 會立即 404，永久網址會失效。
- **歷史故障**：2026-05-10 發現網址全部 404。root cause：repo 不知何時被切成 private，Pages API 回「Your current plan does not support GitHub Pages for this repository」。2026-05-11 把 repo 改回 public 並重啟 Pages 後恢復。
- **處置**：除非升級 GitHub Pro（付費），否則此 repo 絕對不能 private。

### 2. CSP 限制
- GitHub Pages 不能設自訂 HTTP header，所以攝影機 iframe 走「livecam.tw / 政府 CCTV」這類允許 embed 的來源。換來源前先在本機開 iframe 測，確認 X-Frame-Options 不擋。

### 3. PWA 快取
- manifest 內嵌 base64，沒有 service worker，所以「離線可看上次資料」是靠瀏覽器自身快取，不是 SW。改大版本時不需要更新 SW，但若日後加 SW 要小心 cache busting。

## 跨機接續
- 任何電腦：`cd ~/Projects/cycling-weather-taipei && git pull && cat CLAUDE.md`
- 修改後務必 commit + push（GitHub Pages 自動部署 main 分支，30-60 秒生效）

## 部署流程
1. 改 `index.html` 或其他靜態檔
2. `git add . && git commit -m "<繁中訊息>" && git push`
3. 等 30-60 秒
4. 重新整理 https://chenchunkuang0111-lang.github.io/cycling-weather-taipei/ 驗證

## 故障排除
- **網址 404**：先 `gh api repos/chenchunkuang0111-lang/cycling-weather-taipei/pages` 看 Pages 狀態，再 `gh repo view ... --json visibility` 看是不是 private。
- **內容沒更新**：清瀏覽器快取或 query string 加 `?v=<timestamp>`。
- **攝影機掛掉**：通常是上游服務改網址；搜尋 `index.html` 內該攝影機 URL 換新的。
