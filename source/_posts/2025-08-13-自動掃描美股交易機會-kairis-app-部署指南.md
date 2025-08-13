---
title: 自動掃描美股交易機會 - Kairis App - 部署指南
date: 2025-08-13 18:50:32
subtitle: 自動掃描美股交易機會 - Kairis App - 部署指南
categories:
  - DevOps
tags:
  - deployment
  - fintech
  - guide
  - cloud
  - api
cover_index: https://images.unsplash.com/photo-1499750310107-5fef28a66643?w=450&h=450&fit=crop&crop=center
---

# Kairis App 完整部署指南

恭喜您！Kairis App 的核心功能已經全部開發完成。這份指南將引導您完成最後的部署步驟，將您的 App 發布到網路上，讓全世界都可以使用。

---

## 1. 部署概述

本指南旨在提供 Kairis App 的完整部署流程，目標是將應用程式成功發布到網際網路上，使其可供全球用戶存取。我們將採用一套完全免費且主流的服務組合，確保應用程式的長期穩定運行與高可用性。

**部署架構藍圖：**

*   **程式碼儲存庫 (Repository):** GitHub - 用於版本控制和程式碼管理。
*   **網站與後端服務 (Frontend & Backend):** Vercel - 提供靜態網站託管及無伺服器函式 (Serverless Functions) 支援。
*   **資料庫與使用者驗證 (Database & Authentication):** Firebase - 提供即時資料庫 (Firestore) 和多種驗證方式。
*   **外部數據 API 整合:**
    *   **Finnhub:** 用於獲取即時股票報價與新聞資訊。
    *   **Alpha Vantage:** 用於獲取歷史 K 線數據。
    *   **Gemini:** 用於 AI 分析與翻譯功能。

---

## 2. 環境準備

在開始部署之前，請務必完成以下事前準備工作，確保所有必要的帳號、金鑰和專案檔案都已到位。

### 2.1 帳號註冊與 API 金鑰收集

請確認您已註冊以下所有服務的帳號，並將您的 API 金鑰準備好，建議儲存在一個安全的記事本中備用：

