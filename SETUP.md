# 進出貨管家：開啟多人共用（雲端同步）

沒有設定雲端時，網站是**單機模式**：資料只存在那台裝置的瀏覽器。
照下面步驟設定一次（約 15 分鐘），之後負責人和夥伴用 Google 帳號登入，就會看到**同一份資料**，而且即時同步。

架構：GitHub Pages（放網頁）＋ Firebase Authentication（Google 登入）＋ Cloud Firestore（雲端資料庫）。

---

## 1. 建立 Firebase 專案

1. 到 <https://console.firebase.google.com>，用**負責人的 Gmail** 登入。
2. 「建立專案」→ 取一個名字（例如 `stockbook-myshop`）→ Google Analytics 可以關掉 → 建立。

## 2. 開啟 Google 登入

1. 左側選單「建構 → Authentication」→「開始使用」。
2. 「登入方式」分頁 →「Google」→ 啟用 → 選一個支援 Email → 儲存。

## 3. 建立資料庫

1. 左側「建構 → Firestore Database」→「建立資料庫」。
2. 位置選 **`asia-east1`（台灣）**，之後不能改。
3. 選「以正式版模式啟動」。

## 4. 貼上安全規則

1. Firestore 頁面上方「規則」分頁。
2. 把 `firestore.rules` 的內容整段貼上，取代原本的內容。
3. 確認裡面的負責人 Gmail（目前是 `moton63568@gmail.com`）正確。
4. 按「發布」。

規則的效果：

- 只有負責人和名單上的成員可以讀寫。
- 進出貨紀錄**只能新增**，不能修改或刪除。記錯了要用「沖銷」。
- 每筆紀錄的記錄者一定是登入的那個人，沒辦法冒名。
- 金額（成本、售價、單價）另外存放，**現場人員讀不到**。

## 5. 把設定貼進網頁

1. Firebase 專案首頁 → 齒輪「專案設定」→「一般設定」→ 最下面「你的應用程式」→ 點 `</>`（網頁）。
2. 取個暱稱 → 註冊應用程式（**不用**勾選 Firebase Hosting）。
3. 畫面會出現一段 `const firebaseConfig = { apiKey: ..., authDomain: ..., ... };`。
4. 打開 `index.html`，找到最上面的 `CONFIG`，改成這樣：

```js
const CONFIG = {
  firebase: {
    apiKey: "AIza....",
    authDomain: "stockbook-myshop.firebaseapp.com",
    projectId: "stockbook-myshop",
    storageBucket: "stockbook-myshop.firebasestorage.app",
    messagingSenderId: "1234567890",
    appId: "1:1234567890:web:abcdef"
  },
  ownerEmail: '負責人的Gmail@gmail.com',
};
```

> `apiKey` 放在網頁裡是正常的，它不是密碼。真正保護資料的是第 4 步的安全規則。

## 6. 放上網路（GitHub Pages）

雲端模式需要用**網址**開啟，不能直接雙擊 HTML 檔。手機用相機掃條碼時也一樣要用網址（https）。

1. 把 `index.html` push 到 GitHub（repo：`stock-app`）。
2. 到 GitHub repo →「Settings → Pages」→ Source 選 `Deploy from a branch`，Branch 選 `main`、資料夾選 `/ (root)` → Save。
3. 約 1 分鐘後網站會更新：<https://daniely-gift.github.io/stock-app/>

## 7. 授權網域

Firebase →「Authentication → 設定 → 已授權網域」→「新增網域」→ 輸入 `daniely-gift.github.io`（不用加 https 或路徑）。

沒做這一步的話，登入時網站會提示「這個網址還沒加到已授權網域」。

## 8. 開始使用

1. 打開網址，用負責人的 Gmail 登入。
2. 到「後台」→「成員與角色」→ 輸入對方的 Gmail、選角色 →「加入」。
3. 對方用那個 Gmail 登入就能使用。不在名單上的帳號會看到「還沒有使用權限」。

| 角色 | 記帳、調撥、盤點、商品 | 看得到 / 填金額 | 管理成員、倉庫 |
|---|---|---|---|
| 負責人 | ✅ | ✅ | ✅ |
| 管理員 | ✅ | ✅ | ❌ |
| 現場人員 | ✅ | ❌ | ❌ |

> 加入現場人員之前，請先到「後台」確認沒有「舊資料的金額沒有轉換」的提醒，有的話先按轉換。

## 9. 搬移單機模式的舊資料

瀏覽器的資料是**依網址分開存的**。以前用檔案或別的網址開過的資料，在新網址上看不到，所以要用備份檔搬過去：

1. 用**以前的方式**打開舊版網站 →「設定 → 下載備份」，得到一個 `.json` 檔。
2. 打開新網址，用負責人登入 →「設定 → 從備份檔匯入到雲端」→ 選那個檔案。
3. 匯入只會新增，不會覆蓋或刪除雲端上已有的資料。重複匯入同一份也不會變成兩份。

---

## 費用

Firebase 免費方案（Spark）每天有 5 萬次讀取、2 萬次寫入，小店剛開始完全夠用。

要注意的是，這個網站每次打開都會讀取全部紀錄。等紀錄累積到幾千筆、又有好幾個人整天開開關關，有機會超過免費讀取額度。超過時當天會暫停讀取。到時候可以：

- 改用「Blaze 隨用隨付」方案：讀取約每 10 萬次 US$0.03～0.04，小店一個月通常是幾塊到幾十塊台幣。記得在 Google Cloud 設定「預算提醒」。
- 或請人幫你加上「月結快照」，只讀最近幾個月的紀錄。

用量可以在 Firebase →「Firestore → 用量」查看。

## 常見問題

- **網路斷掉還能記帳嗎？** 可以。資料會先存在那台裝置，恢復連線後自動上傳。左上角會顯示「離線中」或「同步中」。
- **兩個人同時記帳會衝突嗎？** 不會。每筆紀錄都是獨立新增的，庫存由所有紀錄即時算出來。
- **盤點進度存在哪裡？** 存在進行盤點的那台裝置。按「完成盤點」後，調整紀錄才會上傳，大家都看得到。
- **改了安全規則？** 這個 repo 的 `firestore.rules` 是後台規則的備份，兩邊要保持一樣：改完貼回 Firebase 後台並按「發布」。
- **想改回單機模式？** 把 `CONFIG.firebase` 改回 `null`。
