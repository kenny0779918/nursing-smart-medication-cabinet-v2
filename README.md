# 護理部智慧藥櫃 Prototype

以 Vue 3＋Vite 製作的需求驗證用動態原型。目前聚焦「Barcode登入、共用Header、角色權限首頁」，不連接真實醫療資料、藥櫃硬體或正式資料庫。

## 頁面範圍

1. Barcode輸入及模擬刷卡登入
2. 護理人員／護理主管角色權限
3. 無權限提示及3秒自動返回
4. 共用Header及5項設備狀態
5. 8項功能首頁及無權限反灰
6. 登入後閒置5分鐘自動登出

測試條碼：護理人員 `N001162`；護理主管 `S000888`；其他條碼會顯示無權限。

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
