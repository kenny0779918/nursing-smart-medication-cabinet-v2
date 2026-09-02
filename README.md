# 護理部智慧藥櫃 Prototype

以 Vue 3＋Vite 製作的需求驗證用動態原型。第一階段聚焦「常備藥取藥」完整流程，不連接真實醫療資料、藥櫃硬體或正式資料庫。

## 頁面範圍

1. 模擬刷卡登入
2. 功能首頁與近期取藥紀錄
3. 常備藥搜尋與選擇
4. 取藥數量確認
5. 模擬藥盒開啟／關閉
6. 作業完成與紀錄建立

## 檔案分工

- `src/App.vue`：頁面流程、互動邏輯與畫面組合
- `src/components/SystemHeader.vue`：共用系統 Header 與三個狀態燈
- `src/data/medicines.js`：模擬藥品及取藥紀錄
- `src/styles.css`：全站樣式、卡片與響應式排版
- `src/main.js`：Vue 啟動入口

## 啟動方式

```bash
npm install
npm run dev
```

瀏覽器開啟終端機顯示的網址即可操作。

## StackBlitz 使用

將整個專案上傳至 StackBlitz，平台會依 `package.json` 安裝套件並啟動 Vite。亦可先放入 GitHub，再於 StackBlitz 匯入該儲存庫。

> 本原型僅使用虛構資料。若未來公開部署，請勿放入病人、員工或正式藥品庫存等敏感資訊。
