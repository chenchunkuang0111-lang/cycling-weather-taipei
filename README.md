# 台北公路車路線天氣監測 PWA 應用

> 騎士氣象台 — 台北公路車天氣 PWA
>
> （Design by 米咕嚕）

## 📱 立即使用

**主程式**：[https://chenchunkuang0111-lang.github.io/cycling-weather-taipei/](https://chenchunkuang0111-lang.github.io/cycling-weather-taipei/)

**📲 分享給朋友（含安裝引導）**：
```
https://chenchunkuang0111-lang.github.io/cycling-weather-taipei/add-to-home.html
```

---

## ✨ 功能特色

### 🚴 13 條北部熱門公路車路線
| 路線 | 距離 | 爬升 |
|------|------|------|
| 仰德小油 | 14.8 km | 805 m |
| 行義小油 | — | — |
| 風櫃嘴 | 10.8 km | 620 m |
| 冷水坑 | — | — |
| 巴拉卡 | — | — |
| 陽金 | — | — |
| 中正山 | 8.2 km | 546 m |
| 汐萬五指 | — | — |
| 大湖五指 | — | — |
| 淡金來回 | — | — |
| 塔塔加 | — | — |
| 思源啞口 | — | — |
| 西進武嶺 | 36.0 km | 2,103 m |

### 🌦 天氣監測
- 即時三日逐時預報（Open-Meteo API）
- 雷達雲圖（NCDR）
- 風向、風速、降雨機率、體感溫度
- 路面狀態（乾燥／微濕／潮濕）
- 各路段獨立天氣分析

### 📷 即時路況攝影機
- 每條路線多個分段攝影機
- 部分路段標示「同氣候帶影像供參考」
- 攝影機重新整理按鈕

### 📊 破紀錄指數
- 整體 0–100 分評分
- 各路段獨立指數
- 自動判斷「最佳視窗」時段
- 全線狀態（全線偏乾／部分潮濕等）

### 📱 PWA 體驗
- 加入手機主畫面後可像 App 全螢幕使用
- **跳出 App 30 秒後回來自動重新整理**
- **手機下拉重新整理（Pull-to-Refresh）**
- 離線可看上次資料

---

## 📂 專案結構

```
cycling-weather-taipei/
├── index.html              # 主應用程式 (PWA)
├── add-to-home.html        # 加入主畫面引導頁（給朋友分享用）
├── boundary.json           # 台灣 22 縣市 GeoJSON 邊界
├── README.md               # 本檔案
└── docs/                   # 開發文件
    ├── CHANGELOG.md        # 變更紀錄
    ├── FEATURES.md         # 功能設計說明
    └── DEVELOPMENT_LOG.md  # 開發過程紀錄
```

---

## 📖 開發文件

- [變更紀錄 (CHANGELOG)](docs/CHANGELOG.md) — 各版本變更內容
- [功能設計 (FEATURES)](docs/FEATURES.md) — 功能說明、攝影機資料、樣式
- [開發紀錄 (DEVELOPMENT_LOG)](docs/DEVELOPMENT_LOG.md) — 技術決策、問題解決過程

---

## 🛠 技術架構

| 項目 | 技術 |
|------|------|
| 前端 | 純 HTML / CSS / JavaScript（無 framework） |
| PWA | Web App Manifest（內嵌 base64） |
| 部署 | GitHub Pages |
| 氣象資料 | [Open-Meteo](https://open-meteo.com/) API |
| 雷達 | NCDR（國家災害防救科技中心） |
| 攝影機 | livecam.tw / 公路總局 / 台北市政府 CCTV |
| 地圖 | Leaflet + OpenStreetMap |

---

## 🚀 部署

本專案使用 GitHub Pages 自動部署 `main` 分支。

任何推送到 `main` 的 commit 都會在 30~60 秒內自動部署到正式環境。

---

## 📝 授權

本專案為個人使用，氣象與攝影機資料來源請依各服務條款使用。

---

## 👤 作者

- Repo Owner: chenchunkuang0111-lang
- UI Design: 米咕嚕
