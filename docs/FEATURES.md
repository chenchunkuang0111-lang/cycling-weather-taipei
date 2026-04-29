# 功能設計說明 (Features)

本文件詳細說明各項功能的設計理念、使用方式與技術實作。

---

## 一、UI 設計

### 1.1 標題區
- **主標題**：北部單車氣象台 (font-size: 20px, font-weight: 600)
- **設計者署名**：(Design by 米咕嚕)
  - 位置：標題正下方獨立一行 (`display:block`)
  - 字體：.55em 主標題大小
  - 透明度：.85
  - HTML：`<span style="display:block;font-size:.55em;font-weight:400;letter-spacing:0;opacity:.85;margin-top:2px">（Design by 米咕嚕）</span>`

### 1.2 路線按鍵排序
依據實用性與地理位置調整，最常用的路線置於前方：

| 順序 | 路線 ID | 路線名稱 |
|------|---------|----------|
| 1 | yangde | 仰德小油 |
| 2 | xingyi | 行義小油 |
| 3 | feng | 風櫃嘴 |
| 4 | leng | 冷水坑 |
| 5 | balaka | 巴拉卡 |
| 6 | yangjin | 陽金 |
| 7 | zhzheng | 中正山 |
| 8 | xiwan | 汐萬五指 |
| 9 | dahu | 大湖五指 |
| 10 | danjin | 淡金來回 |
| 11 | tatajia | 塔塔加 |
| 12 | siyuan | 思源啞口 |
| 13 | wuling | 西進武嶺 |

---

## 二、攝影機浮水印系統

### 2.1 設計目的
有些路線的攝影機並非位於路線本身，而是同氣候帶的鄰近攝影機（用於推估天氣狀況），標示「同氣候帶影像供參考」避免使用者誤解。

### 2.2 適用路線與攝影機

| 路線 | 套用浮水印的攝影機 (索引從 0 開始) |
|------|-----------------------------------|
| 風櫃嘴 (feng) | seg [1, 2, 3] (cams 2, 3, 4) |
| 冷水坑 (leng) | seg [1, 2] (cams 2, 3) |
| 巴拉卡 (balaka) | seg [1, 2] (cams 2, 3) |
| 陽金 (yangjin) | seg [1, 2] (cams 2, 3) |
| 中正山 (zhzheng) | seg [1, 2, 3] (cams 2, 3, 4) |

### 2.3 樣式
```css
.wm-overlay{
  position:absolute;left:50%;top:50%;
  transform:translate(-50%,-50%);
  color:#fff;font-weight:900;
  font-size:clamp(13px,3.8vw,22px);
  letter-spacing:.5px;
  text-shadow:0 2px 6px rgba(0,0,0,.85),0 0 12px rgba(0,0,0,.7);
  background:rgba(0,0,0,.42);
  padding:6px 10px;border-radius:8px;
  pointer-events:none;z-index:5;
  white-space:nowrap;text-align:center;
  max-width:92%;
}
```

### 2.4 動態套用機制
透過 `MutationObserver` 監聽 `#main` 變化，每次切換路線時重新套用浮水印。

---

## 三、攝影機資料

### 3.1 中正山 - 格致路凱旋路 (BOT373)
- 來源：台北市政府工務局新建工程處
- URL：`https://livecam.tw/cam/BOT373/`
- 用於 zhzheng 路線的第 2 個 seg

### 3.2 西進武嶺攝影機更新

| Seg | 名稱 | 攝影機 ID | URL |
|-----|------|-----------|-----|
| 鳶峰 | (已移除) | - | - |
| 昆陽 | 合歡山昆陽停車場 | CCTV-26-014A-029-001 | https://livecam.tw/cam/CCTV-26-014A-029-001/ |
| 武嶺牌樓 | 合歡山-武嶺亭 | CCTV-26-014A-031-001 | https://livecam.tw/cam/CCTV-26-014A-031-001/ |

---

## 四、PWA 與分享連結

### 4.1 分享連結引導頁 (`add-to-home.html`)

**用途**：當使用者要分享給朋友時，引導對方加入手機主畫面。

**功能設計**：
- 自動偵測作業系統 (iOS / Android) 顯示對應安裝步驟
- 提供分頁切換按鈕讓使用者手動切換
- 已從主畫面開啟者自動跳轉到主程式 (避免重複看到引導)
- 含「直接開啟網站」按鈕作為降級方案

**iOS Safari 安裝步驟**：
1. 使用 Safari 瀏覽器開啟本頁
2. 點下方中間的分享圖示
3. 向下滑動，點選「加到主畫面」
4. 右上角點「新增」，完成

**Android Chrome 安裝步驟**：
1. 使用 Chrome 瀏覽器開啟本頁
2. 點右上角 ⋮ (選單)
3. 點選「安裝應用程式」或「加入主畫面」
4. 確認名稱後點「安裝」，完成

**分享網址**：
```
https://chenchunkuang0111-lang.github.io/cycling-weather-taipei/add-to-home.html
```

**重要限制**：
> 沒有任何網站可以「強制」自動把圖示加到使用者的主畫面 — 這是 iOS 和 Android 的安全限制，只有使用者本人能在瀏覽器選單按「加入主畫面」。所以我們做的是引導頁面。

### 4.2 跳出 App 自動重新整理

**觸發條件**：
- 切換到其他 App 後再回來
- 離開時間 > 30 秒

**實作**：
```javascript
let lastHidden = 0;
const REFRESH_AFTER_MS = 30 * 1000;

document.addEventListener('visibilitychange', () => {
  if(document.visibilityState === 'hidden'){
    lastHidden = Date.now();
  } else if(document.visibilityState === 'visible'){
    if(lastHidden && (Date.now() - lastHidden) > REFRESH_AFTER_MS){
      location.reload();
    }
  }
});

window.addEventListener('pageshow', e => {
  if(e.persisted){ location.reload(); }
});
```

**設計考量**：
- 30 秒閾值避免短暫切換 (例如看通知) 就重整影響體驗
- 處理 iOS Safari bfcache 的 `pageshow` 事件

### 4.3 手機下拉重新整理 (Pull-to-Refresh)

**觸發條件**：
- 頁面已捲動到最上方 (`scrollY <= 0`)
- 手指往下拖曳超過 70px 後放開

**互動流程**：
1. 使用者下拉 → 出現指示條「下拉重新整理」
2. 拉超過閾值 → 文字變為「釋放重新整理」
3. 放開 → 顯示 spinner「整理中...」
4. 300ms 後執行 `location.reload()`

**指示條樣式**：
- 從畫面頂端滑入
- 綠色 spinner (`#4CAF50`)
- 黑色漸層背景 (避免影響下方內容)
