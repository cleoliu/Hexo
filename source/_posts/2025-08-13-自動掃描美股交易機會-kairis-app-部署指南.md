---
title: 自動掃描美股交易機會 - Kairis App - 部署指南
date: 2025-08-13 18:50:32
subtitle: 自動掃描美股交易機會 - Kairis App - 部署指南
categories:
  - DevOps
tags:
  - guide
  - deployment
  - cloud
  - fintech
  - api
cover_index: https://images.unsplash.com/photo-1499750310107-5fef28a66643?w=450&h=450&fit=crop&crop=center
---

# Kairis App 最終部署指南

恭喜您！Kairis App 的核心功能已經全部開發完成。這份指南將引導您完成最後的部署步驟，將您的 App 發布到網路上，讓全世界都可以使用。

## 1. 部署概述

本指南旨在提供 Kairis App 的完整部署流程，目標是將應用程式發布到網際網路，使其可供公眾存取。我們將採用一套完全免費且主流的服務組合，以確保您的 App 能夠長期穩定且高效地運作。

**部署架構：**

*   **程式碼儲存庫 (Code Repository):** GitHub
    *   用於版本控制和程式碼託管。
*   **網站與後端服務 (Website & Backend Service):** Vercel
    *   提供前端靜態網站託管及後端 Serverless Functions (API)。
*   **資料庫與使用者認證 (Database & User Authentication):** Firebase (Firestore & Authentication)
    *   提供 NoSQL 雲端資料庫和多種認證方式。
*   **外部數據 API (External Data APIs):**
    *   **Finnhub:** 用於即時股票報價與新聞資訊。
    *   **Alpha Vantage:** 用於歷史 K 線數據。
    *   **Gemini (Google AI Studio):** 用於 AI 分析與翻譯功能。

## 2. 環境準備

在開始部署之前，請確保您已完成以下準備工作，並收集所有必要的金鑰與帳號資訊。

### 2.1 系統需求

*   現代網頁瀏覽器 (用於存取各服務控制台)。
*   基本的 Git 知識 (用於將程式碼上傳至 GitHub)。

### 2.2 帳號與金鑰收集

請確認您已註冊以下所有服務的帳號，並將您的 API 金鑰準備好，建議儲存於安全的位置以備後續使用：

