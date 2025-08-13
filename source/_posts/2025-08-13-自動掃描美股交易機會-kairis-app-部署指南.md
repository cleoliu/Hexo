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

cover_index: https://res.cloudinary.com/dsvl326mi/image/upload/v1755082231/blog_covers/icon-192x192_jlih3e.png
---

# Kairis App：完整部署指南

恭喜您！Kairis App 的核心功能已經全部開發完成。這份指南將引導您完成最後的部署步驟，將您的 App 發布到網路上，讓全世界都可以使用。

---

## 1. 部署概述

本指南旨在提供 Kairis App 的完整部署流程，確保應用程式能夠穩定、高效地在線上運行。我們將採用一套完全免費且主流的服務組合，以實現長期穩定運作的目標。

**部署目標：**

*   將 Kairis App 的核心功能部署至公開網路。
*   提供穩定的服務，確保全球使用者可隨時存取。
*   利用免費且主流的雲端服務，降低營運成本。

**部署架構：**

*   **程式碼儲存庫 (Code Repository):** GitHub
    *   用於版本控制和專案協作。
*   **網站與後端服務 (Website & Backend Services):** Vercel
    *   提供前端靜態網站託管及無伺服器 API 功能。
    *   整合 Vercel KV (基於 Upstash Redis) 作為快取資料庫。
*   **資料庫與使用者認證 (Database & User Authentication):** Firebase (Google Cloud)
    *   Firestore 提供 NoSQL 資料庫服務。
    *   Authentication 提供多種登入方式（如 Google 登入、匿名登入）。
*   **外部數據 API (External Data APIs):**
    *   **Finnhub:** 提供即時股票報價與新聞資訊。
    *   **Alpha Vantage:** 提供歷史 K 線數據。
    *   **Gemini (Google AI Studio):** 提供 AI 分析與翻譯功能。

---

## 2. 環境準備

在開始部署之前，請確保您的開發環境和相關服務帳號已準備就緒。

### 2.1 系統需求

*   現代網路瀏覽器 (如 Chrome, Firefox, Edge)。
*   穩定的網路連線。
*   文字編輯器或 IDE (用於修改 `index.html` 檔案)。

### 2.2 依賴項目與帳號註冊

請務必提前註冊以下所有服務的帳號，並將相關的 API 金鑰妥善保存。

