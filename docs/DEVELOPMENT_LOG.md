# 開發紀錄 (Development Log)

本文件記錄專案的完整開發過程，包含問題、解決方案、技術決策。

---

## 2026-04-29：本期功能開發

### 階段一：UI 微調與資料更新

#### 任務清單
1. 標題後加入「(Design by 米咕嚕)」
2. 置頂按鍵位置互換 (風櫃嘴↔仰德小油、行義小油↔冷水坑)
3. 5 條路線的部分攝影機加入「同氣候帶影像供參考」浮水印
4. 中正山第 2 隻攝影機更換為「格致路凱旋路」
5. 西進武嶺路線：移除鳶峰、更換昆陽與武嶺牌樓攝影機

#### 開發過程

**問題 1：原始檔案結構分析**
- 檔案：`index.html` (90,441 bytes, 927 行)
- 路線資料以 `const ROUTES=[...]` 形式硬寫在 JavaScript 中
- 13 條路線，每條包含 `segs` 陣列（每個 seg 含 cam 物件）

**問題 2：中文字元編碼**
- 原始 HTML 使用數字實體 `&#xNNNN;` 表示中文
- 第一次修改時誤用 `&#x54AB;&#x565A;` 導致顯示「米咽嘽」
- 正確編碼：米 = 0x7C73、咕 = 0x5495、嚕 = 0x5695
- 修復：產生第二個 commit 修正字元

**問題 3：找出格致路凱旋路攝影機 ID**
- 透過 livecam.tw 的 cameras.json (約 7,464 個攝影機) 搜尋
- 找到 ID = `BOT373`，URL = `https://livecam.tw/cam/BOT373/`

**問題 4：seg 順序變動**
- 原始西進武嶺有 8 個 segs，移除「鳶峰」後變 7 個
- 操作順序很重要：先移除鳶峰，再依新索引更新昆陽與武嶺牌樓

**問題 5：原本的 cam URL 字串拼錯**
- 我預期的 `'\\u53F014\\u7532\\u753232K...'` 實際是 `'\\u53F014\\u753232K...'`
- 多寫了一個 \\u7532 (甲)
- 修正：用程式直接從 GitHub 下載原始檔對照

#### 技術手法

**直接修改 ROUTES 資料 (vs. 動態 patch)**
- 浮水印因為是視覺效果，使用 patch script + MutationObserver 注入較簡單
- 路線重排與 cam 更換則直接修改原始 JavaScript 物件，避免依賴 runtime patch

**透過 React Fiber 操作 CodeMirror 6**
- GitHub 編輯器使用 CodeMirror 6 + React 包裝
- 取得 EditorView 的路徑：
  1. `document.querySelector('.cm-editor').parentElement`
  2. 找 React fiber: `Object.keys(el).find(k => k.startsWith('__reactFiber'))`
  3. 向上遍歷 fiber tree 找 `type.displayName === 'CodeMirror'`
  4. 在 `memoizedState` hook chain 中找 `ref.current` (即 EditorView)
- 透過 `view.dispatch({changes:{from:0, to:doc.length, insert:newContent}})` 整檔取代

---

### 階段二：設計者署名位置調整

#### 任務
將「(Design by 米咕嚕)」從標題右側移到下方獨立一行。

#### 解法
原本的 inline span：
```html
<span style="font-size:.55em;...;margin-left:6px">…</span>
```
改為 block-level：
```html
<span style="display:block;font-size:.55em;...;margin-top:2px">…</span>
```

---

### 階段三：分享連結與 PWA 強化

#### 需求
1. 分享連結 → 朋友點擊直接加入手機主畫面
2. 跳出 App 再回來自動重新整理
3. 手機在最上方再往下拉自動重新整理

#### 重要釐清
**「強制」自動加入主畫面是不可能的**：
- iOS 與 Android 的安全限制：只有使用者本人能在瀏覽器選單按「加入主畫面」
- 解決方案：建立美觀的引導頁 `add-to-home.html`，依平台顯示步驟教學

#### 實作

**A. 引導頁 (`add-to-home.html`)**
- 4,772 bytes 獨立 HTML 檔
- 自動偵測 `navigator.userAgent` 顯示對應平台步驟
- 偵測 `window.navigator.standalone` 與 `(display-mode: standalone)` MQ
  - 已從主畫面開啟者自動跳轉到主程式

**B. visibilitychange 自動重整**
- 使用 `document.visibilityState` 取代舊的 `hidden/visible` 屬性
- 30 秒閾值避免短暫切換就重整
- 加上 `pageshow` 事件處理 iOS Safari bfcache

**C. Pull-to-refresh**
- 純原生 `touchstart/touchmove/touchend` 實作（不引入 library）
- 條件：`scrollY <= 0` 且 `dy > 0`
- 阻力係數 0.5（拉 1px 顯示 0.5px，類似 iOS 原生效果）
- 70px 觸發閾值
- 文字狀態切換：「下拉重新整理」→「釋放重新整理」→「整理中...」

#### 技術細節

**為什麼要 30 秒閾值？**
- 太短：使用者只是切去看一下通知就重整，影響體驗
- 太長：天氣資料已經過時還沒重整，失去意義
- 30 秒是「一次完整通知互動」的合理時長

**為什麼 pull-to-refresh 用阻力係數？**
- 沒有阻力 → 拖動感太靈敏，容易誤觸發
- 0.5 阻力 → 接近 iOS Safari 原生下拉感
- `Math.min(dy * 0.5, 120)` 限制最大顯示距離

---

## 已知限制

### A. 無法強制加入主畫面
- 受限於 iOS / Android 平台安全機制
- 解法：引導頁 + 步驟教學

### B. PWA 圖示與名稱
- 目前 manifest 透過 `data:application/json;base64,...` 形式內嵌
- 後續若要更換圖示需修改該 base64 內容

### C. 攝影機 URL 來源依賴第三方
- livecam.tw、thbcctv.thb.gov.tw、台北市政府 CCTV
- 第三方 URL 變更時需手動更新

---

## 部署與發布

### 平台
GitHub Pages，自動部署 `main` 分支。

### 部署時間
通常 commit 後 30~60 秒內完成。可透過下列方式驗證：
```javascript
fetch('/cycling-weather-taipei/?v=' + Date.now(), {cache:'no-store'})
  .then(r => r.text())
  .then(t => /* 檢查特徵字串 */);
```

### URL
- 主程式：https://chenchunkuang0111-lang.github.io/cycling-weather-taipei/
- 引導頁：https://chenchunkuang0111-lang.github.io/cycling-weather-taipei/add-to-home.html

---

## 工具鏈

| 工具 | 用途 |
|------|------|
| GitHub Pages | 靜態網站託管 |
| GitHub Web Editor | 線上編輯（透過 CodeMirror 6）|
| Open-Meteo API | 氣象預報資料 |
| NCDR | 雷達雲圖 |
| livecam.tw | 路線攝影機 |
| thbcctv.thb.gov.tw | 公路總局攝影機（武嶺路線）|
| 台北市政府 CCTV | 中正山攝影機 |