*   **GitHub 帳號:**
    *   前往 [GitHub 註冊頁面](https://github.com/join) 註冊。
*   **Vercel 帳號:**
    *   前往 [Vercel 註冊頁面](https://vercel.com/signup) 註冊 (建議使用 GitHub 帳號直接登入)。
*   **Firebase 專案:**
    *   我們將在部署步驟中建立。
*   **API 金鑰:**
    *   `FINNHUB_API_KEY`: 從您的 [Finnhub 儀表板](https://finnhub.io/dashboard) 複製。
    *   `ALPHA_VANTAGE_API_KEY`: 從 [Alpha Vantage 網站](https://www.alphavantage.co/support/#api-key) 申請並複製。
    *   `GEMINI_API_KEY`: 從 [Google AI Studio](https://aistudio.google.com/app/apikey) 建立並複製。

### 2.3 專案檔案結構確認

請確保您的 Kairis App 專案資料夾中，包含以下檔案與資料夾結構：

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

## 3. 部署步驟

本節將詳細說明 Kairis App 的部署流程，請依照順序執行。

### 3.1 上傳專案至 GitHub

確認您的專案檔案結構無誤後，請將整個專案資料夾上傳到您在 GitHub 建立的儲存庫 (Repository) 中。

**操作步驟：**

1.  在 GitHub 上建立一個新的儲存庫 (例如 `kairis-app`)。
2.  初始化本地專案為 Git 儲存庫：
    ```bash
    cd /path/to/your/kairis-app
    git init
    git add .
    git commit -m "Initial commit of Kairis App"
    ```
3.  將本地儲存庫連結到 GitHub 遠端儲存庫並推送：
    ```bash
    git remote add origin https://github.com/YOUR_GITHUB_USERNAME/YOUR_REPO_NAME.git
    git branch -M main
    git push -u origin main
    ```
    *   請將 `YOUR_GITHUB_USERNAME` 和 `YOUR_REPO_NAME` 替換為您的實際 GitHub 用戶名和儲存庫名稱。

### 3.2 設定 Firebase

Firebase 將用於處理使用者認證和資料庫儲存。

**操作步驟：**

1.  **建立 Firebase 專案:**
    *   前往 [Firebase 控制台](https://console.firebase.google.com/)。
    *   點擊「新增專案」，為專案命名 (例如 `kairis-app`)，並依照指示完成建立。您可以選擇關閉 Google Analytics 以簡化設定。

2.  **註冊網頁應用程式並整合 `firebaseConfig`:**
    *   在 Firebase 專案主控台，點擊網頁圖示 (`</>`)。
    *   為您的應用程式取一個暱稱 (例如 `Kairis Web`)，然後點擊「註冊應用程式」。
    *   **重要：** Firebase 會顯示 `firebaseConfig` 物件，請立即複製這段程式碼。
    *   開啟您本地的 `index.html` 檔案，找到 `// *** START: Firebase Integration ***` 區塊，將複製的 `firebaseConfig` 程式碼貼入其中。
    *   **範例 `index.html` 整合區塊：**
        ```html
        <!DOCTYPE html>
        <html lang="zh-Hant">
        <head>
            <!-- ... 其他 head 內容 ... -->
        </head>
        <body>
            <!-- ... 其他 body 內容 ... -->

            <!-- Firebase SDK 引入 -->
            <script src="https://www.gstatic.com/firebasejs/9.x.x/firebase-app-compat.js"></script>
            <script src="https://www.gstatic.com/firebasejs/9.x.x/firebase-auth-compat.js"></script>
            <script src="https://www.gstatic.com/firebasejs/9.x.x/firebase-firestore-compat.js"></script>

            <script>
                // *** START: Firebase Integration ***
                // 請將您從 Firebase 控制台複製的 firebaseConfig 物件貼到這裡
                const firebaseConfig = {
                    apiKey: "YOUR_API_KEY",
                    authDomain: "YOUR_AUTH_DOMAIN",
                    projectId: "YOUR_PROJECT_ID",
                    storageBucket: "YOUR_STORAGE_BUCKET",
                    messagingSenderId: "YOUR_MESSAGING_SENDER_ID",
                    appId: "YOUR_APP_ID",
                    measurementId: "YOUR_MEASUREMENT_ID" // 如果啟用 Google Analytics 會有
                };

                // 初始化 Firebase
                const app = firebase.initializeApp(firebaseConfig);
                const auth = firebase.auth();
                const db = firebase.firestore();
                // *** END: Firebase Integration ***

                // ... 其他 JavaScript 程式碼 ...
            </script>
        </body>
        </html>
        ```
    *   完成 `index.html` 修改後，請將更新後的檔案再次上傳到 GitHub 儲存庫：
        ```bash
        git add index.html
        git commit -m "Integrate Firebase config into index.html"
        git push origin main
        ```

3.  **啟用登入方式 (Authentication):**
    *   在 Firebase 控制台左側選單選擇「Authentication」。
    *   點擊「開始使用」，然後切換到「Sign-in method」分頁。
    *   在服務供應商列表中，找到並點擊「Google」，將其啟用，並設定您的專案支援電子郵件，然後儲存。
    *   同樣地，找到並啟用「匿名」登入方式。

4.  **建立並設定 Firestore 資料庫 (Database):**
    *   在 Firebase 控制台左側選單選擇「Firestore Database」。
    *   點擊「建立資料庫」，選擇「以正式版模式啟動」，並選擇一個離您最近的地區 (例如 `asia-east1`)。
    *   建立後，切換到「規則 (Rules)」分頁，將所有內容替換為以下規則，然後點擊「發布」：
        ```firestore
        rules_version = '2';
        service cloud.firestore {
          match /databases/{database}/documents {
            // 允許已驗證的使用者讀寫其自己的 /users/{userId} 文件
            match /users/{userId} {
              allow read, write: if request.auth != null && request.auth.uid == userId;
            }
          }
        }
        ```
        *   **說明：** 這些規則確保只有已登入的使用者才能讀取和寫入其在 `/users/{userId}` 路徑下的專屬資料。

### 3.3 部署到 Vercel

Vercel 將負責託管您的前端應用程式和後端 API 服務。

**操作步驟：**

1.  **匯入專案:**
    *   登入 [Vercel](https://vercel.com/)，點擊「Add New...」 -> 「Project」。
    *   選擇您存放 Kairis 專案的 GitHub 儲存庫，並點擊「Import」。Vercel 會自動檢測到您的專案類型。

2.  **設定 Vercel KV (快取資料庫):**
    *   在專案設定頁面，找到「Storage」分頁。
    *   在「Create a database」區塊中，找到 Upstash for Redis 旁邊的「Create」。
    *   選擇 `Hobby (Free)` 方案，選擇一個離您最近的地區，然後完成建立與連結。Vercel 會自動幫您設定好所有 KV 相關的環境變數。

3.  **設定 API 金鑰 (環境變數):**
    *   回到專案設定的「Settings」 -> 「Environment Variables」。
    *   新增以下三個環境變數，將您在 `2.2 帳號與金鑰收集` 步驟中準備好的金鑰貼入：
        *   **Key:** `FINNHUB_API_KEY`, **Value:** `(貼上您的 Finnhub 金鑰)`
        *   **Key:** `ALPHA_VANTAGE_API_KEY`, **Value:** `(貼上您的 Alpha Vantage 金鑰)`
        *   **Key:** `GEMINI_API_KEY`, **Value:** `(貼上您的 Gemini 金鑰)`
    *   **重要：** 確保沒有多餘的空格或其他字元。

4.  **觸發部署！:**
    *   回到「Deployments」分頁，找到最新的部署紀錄，點擊它右邊的三個點 `...` -> 「Redeploy」來觸發一次新的部署，確保所有新設定 (尤其是環境變數) 都生效。
    *   Vercel 會自動開始建置和部署您的應用程式。

5.  **設定 Firebase 授權網域 (關鍵步驟):**
    *   部署成功後，Vercel 會給您一個公開的網址 (例如 `https://kairis.vercel.app`)。
    *   請將這個網址複製起來，回到 Firebase 控制台的 `Authentication` -> `Settings` -> `已授權的網域`。
    *   點擊「新增網域」並將複製的 Vercel 網址貼上後新增。
    *   **說明：** 這是為了讓 Firebase 知道您的應用程式在哪個網域下運行，從而允許來自該網域的認證請求。

恭喜您！完成以上所有步驟後，您的 Kairis App 就正式上線了！

## 4. 驗證測試

部署完成後，請務必進行以下驗證測試，確保應用程式的各項功能正常運作。

1.  **應用程式載入：**
    *   在瀏覽器中開啟您的 Vercel 部署網址 (例如 `https://kairis.vercel.app`)。
    *   確認應用程式介面正常載入，沒有顯示錯誤訊息。

2.  **使用者認證：**
    *   嘗試使用 Google 帳號登入。
    *   嘗試使用匿名方式登入。
    *   確認登入/登出功能正常，且使用者狀態正確更新。

3.  **數據 API 整合：**
    *   測試股票報價功能 (Finnhub)。
    *   測試歷史 K 線圖功能 (Alpha Vantage)。
    *   測試 AI 分析或翻譯功能 (Gemini)。
    *   確認數據能夠正確顯示，沒有 API 錯誤。

4.  **Firestore 資料庫操作：**
    *   如果您的應用程式有儲存使用者偏好或收藏股票的功能，請測試這些資料是否能夠正確讀取和寫入 Firestore。
    *   在 Firebase 控制台的 Firestore Database 中，確認資料是否正確更新。

5.  **Vercel 日誌檢查：**
    *   在 Vercel 專案儀表板中，查看「Logs」分頁。
    *   檢查是否有任何錯誤或警告訊息，特別是與 `api/get-stock-data.js` 相關的 Serverless Function 執行日誌。

## 5. 常見問題排解

在部署過程中，您可能會遇到以下一些常見問題及其解決方案：

*   **問題：應用程式無法載入或顯示空白頁面。**
    *   **解決方案：**
        *   檢查瀏覽器的開發者工具 (F12)，查看 Console 和 Network 標籤是否有錯誤訊息。
        *   確認 `index.html` 中的 `firebaseConfig` 是否正確複製貼上，且沒有語法錯誤。
        *   檢查 Vercel 部署日誌，看是否有建置失敗的錯誤。

*   **問題：Firebase 登入失敗，顯示「auth/unauthorized-domain」錯誤。**
    *   **解決方案：**
        *   這是最常見的問題之一。請務必回到 Firebase 控制台的 `Authentication` -> `Settings` -> `已授權的網域`，將您的 Vercel 部署網址 (例如 `kairis.vercel.app`) 加入白名單。

*   **問題：無法取得股票數據或 AI 分析結果。**
    *   **解決方案：**
        *   **API 金鑰問題：**
            *   在 Vercel 專案設定的 `Environment Variables` 中，仔細檢查 `FINNHUB_API_KEY`、`ALPHA_VANTAGE_API_KEY` 和 `GEMINI_API_KEY` 的值是否正確，沒有多餘的空格或換行符。
            *   確認這些金鑰本身是有效的，可以在各自的服務提供商儀表板上進行測試。
        *   **Vercel Serverless Function 問題：**
            *   檢查 Vercel 部署日誌，特別是 `/api/get-stock-data.js` 函式的執行日誌，看是否有錯誤訊息。
            *   確認 `get-stock-data.js` 檔案位於 `api/` 資料夾下，且導出方式符合 Vercel Serverless Function 的要求。

*   **問題：Firestore 資料無法讀寫，顯示「permission denied」錯誤。**
    *   **解決方案：**
        *   回到 Firebase 控制台的 `Firestore Database` -> `規則 (Rules)` 分頁。
        *   確認您的安全規則是否正確設定，特別是 `allow read, write: if request.auth != null && request.auth.uid == userId;` 這一行，確保它正確限制了存取權限。
        *   如果您正在嘗試讀寫 `users/{userId}` 以外的路徑，但沒有設定相應的規則，也會導致此錯誤。

*   **問題：Vercel 部署失敗。**
    *   **解決方案：**
        *   查看 Vercel 部署日誌，它會提供詳細的錯誤信息，例如缺少依賴、程式碼語法錯誤或配置問題。
        *   確保 `package.json` 檔案存在且格式正確。
        *   如果使用了 `npm` 或 `yarn` 依賴，確保這些依賴在 `package.json` 中有列出。

## 6. 維護和監控

應用程式上線後，持續的維護和監控對於確保其穩定運行至關重要。

*   **持續監控：**
    *   **Vercel 儀表板：** 定期查看 Vercel 專案儀表板，監控部署狀態、流量、Serverless Function 執行時間和錯誤日誌。
    *   **Firebase 控制台：** 監控 Authentication 使用情況、Firestore 讀寫操作和儲存量，以及任何潛在的錯誤報告。
    *   **API 提供商儀表板：** 監控 Finnhub, Alpha Vantage, Gemini 的 API 使用量，確保沒有超出免費額度或速率限制。

*   **更新與升級：**
    *   **程式碼更新：** 當您對 Kairis App 進行功能更新或錯誤修復時，只需將更改推送到 GitHub 儲存庫的 `main` 分支，Vercel 會自動觸發新的部署。
    *   **依賴更新：** 定期檢查 `package.json` 中的依賴項是否有新版本，並考慮升級以獲得性能改進和安全補丁。
    *   **Firebase SDK：** 留意 Firebase SDK 的更新，並根據需要升級應用程式中的 SDK 版本。

*   **安全性考量：**
    *   **Firebase 規則：** 定期審查您的 Firestore 安全規則，確保它們仍然符合您的應用程式需求，並防止未經授權的資料存取。
    *   **API 金鑰：** 確保您的 API 金鑰安全儲存在 Vercel 環境變數中，切勿將其硬編碼在公開的程式碼中。如果金鑰洩露，請立即更換。

*   **備份：**
    *   對於 Firestore 資料，Firebase 提供了自動備份和匯出功能。對於小型應用程式，通常不需要手動備份，但對於關鍵數據，建議了解其備份機制。

遵循本指南，您將能夠成功部署 Kairis App，並在未來有效地進行維護和監控。