*   **GitHub 帳號:**
    *   用途：存放您的專案程式碼。
    *   註冊連結：[https://github.com/join](https://github.com/join)
*   **Vercel 帳號:**
    *   用途：部署您的網站和後端 API。
    *   註冊連結：[https://vercel.com/signup](https://vercel.com/signup) (建議直接使用 GitHub 帳號登入)
*   **Firebase 專案:**
    *   用途：提供資料庫 (Firestore) 和使用者認證 (Authentication) 服務。
    *   將在後續步驟中建立。
*   **API 金鑰:**
    *   **`FINNHUB_API_KEY`:**
        *   來源：Finnhub 儀表板
        *   獲取方式：註冊 Finnhub 帳號後，從您的儀表板複製。
    *   **`ALPHA_VANTAGE_API_KEY`:**
        *   來源：Alpha Vantage
        *   獲取方式：前往 Alpha Vantage 網站申請並複製。
    *   **`GEMINI_API_KEY`:**
        *   來源：Google AI Studio
        *   獲取方式：前往 Google AI Studio 建立新的 API 金鑰並複製。

### 2.3 專案檔案結構要求

請確認您的專案資料夾中，包含以下檔案與資料夾結構。這是 Vercel 能夠正確識別並部署您的應用程式的基礎。

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

---

## 3. 部署步驟

本節將詳細說明 Kairis App 的部署流程，請按照順序操作。

### 3.1 事前準備：收集您的金鑰與帳號

在開始任何部署操作前，請確保您已完成「2.2 依賴項目與帳號註冊」中提及的所有帳號註冊和 API 金鑰的獲取。將這些金鑰統一整理在一個記事本中，以便後續配置。

### 3.2 準備您的專案檔案並上傳至 GitHub

1.  **確認專案結構：** 再次檢查您的本地專案資料夾是否符合「2.3 專案檔案結構要求」中所示的結構。
2.  **建立 GitHub 儲存庫：**
    *   登入您的 GitHub 帳號。
    *   點擊「New repository」或「Create repository」。
    *   為您的儲存庫命名 (例如 `kairis-app`)。
    *   選擇公開或私人 (建議公開，方便 Vercel 部署)。
    *   點擊「Create repository」。
3.  **上傳專案檔案：**
    *   將您的整個專案資料夾內容（不包含 `.git` 資料夾，如果有的話）上傳到您剛建立的 GitHub 儲存庫中。您可以透過 Git 命令列工具 (`git push`) 或 GitHub 網頁介面上傳。

    ```bash
    # 假設您已在本地初始化 Git 儲存庫並添加了遠端
    # 如果還沒有，請在專案根目錄執行：
    # git init
    # git add .
    # git commit -m "Initial Kairis App project"
    # git branch -M main
    # git remote add origin <您的 GitHub 儲存庫 URL>
    # git push -u origin main

    # 如果您已經有本地 Git 儲存庫，只需：
    git add .
    git commit -m "Prepare for deployment"
    git push origin main
    ```

### 3.3 設定 Firebase

Firebase 將提供 Kairis App 的資料庫和使用者認證功能。

#### 3.3.1 建立 Firebase 專案

1.  前往 [Firebase 控制台](https://console.firebase.google.com/)。
2.  點擊「新增專案 (Add project)」。
3.  為專案命名 (例如 `kairis-app`)，然後點擊「繼續」。
4.  在下一步中，您可以選擇是否啟用 Google Analytics。對於本應用程式，您可以選擇「關閉 Google Analytics」，然後點擊「建立專案」。
5.  等待專案建立完成。

#### 3.3.2 註冊您的網頁應用程式

1.  在您剛建立的 Firebase 專案主控台頁面，點擊網頁圖示 `</>` (新增應用程式)。
2.  為您的應用程式取一個暱稱 (例如 `Kairis Web`)，然後點擊「註冊應用程式 (Register app)」。
3.  **重要步驟：** Firebase 會顯示一段包含 `firebaseConfig` 物件的程式碼片段。**請立即複製這段程式碼。**

    ```javascript
    // 範例：您的 firebaseConfig 會類似這樣
    const firebaseConfig = {
      apiKey: "YOUR_API_KEY",
      authDomain: "YOUR_PROJECT_ID.firebaseapp.com",
      projectId: "YOUR_PROJECT_ID",
      storageBucket: "YOUR_PROJECT_ID.appspot.com",
      messagingSenderId: "YOUR_SENDER_ID",
      appId: "YOUR_APP_ID",
      measurementId: "YOUR_MEASUREMENT_ID" // 如果您啟用了 Analytics
    };

    // 初始化 Firebase
    // firebase.initializeApp(firebaseConfig);
    ```

4.  打開您本地的 `index.html` 檔案。找到對應的 `// *** START: Firebase Integration ***` 區塊，將剛才複製的 `firebaseConfig` 程式碼貼入其中，替換掉範例程式碼。

    ```html
    <!-- index.html 檔案中 -->
    <script type="module">
      // *** START: Firebase Integration ***
      // 將您從 Firebase 控制台複製的 firebaseConfig 貼在這裡
      const firebaseConfig = {
        apiKey: "YOUR_API_KEY",
        authDomain: "YOUR_PROJECT_ID.firebaseapp.com",
        projectId: "YOUR_PROJECT_ID",
        storageBucket: "YOUR_PROJECT_ID.appspot.com",
        messagingSenderId: "YOUR_SENDER_ID",
        appId: "YOUR_APP_ID",
        // measurementId: "YOUR_MEASUREMENT_ID" // 如果您啟用了 Analytics
      };

      // 初始化 Firebase
      firebase.initializeApp(firebaseConfig);
      // *** END: Firebase Integration ***

      // 其他您的 JavaScript 程式碼
      // ...
    </script>
    ```

5.  **完成後，請務必將修改後的 `index.html` 檔案上傳到您的 GitHub 儲存庫，覆蓋舊版本。**

    ```bash
    git add index.html
    git commit -m "Update index.html with Firebase config"
    git push origin main
    ```

#### 3.3.3 啟用登入方式

1.  在 Firebase 控制台的左側選單中，選擇「Authentication」。
2.  點擊「開始使用 (Get started)」。
3.  切換到「Sign-in method」分頁。
4.  在服務供應商列表中，找到並點擊「Google」。
    *   將其啟用 (Enable)。
    *   設定您的專案支援電子郵件 (Project support email)。
    *   點擊「儲存 (Save)」。
5.  同樣地，找到並啟用「匿名 (Anonymous)」登入方式。

#### 3.3.4 建立並設定 Firestore 資料庫

1.  在 Firebase 控制台的左側選單中，選擇「Firestore Database」。
2.  點擊「建立資料庫 (Create database)」。
3.  選擇「以正式版模式啟動 (Start in production mode)」，然後點擊「下一步」。
4.  選擇一個離您最近的地區 (例如 `asia-east1` 或 `us-central1`)，然後點擊「啟用 (Enable)」。
5.  資料庫建立完成後，切換到「規則 (Rules)」分頁。
6.  將所有內容替換為以下規則，然後點擊「發布 (Publish)」：

    ```firestore
    rules_version = '2';
    service cloud.firestore {
      match /databases/{database}/documents {
        // 允許已驗證的使用者讀寫自己的 /users/{userId} 文件
        match /users/{userId} {
          allow read, write: if request.auth != null && request.auth.uid == userId;
        }
        // 如果您有其他集合，請在此處添加相應的規則
        // 例如：
        // match /publicData/{document=**} {
        //   allow read: if true; // 允許所有人讀取公開資料
        //   allow write: if false; // 禁止寫入
        // }
      }
    }
    ```

    這條規則確保每個使用者只能讀寫他們自己的資料（基於 `userId`），提高了資料安全性。

### 3.4 部署到 Vercel

這是將您的應用程式上線的最後一步。

#### 3.4.1 匯入專案

1.  登入 [Vercel 儀表板](https://vercel.com/dashboard)。
2.  點擊「Add New...」 -> 「Project」。
3.  選擇您存放 Kairis 專案的 GitHub 儲存庫，然後點擊「Import」。

#### 3.4.2 設定 Vercel KV (快取資料庫)

Kairis App 可能會使用 Vercel KV 作為快取資料庫，以提高數據檢索效率。

1.  在 Vercel 專案設定頁面，找到「Storage」分頁。
2.  在「Create a database」區塊中，找到 Upstash for Redis 旁邊的「Create」按鈕並點擊。
3.  選擇「Hobby (Free)」方案。
4.  選擇一個離您最近的地區 (建議與 Vercel 專案部署地區相近)。
5.  完成建立與連結。Vercel 會自動幫您設定好所有 KV 相關的環境變數。

#### 3.4.3 設定 API 金鑰 (環境變數)

為了保護您的 API 金鑰並使其在 Vercel 部署環境中可用，您需要將它們設定為環境變數。

1.  回到 Vercel 專案設定的「Settings」 -> 「Environment Variables」。
2.  新增以下三個環境變數：
    *   **Key:** `FINNHUB_API_KEY`, **Value:** (貼上您的 Finnhub 金鑰)
    *   **Key:** `ALPHA_VANTAGE_API_KEY`, **Value:** (貼上您的 Alpha Vantage 金鑰)
    *   **Key:** `GEMINI_API_KEY`, **Value:** (貼上您的 Gemini 金鑰)
3.  確保勾選「Production」和「Preview」環境，以便在所有部署中都可用。

#### 3.4.4 部署！

現在，所有配置都已完成，您可以觸發一次新的部署。

1.  回到 Vercel 專案的「Deployments」分頁。
2.  找到最新的部署紀錄，點擊它右邊的三個點 `...` -> 「Redeploy」。
3.  這將觸發一次新的部署，確保所有最新的程式碼更改和環境變數設定都生效。

#### 3.4.5 設定 Firebase 授權網域 (關鍵步驟)

Vercel 部署成功後，Firebase 需要知道您的應用程式的網域，以便正確處理使用者認證。

1.  部署成功後，Vercel 會給您一個公開的網址 (例如 `kairis.vercel.app` 或您自訂的網域)。請將這個網址複製起來。
2.  回到 [Firebase 控制台](https://console.firebase.google.com/)，選擇您的專案。
3.  在左側選單中，選擇「Authentication」。
4.  切換到「Settings」分頁。
5.  在「已授權的網域 (Authorized domains)」區塊，點擊「新增網域 (Add domain)」。
6.  將您從 Vercel 複製的網址貼上 (例如 `kairis.vercel.app`)，然後點擊「新增」。

---

## 4. 驗證測試

完成所有部署步驟後，請務必進行全面的測試，以確保應用程式的各項功能正常運作。

1.  **存取應用程式：**
    *   在瀏覽器中輸入您的 Vercel 部署網址 (例如 `https://kairis.vercel.app`)。
    *   確認應用程式能夠正常載入，沒有顯示錯誤頁面。
2.  **使用者認證測試：**
    *   嘗試使用 Google 帳號登入。
    *   嘗試使用匿名登入。
    *   確認登入成功後，應用程式能正確識別使用者狀態。
3.  **資料庫功能測試：**
    *   如果您的應用程式有儲存使用者特定資料的功能，請嘗試新增、讀取、更新和刪除資料。
    *   登入 Firebase 控制台的 Firestore Database，確認資料已正確儲存且符合您的預期。
4.  **外部 API 整合測試：**
    *   測試應用程式中所有依賴 Finnhub、Alpha Vantage 和 Gemini API 的功能。
    *   例如，查詢股票即時報價、查看歷史 K 線圖、使用 AI 分析功能。
    *   確認數據能夠正確顯示，且沒有 API 錯誤訊息。
5.  **快取功能測試 (Vercel KV):**
    *   如果應用程式使用了 Vercel KV 進行快取，測試重複請求是否能從快取中獲取數據，以驗證快取是否生效。

---

## 5. 常見問題排解

在部署過程中，您可能會遇到以下一些常見問題及其解決方案：

*   **問題：應用程式無法載入，顯示空白頁或錯誤。**
    *   **解決方案：**
        *   檢查瀏覽器開發者工具 (F12) 的 Console 和 Network 分頁，查看是否有 JavaScript 錯誤或網路請求失敗。
        *   確認 `index.html` 中的 `firebaseConfig` 是否已正確複製貼上，且沒有語法錯誤。
        *   檢查 GitHub 儲存庫中的檔案結構是否與「2.3 專案檔案結構要求」一致。
        *   查看 Vercel 部署日誌，是否有建置失敗的錯誤訊息。
*   **問題：登入功能失效，Firebase 提示「auth/unauthorized-domain」。**
    *   **解決方案：**
        *   這通常是因為您沒有在 Firebase 控制台的 Authentication -> Settings -> 已授權的網域中添加您的 Vercel 部署網址。請按照「3.4.5 設定 Firebase 授權網域」步驟操作。
*   **問題：無法從 Finnhub/Alpha Vantage/Gemini 獲取數據。**
    *   **解決方案：**
        *   檢查 Vercel 專案設定的「Environment Variables」中，對應的 API 金鑰 (`FINNHUB_API_KEY`, `ALPHA_VANTAGE_API_KEY`, `GEMINI_API_KEY`) 是否已正確設定且值是有效的。
        *   確認您的 API 金鑰本身沒有過期或達到使用限制。
        *   檢查 `api/get-stock-data.js` 等後端 API 程式碼是否有錯誤。
*   **問題：Firestore 資料無法讀寫。**
    *   **解決方案：**
        *   檢查 Firebase 控制台的 Firestore Database -> Rules 分頁，確認規則是否已正確設定並「發布」，特別是關於 `request.auth` 和 `userId` 的條件。
        *   確認您的應用程式在讀寫資料時，傳遞的 `userId` 是否與當前登入使用者的 `uid` 相符。
*   **問題：Vercel 部署失敗。**
    *   **解決方案：**
        *   點擊 Vercel 部署頁面上的失敗部署，查看詳細的建置日誌。日誌中會明確指出失敗的原因，例如缺少 `package.json`、依賴安裝失敗或程式碼錯誤。
        *   確保您的 `package.json` 檔案是有效的，並且所有依賴項都能被 Vercel 正確安裝。

---

## 6. 維護和監控

部署完成後，持續的維護和監控對於確保 Kairis App 的穩定運行至關重要。

*   **監控 Vercel 部署日誌：**
    *   定期檢查 Vercel 儀表板中的部署日誌和執行日誌，特別是針對 `api/` 目錄下的無伺服器函數。任何錯誤或異常都將在此處顯示。
*   **監控 Firebase 使用情況：**
    *   在 Firebase 控制台中，監控 Firestore 的讀寫操作量、儲存空間、Authentication 的使用者活動等。雖然這些服務有免費額度，但了解使用趨勢有助於預防潛在的費用超支（如果未來應用程式規模擴大）。
*   **API 金鑰管理：**
    *   定期檢查您的 Finnhub, Alpha Vantage, Gemini API 金鑰是否有效。部分服務可能會有金鑰輪換或到期策略。
*   **程式碼更新與部署：**
    *   當您對 Kairis App 進行功能更新或錯誤修復時，只需將更改推送到 GitHub 儲存庫的 `main` 分支，Vercel 將會自動觸發新的部署。
*   **安全性規則審查：**
    *   定期審查 Firebase Firestore 的安全規則，確保它們仍然符合您的應用程式需求，並且沒有引入新的安全漏洞。
*   **備份策略：**
    *   考慮為您的 Firestore 資料庫設定定期備份，以防萬一。Firebase 提供了資料匯出功能。

---

恭喜您！完成以上所有步驟後，您的 Kairis App 就正式上線了，可以讓全世界的使用者存取和使用了！