# 股票投資 PWA + App Store 打包指南

## 為什麼需要 wrapper?

Google Apps Script Web App 的 HTML 跑在 iframe sandbox 內,瀏覽器**不允許**在
iframe 裡註冊 Service Worker / 應用 manifest。所以要做 PWA,必須有一個**外部
殼頁面**(`index.html`) 把 GAS Web App URL 嵌進 iframe。

## ━━━━━━━━━━━━━━━━━━━━━━━━━━
## Part A: PWA (今天就能用 · 免費)
## ━━━━━━━━━━━━━━━━━━━━━━━━━━

### 步驟 1 — 準備 App Icons

需要 3 個 png:
- `icon-512.png` (512×512) — Android / PWA install
- `icon-192.png` (192×192) — PWA install fallback
- `icon-180.png` (180×180) — iOS apple-touch-icon

**最快做法**:用 [favicon.io](https://favicon.io/favicon-generator/) 輸入 "股票"
兩字 + 綠底 → 下載 → 改名放進此資料夾。

或用 emoji icon generator: <https://maskable.app/editor>

### 步驟 2 — 把 GAS URL 填進 index.html

打開 `index.html` 找到這行:
```html
src="GAS_URL_PLACEHOLDER"
```

替換成你的 Apps Script Web App 部署網址(類似):
```
https://script.google.com/macros/s/AKfycby.../exec
```

### 步驟 3 — 託管到 HTTPS (三選一)

PWA 必須 HTTPS。免費選項:

| 選項 | 步驟 | 速度 |
|---|---|---|
| **GitHub Pages** | 建 repo `stock-pwa` → 上傳這個資料夾 → Settings → Pages → main branch → 等 1 分鐘 | 推薦 |
| **Cloudflare Pages** | 連 GitHub repo → 自動部署 → 拿 `xxx.pages.dev` URL | 最快 |
| **Netlify Drop** | 拖整個資料夾到 <https://app.netlify.com/drop> | 30 秒上線 |

最終你會得到一個網址,例如:
```
https://xavierchow61.github.io/stock-pwa/
```

### 步驟 4 — 手機「加到主畫面」

**iPhone (Safari)**:
1. 打開上面那個網址
2. 點分享 ⎘ → 「加到主畫面」
3. 桌面出現 app icon · 點下去全螢幕、無瀏覽器外框

**Android (Chrome)**:
1. 打開網址 → 自動彈出「安裝」提示
2. 或點右上 ⋮ → 「安裝應用程式」

完成! 這就是「URL 變成 app」。

### Apple Smart App Banner (可選)
若日後上架 App Store,可在 `<head>` 加:
```html
<meta name="apple-itunes-app" content="app-id=YOUR_APP_ID">
```

---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━
## Part B: 上架 Apple App Store
## ━━━━━━━━━━━━━━━━━━━━━━━━━━

### ⚠ 重要警告
Apple 審核準則 **4.2 Minimum Functionality** 明文反對「純粹 WebView 包裝」的 app。
直接把 GAS URL 包成 native app **很大機率被拒**。

要過審,必須加 **native 功能**(例如):
- ✅ Face ID / Touch ID 登入
- ✅ Push Notification(到價提示)
- ✅ Widget(主畫面小工具)
- ✅ 本地 Database(離線交易紀錄)
- ✅ 相機(QR Code 掃描券商對帳單)

### 前置需求
1. **Apple Developer Program**: $99/年 — <https://developer.apple.com/programs/>
2. **Mac 電腦**(必須,Xcode 只能在 macOS 跑)
3. **Xcode** (從 Mac App Store 免費下載,~15GB)
4. **Node.js + npm**

### 工具選擇

**推薦 Capacitor** (Ionic 出的,2024 最主流的 hybrid wrapper):

```bash
# 一、初始化專案
npm create @capacitor/app stock-app
cd stock-app
npm install
npm install @capacitor/ios
npx cap add ios

# 二、把 pwa-wrapper/ 內容複製到 www/ 資料夾
cp -r ../pwa-wrapper/* www/

# 三、Capacitor 設定 (capacitor.config.json)
# 改 server.url 為 GAS 網址 (跳過本地 www, 直接載入線上)
# 或保持 webDir: 'www' 走本地 wrapper

# 四、加 native 功能 (過審必備)
npm install @capacitor/push-notifications
npm install @capacitor/local-notifications
npm install @capacitor-community/face-id
npm install @capacitor/preferences  # 本地儲存

# 五、同步 + 開 Xcode
npx cap sync ios
npx cap open ios
```

### 在 Xcode 內

1. 選你的 Team(綁定 Developer Account)
2. Bundle Identifier: `com.xavierchow.stockinvest`
3. 簽好 Provisioning Profile
4. 模擬器跑一次 → 確認 GAS URL 能載入
5. Product → Archive → Upload to App Store Connect

### 在 App Store Connect

1. <https://appstoreconnect.apple.com> → 新增 app
2. 填寫:
   - App Name: 股票投資儀表板
   - Subtitle: AI 持倉 / 訊號 / 回測
   - Category: Finance
   - Screenshots(6.5" iPhone 必須): 5 張以上
   - Privacy Policy URL (必填,可放在 GitHub Pages 同層)
3. 填寫 App Review Information:
   - 提供測試帳號(讓 reviewer 能登入)
   - Notes: 強調 native 功能(Push、Face ID、Widget)
4. 送審 → 等 24-48 小時

### 過審秘訣
- **截圖必須展示 native 功能** (Face ID 解鎖 / Push 通知 / Widget)
- **不要在描述提「WebView」、「網頁」**
- **首次開啟必須顯示 native UI** (例如登入畫面用 native iOS 元件,不要直接跳 GAS)
- 若被拒,改稿後可立即重送、不必再付費

### 預期時程
- 第一次提交到上架: **1-3 週**(含被拒重提)
- 純 webview wrapper 不加 native: **幾乎一定被拒**
- 認真做 native 功能: **約 1-2 個月開發**

---

## ━━━━━━━━━━━━━━━━━━━━━━━━━━
## 建議路線
## ━━━━━━━━━━━━━━━━━━━━━━━━━━

1. **第 1 週**: 跑 Part A → PWA 上線、家人朋友先用
2. **第 1 個月**: 蒐集回饋、優化 UI
3. **第 2 個月**: 開始 Capacitor + 加 native 功能
4. **第 3 個月**: 送 App Store 審核

純 PWA 已能達成 95% 的「手機 app 體驗」,App Store 上架是流量曝光的加分項。
