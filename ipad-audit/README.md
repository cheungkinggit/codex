# 藍循 · 學生 iPad 抽查

手機優先的學校內部抽查紀錄系統。管理員建立行動、指定統籌和每班老師、貼上各班學生名單及設定抽樣人數；老師用 Google 帳戶登入，隨機抽樣、記錄結果、處理缺席重抽。統籌及管理員可檢視整體報告，列印／另存 PDF，完成行動仍可從歷史紀錄重開。

## 上線前設定

此 repository 只存放網頁程式，**不可上傳學生名單或報告**。學生資料保存在你自己的 Firebase Cloud Firestore。GitHub Pages 網頁本身可公開讀取，但沒有 Firebase 權限的人無法讀取資料。上線前請學校確認資料保留、帳戶管理及內部私隱安排。

1. 在 [Firebase 控制台](https://console.firebase.google.com/) 建立學校控制的專案和 Web app；啟用 **Authentication → Google** 登入，並建立 **Cloud Firestore** 資料庫。將 GitHub Pages 網域（例如 `cheungkinggit.github.io`）加到 Authentication 的 Authorized domains。
2. 在 Firestore **Rules** 貼上 `firestore.rules` 全文並發佈。預設所有讀寫都會拒絕，直至建立管理員。不要改用測試模式規則處理真實學生資料。
3. 複製 `firebase-config.example.js` 為 `firebase-config.js`，填上 Firebase Web app 的 `apiKey`、`authDomain`、`projectId`、`appId`。這些是公開的專案識別資料，不是後端密鑰；真正的存取權由 Authentication 與 Firestore Rules 控制。
4. 管理員先用 Google 登入一次（會因尚未建立管理員而見到空白／權限提示）。到 Firebase Authentication 找出該人的 **UID**，再在 Firestore 控制台手動新增文件 `admins/<UID>`（內容可設 `name: "管理員"`）。只由 Firebase 控制台管理這個名單；網頁本身無法提升權限。登出再登入。
5. GitHub Pages 可用 GitHub Actions 將 `ipad-audit/` 發佈到專案網址 `/codex/ipad-audit/`。設定 Firebase config 時，建議把 `firebase-config.js` 寫入 GitHub Actions **repository secrets** 並在建置時產生檔案；不要把實際設定和學生資料混為一談。另可直接在發布分支放公開 config（它並非密鑰）。
6. 先以虛構學生名單測試：建立行動、用老師帳戶登入抽樣及記錄、另用統籌帳戶看報告，再輸入真實學生資料。

## 權限和工作流程

- **管理員**：可見全部行動及班別；建立行動和閱覽報告。管理員由 Firestore 控制台 `admins/<UID>` 決定。
- **統籌人**：登入電郵須和該次行動指定電郵相同；可看該次所有班別詳情、報告，並結束或重新開啟行動。
- **負責老師**：只可看自己班的抽樣名單及紀錄；各老師都會看到所有班的完成進度，但無權讀其他班的學生資料。資料更改後其他裝置會即時接收行動進度。
- 每班抽查人數預設五人，可更改。抽樣後的缺席重抽會保留誰被替換及時間；已有檢查結果的學生不能用「缺席」重抽。抽樣使用瀏覽器的安全隨機數來源。
- 報告以 A4 橫向緊湊排版；若選取人數或備註太多，列印可能分成多頁。瀏覽器「列印 → 儲存為 PDF」即可保存。

## 技術及限制

純 HTML/CSS/JavaScript，Firebase Authentication 和 Firestore；無需自設伺服器。行動建立時，學生名單會複製到該次行動；新的行動需再次貼上名單。請勿把 CSV／學生名單 commit 到 GitHub。班內有多位老師同時操作同一班時，系統仍按指定的一個負責老師帳戶管理。Firestore 規則保障資料存取和主要不可變欄位，已授權老師仍需按學校流程正確記錄。網頁登出並不會刪除已列印或下載的報告。

本機預覽：在此目錄執行 `python -m http.server 8000`，然後開啟 `http://localhost:8000/`。Firebase 須把 `localhost` 加入 Authorized domains。沒有 `firebase-config.js` 時會顯示設定提示，不會接收學生資料。