*   **GitHub 帳號:**
    *   用途：存放您的應用程式程式碼。
    *   註冊連結：[前往 GitHub 註冊](https://github.com/join)
*   **Vercel 帳號:**
    *   用途：部署您的網站與後端 API。
    *   註冊連結：[前往 Vercel 註冊](https://vercel.com/signup) (建議使用 GitHub 帳號直接登入，方便後續匯入專案)
*   **Firebase 專案:**
    *   用途：提供資料庫 (Firestore) 和使用者驗證 (Authentication) 服務。
    *   *注意：Firebase 專案將在後續步驟中建立。*
*   **外部 API 金鑰:**
    *   `FINNHUB_API_KEY`: 從您的 [Finnhub 儀表板](https://finnhub.io/dashboard) 複製。
    *   `ALPHA_VANTAGE_API_KEY`: 從 [Alpha Vantage](https://www.alphavantage.co/support/#api-key) 申請並複製。
    *   `GEMINI_API_KEY`: 從 [Google AI Studio](https://aistudio.google.com/app/apikey) 建立並複製。

### 2.2 專案檔案結構確認

請確認您的 Kairis App 專案資料夾中，包含以下檔案與資料夾結構：

```
/ (專案根目錄)
├── index.html
├── package.json
├── manifest.json
├── sw.js
├── /api/
│   └── get-stock-data.js
└── /icons/
    ├── icon-192x192.png
    └── icon-512x512.png
```

**重要提示：** 在將專案上傳到 GitHub 之前，您需要先完成 `index.html` 檔案的修改，將 Firebase 設定程式碼整合進去。詳情請參閱後續的「設定 Firebase 專案」步驟。

---

## 3. 部署步驟

本節將詳細說明 Kairis App 的部署流程，請依照以下步驟依序操作。

### 3.1 準備您的專案檔案並上傳至 GitHub

1.  **整合 Firebase 設定至 `index.html`：**
    *   在完成 Firebase 專案建立並註冊網頁應用程式後，Firebase 會提供一段 `firebaseConfig` 物件的程式碼。
    *   請務必將這段程式碼複製，並貼到您本地專案的 `index.html` 檔案中，尋找並替換 `// *** START: Firebase Integration ***` 區塊。

    ```html
    <!-- index.html (部分內容) -->
    <!DOCTYPE html>
    <html lang="zh-Hant">
    <head>
        <!-- ... 其他 head 內容 ... -->
    </head>
    <body>
        <!-- ... 其他 body 內容 ... -->

        <!-- Firebase SDK -->
        <script type="module">
            // *** START: Firebase Integration ***
            // 將您從 Firebase 複製的 firebaseConfig 物件貼到這裡
            const firebaseConfig = {
                apiKey: "YOUR_API_KEY",
                authDomain: "YOUR_AUTH_DOMAIN",
                projectId: "YOUR_PROJECT_ID",
                storageBucket: "YOUR_STORAGE_BUCKET",
                messagingSenderId: "YOUR_MESSAGING_SENDER_ID",
                appId: "YOUR_APP_ID"
            };

            // 初始化 Firebase
            import { initializeApp } from "https://www.gstatic.com/firebasejs/9.6.1/firebase-app.js";
            import { getAuth } from "https://www.gstatic.com/firebasejs/9.6.1/firebase-auth.js";
            import { getFirestore } from "https://www.gstatic.com/firebasejs/9.6.1/firebase-firestore.js";

            const app = initializeApp(firebaseConfig);
            const auth = getAuth(app);
            const db = getFirestore(app);
            // *** END: Firebase Integration ***

            // ... 其他 JavaScript 程式碼 ...
        </script>
    </body>
    </html>
    ```

2.  **上傳專案至 GitHub 儲存庫：**
    *   確認 `index.html` 已更新並保存。
    *   將整個 Kairis App 專案資料夾初始化為 Git 儲存庫，並將其上傳到您在 GitHub 建立的遠端儲存庫中。

    ```bash
    # 在專案根目錄執行
    git init
    git add .
    git commit -m "Initial commit: Kairis App project files"
    git branch -M main
    git remote add origin https://github.com/您的GitHub帳號/您的儲存庫名稱.git
    git push -u origin main
    ```

### 3.2 設定 Firebase 專案

1.  **建立 Firebase 專案：**
    *   前往 [Firebase 控制台](https://console.firebase.google.com/)。
    *   點擊「新增專案」。
    *   為專案命名 (例如 `kairis-app`)，並依照指示完成建立。您可以選擇關閉 Google Analytics 以簡化設定。

2.  **註冊網頁應用程式並整合 Firebase 設定：**
    *   在您的 Firebase 專案主控台頁面，點擊網頁圖示 (`</>`)。
    *   為您的應用程式取一個暱稱 (例如 `Kairis Web`)，然後點擊「註冊應用程式」。
    *   **重要：** Firebase 會顯示 `firebaseConfig` 物件程式碼。請立即複製這段程式碼，並貼到您本地的 `index.html` 檔案中對應的 `// *** START: Firebase Integration ***` 區塊（如 3.1 步驟所示）。完成後，請確保將修改後的 `index.html` 重新上傳到 GitHub。

    ```bash
    # 在本地專案目錄，將修改後的 index.html 推送到 GitHub
    git add index.html
    git commit -m "Update index.html with Firebase config"
    git push origin main
    ```

3.  **啟用登入方式 (Authentication)：**
    *   在 Firebase 控制台的左側選單，選擇「Authentication」。
    *   點擊「開始使用」，然後切換到「Sign-in method」分頁。
    *   在服務供應商列表中，找到並點擊「Google」，將其啟用，並設定您的專案支援電子郵件，然後儲存。
    *   同樣地，找到並啟用「匿名 (Anonymous)」登入方式。

4.  **建立並設定 Firestore 資料庫：**
    *   在 Firebase 控制台的左側選單，選擇「Firestore Database」。
    *   點擊「建立資料庫」，選擇「以正式版模式啟動」，並選擇一個離您最近的地區 (例如 `asia-east1`)。
    *   建立後，切換到「規則 (Rules)」分頁，將所有內容替換為以下規則，然後點擊「發布」：

    ```firestore
    rules_version = '2';
    service cloud.firestore {
      match /databases/{database}/documents {
        // 允許使用者讀寫自己的 /users/{userId} 文件
        match /users/{userId} {
          allow read, write: if request.auth != null && request.auth.uid == userId;
        }
        // 預設拒絕其他所有文件的讀寫，除非明確允許
        match /{document=**} {
          allow read, write: if false;
        }
      }
    }
    ```

### 3.3 部署到 Vercel

1.  **匯入專案：**
    *   登入 [Vercel](https://vercel.com/)。
    *   點擊儀表板上的「Add New...」 -> 「Project」。
    *   選擇您存放 Kairis 專案的 GitHub 儲存庫，並點擊「Import」。

2.  **設定 Vercel KV (快取資料庫)：**
    *   在專案設定頁面，找到「Storage」分頁。
    *   在「Create a database」區塊中，找到 Upstash for Redis 旁邊的「Create」。
    *   選擇 `Hobby (Free)` 方案，選擇一個離您最近的地區，然後完成建立與連結。Vercel 會自動為您設定所有 KV 相關的環境變數。

3.  **設定 API 金鑰 (環境變數)：**
    *   回到專案設定的「Settings」 -> 「Environment Variables」。
    *   新增以下三個環境變數，將您收集到的金鑰貼入對應的 `Value` 欄位：
        *   `Key`: `FINNHUB_API_KEY`, `Value`: (貼上您的 Finnhub 金鑰)
        *   `Key`: `ALPHA_VANTAGE_API_KEY`, `Value`: (貼上您的 Alpha Vantage 金鑰)
        *   `Key`: `GEMINI_API_KEY`, `Value`: (貼上您的 Gemini 金鑰)

4.  **觸發部署：**
    *   回到「Deployments」分頁，找到最新的部署紀錄。
    *   點擊該紀錄右邊的三個點 `...` -> 「Redeploy」來觸發一次新的部署，確保所有新設定 (Vercel KV 和環境變數) 都生效。

5.  **設定 Firebase 授權網域 (關鍵步驟)：**
    *   部署成功後，Vercel 會給您一個公開的網址 (例如 `https://kairis.vercel.app`)。
    *   請將這個網址複製起來。
    *   回到 Firebase 控制台的「Authentication」 -> 「Settings」 -> 「已授權的網域」。
    *   點擊「新增網域」並將您從 Vercel 複製的網址貼上後新增。
    *   **重要：** 此步驟是確保 Firebase 驗證服務能在您的 Vercel 應用程式上正常運作的關鍵安全設定。

---

## 4. 驗證測試

在完成所有部署步驟後，請務必進行以下驗證測試，確保您的 Kairis App 正常運作：

1.  **應用程式訪問：**
    *   在瀏覽器中輸入您的 Vercel 部署網址 (例如 `https://kairis.vercel.app`)，確認應用程式頁面能正常載入。

2.  **使用者登入測試：**
    *   嘗試使用「Google 登入」功能，確認是否能成功透過 Google 帳號登入。
    *   嘗試使用「匿名登入」功能，確認是否能成功匿名登入。
    *   登入後，檢查您的 Firebase Firestore 資料庫中是否生成了對應的使用者文件 (`/users/{userId}`)。

3.  **數據顯示與 API 整合測試：**
    *   在應用程式中搜尋股票或查看報價，確認 Finnhub 提供的即時報價和新聞是否正常顯示。
    *   檢查歷史 K 線圖是否能正常載入，確認 Alpha Vantage 數據整合無誤。
    *   如果您的應用程式有 AI 分析或翻譯功能，請嘗試使用，確認 Gemini API 是否能正常響應。

4.  **瀏覽器開發者工具檢查：**
    *   開啟瀏覽器的開發者工具 (通常是 `F12`)，切換到「Console」分頁，檢查是否有任何錯誤訊息。
    *   切換到「Network」分頁，監控 API 請求是否成功 (HTTP 狀態碼 200)。

5.  **Vercel 與 Firebase 日誌檢查：**
    *   登入 Vercel 儀表板，進入您的專案，查看「Logs」分頁，檢查是否有任何部署或運行時錯誤。
    *   登入 Firebase 控制台，查看「Functions」（如果您的 `/api` 有使用 Firebase Functions）或「Usage」頁面，監控服務使用情況和潛在錯誤。

---

## 5. 常見問題排解

在部署過程中，您可能會遇到一些常見問題。以下是針對這些問題的排解建議：

*   **問題：應用程式無法載入或顯示空白頁面。**
    *   **解決方案：**
        *   檢查您的 GitHub 儲存庫中是否包含所有必要的檔案 (`index.html`, `package.json`, `manifest.json`, `sw.js`, `/api` 資料夾等)。
        *   確認 `index.html` 中的所有路徑 (例如 CSS、JavaScript 檔案的路徑) 是否正確。
        *   檢查 Vercel 部署日誌，查看是否有建置失敗的錯誤訊息。

*   **問題：API 數據無法載入或顯示錯誤。**
    *   **解決方案：**
        *   確認您在 Vercel 環境變數中設定的 `FINNHUB_API_KEY`, `ALPHA_VANTAGE_API_KEY`, `GEMINI_API_KEY` 是否正確且沒有多餘的空格。
        *   檢查您的 API 金鑰是否有效，有時 API 服務商會更新金鑰或有使用限制。
        *   檢查 Vercel 部署的 `/api/get-stock-data.js` 函式日誌，看是否有後端錯誤。

*   **問題：Firebase 登入失敗，顯示「Auth/unauthorized-domain」錯誤。**
    *   **解決方案：**
        *   這是最常見的 Firebase 驗證問題。請確認您已將 Vercel 部署的公開網址 (例如 `https://kairis.vercel.app`) 添加到 Firebase 控制台的「Authentication」->「Settings」->「已授權的網域」列表中。務必包含 `https://` 前綴。

*   **問題：Firestore 資料庫操作失敗，顯示權限錯誤。**
    *   **解決方案：**
        *   請仔細檢查您在 Firebase Firestore「規則 (Rules)」中設定的權限。確保 `request.auth != null && request.auth.uid == userId` 邏輯正確，並且規則已成功「發布」。任何語法錯誤或未發布的規則都可能導致問題。

*   **問題：Vercel 部署失敗。**
    *   **解決方案：**
        *   檢查 Vercel 部署日誌，通常會明確指出失敗的原因，例如缺少依賴項、程式碼錯誤或配置問題。
        *   確認 `package.json` 中的依賴項已正確列出，並且 `npm install` (Vercel 會自動執行) 不會出錯。

---

## 6. 維護和監控

部署完成後，持續的維護和監控對於確保應用程式的穩定運行至關重要。

*   **持續部署 (CI/CD)：**
    *   由於您使用 GitHub 和 Vercel，您的應用程式已自動配置為持續部署。每次您將新的程式碼推送到 GitHub 儲存庫的 `main` 分支時，Vercel 都會自動觸發一次新的部署。這大大簡化了更新和發布的流程。

*   **監控應用程式性能與使用情況：**
    *   **Vercel 儀表板：** 定期檢查 Vercel 專案的「Usage」和「Logs」分頁，監控流量、函式執行時間和任何潛在的錯誤日誌。
    *   **Firebase 控制台：**
        *   **Authentication：** 監控使用者註冊和登入趨勢。
        *   **Firestore Database：** 查看讀寫操作次數、儲存空間使用量和規則評估結果，以確保資料庫健康。
        *   **Usage and Billing：** 雖然我們使用免費服務，但了解各項服務的使用量可以幫助您預防超出免費額度。

*   **API 金鑰管理：**
    *   如果您的外部 API 金鑰（Finnhub, Alpha Vantage, Gemini）需要更新或重新生成，請務必回到 Vercel 專案的「Settings」->「Environment Variables」中進行更新，然後觸發一次新的部署以使更改生效。

*   **安全更新：**
    *   定期檢查您的專案依賴項（`package.json`）是否有安全漏洞更新。雖然 Vercel 會自動處理大部分運行時環境，但保持程式碼本身的依賴項最新也是良好的實踐。

*   **備份策略 (針對資料庫)：**
    *   Firebase Firestore 提供了自動備份和恢復功能。對於生產環境，您可以考慮配置定期導出資料到 Cloud Storage，以提供額外的資料保護層。

透過上述的部署指南，您的 Kairis App 將能夠穩定、高效地運行，並為全球用戶提供服務。祝您的應用程式一切順利！