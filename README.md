# sign-ceremony

一個可同時在「大螢幕展示端」與「行動簽名端」協作的即時簽名/儀式系統。管理者在電腦/電視上建立房間並顯示簽名版面與 QR Code，各位簽名者用手機掃碼進入指定欄位書寫，筆跡即時同步到大螢幕。

> 前端：`admin.html`（管理/展示）與 `signer.html`（手機簽名端）
>
> 後端：`server.js`（Node.js + Express + Socket.IO + SQLite）

---

## ✨ 特色

* **4 碼房號**：建立房間時自動生成 `1000–9999` 的 4 碼數字 ID，分享超快速。
* **分欄簽名**：預設甲/乙各 1 欄，管理端可即時調整甲/乙方欄位數（自動排版到畫布底部兩排）。
* **逐欄同步**：每位簽名者只會畫到自己欄位；支援 **單欄清除**，不影響其他欄。
* **卡片式 QR Code**：每一欄都產生一張 QR 卡，簽名者掃描後自動進入對應欄位。
* **帳號系統（內建）**：Email + 6 位數字密碼（無第三方），支援忘記/重設。
* **權限**：一般使用者只看/清自己的歷史；管理者可切換「顯示全部」並一鍵清除全站歷史。
* **資料庫**：單一 `SQLite` 檔（`data.db`），免安裝伺服器即可落地。
* **部署容易**：預設綁 `0.0.0.0`，同網段手機可直接掃碼進入。

---

## 📂 專案結構

```
.
├─ admin.html      # 管理/大螢幕端（建立房間、調整欄位、顯示 QR、同步書寫）
├─ signer.html     # 手機簽名端（顏色/粗細/清欄；掃碼輸入 4 碼房號）
├─ server.js       # 後端 API + WebSocket + SQLite + Auth
└─ data.db         # 首次啟動會自動建立（如不存在）
```

---

## 🔧 系統需求

* Node.js ≥ 18（含 `npm`）
* 作業系統：Windows / macOS / Linux 皆可
* 同網段行動裝置（掃描 QR 進入簽名端）

---

## 🚀 安裝與啟動

1. **安裝相依**

   ```bash
   npm i express socket.io sqlite3 bcryptjs jsonwebtoken cors
   ```

2. **設定環境變數（可選）**

   * `PORT`：服務埠，預設 `3000`
   * `JWT_SECRET`：JWT 簽章金鑰（務必在正式環境修改）
   * `ADMIN_EMAILS`：以逗號分隔的管理者 Email 清單（首啟動會自動播種）
   * `ADMIN_INITIAL_PIN`：管理者初始 6 位數字密碼（僅首次播種使用）

   例（Linux/macOS）：

   ```bash
   export PORT=3000
   export JWT_SECRET="please-change-me"
   export ADMIN_EMAILS="boss@example.com,teacher@example.org"
   export ADMIN_INITIAL_PIN=000000
   ```

3. **啟動**

   ```bash
   node server.js
   ```

   終端機將顯示伺服器位址，例如：`http://0.0.0.0:3000`

4. **開啟管理端**：在桌面瀏覽器進入：

   * `http://<你的電腦IP>:3000/admin.html`

5. **登入/註冊**：

   * 首次使用可直接在登入彈窗點「註冊」。
   * 若事先設定 `ADMIN_EMAILS`，該帳號會自動擁有管理者權限。

6. **建立房間**：

   * 在管理端輸入標題與畫布尺寸（或用預設值），點「建立房間」。
   * 會產生 **4 碼房號**，並在左右區塊顯示每一欄位對應的 **QR Code 卡片**。

7. **行動端簽名**：

   * 用手機掃描對應欄位的 QR，或開 `http://<你的電腦IP>:3000/signer.html` 手動輸入房號。
   * 調整顏色/粗細、在畫布書寫；筆跡立即同步到大螢幕指定欄位。

---

## 🔐 帳號 & 權限

* **註冊**：Email + **6 位數字密碼**（如 `123456`）。
* **登入**：成功後前端會保存 JWT；`/api/me` 取得使用者資訊與角色。
* **忘記/重設**：

  * 送出 Email 產生 6 碼驗證碼（以 API 回傳顯示在畫面，方便測試）。
  * 輸入驗證碼與新 6 位數字密碼完成重設。
* **角色**：

  * `user`：僅能看到與清除自己的歷史。
  * `admin`：可切換「顯示全部房間」、一鍵**清除所有人的歷史**。

> 提示：正式環境務必更換 `JWT_SECRET`，並將伺服器放在公司/校園可信任網段，或加上反向代理與 HTTPS。

---

## 🧭 使用流程（管理端）

1. 登入 → `建立房間`。
2. （可選）用 `甲方欄位數 / 乙方欄位數` 調整欄位數 → `套用欄位數（自動排版）`。
3. 將螢幕投放到電視或投影；現場人員掃描 **自己欄位的 QR 卡片**。
4. 若某欄需重簽，可請該簽名者在手機按 `清除本欄`，或管理端手動局部清理。
5. 活動結束後，在「歷史房間」可刪除單一房間，或（管理者）一鍵清除全部。

---

## 📡 API 一覽（簡要）

> 所有 `POST/DELETE` 除標註外均需 **Authorization: Bearer <token>**。

### Auth

* `POST /api/register` — 建立帳號（Body: `{email, password: 6位數字}`）
* `POST /api/login` — 登入（Body: `{email, password}`）→ `{token}`
* `GET  /api/me` — 取得我的資訊（需登入）
* `POST /api/forgot` — 送驗證碼（Body: `{email}`）→ `{code}`（測試用回傳）
* `POST /api/reset` — 重設密碼（Body: `{email, code, newPassword}`）

### Events

* `POST /api/create-event` — 建立房間（Body: `{title?, stageWidth?, stageHeight?}`）→ `{eventId}`
* `GET  /api/events` — 取得歷史（一般：僅自己；管理者：`?all=1` 全部）
* `GET  /api/event/:eventId` — 讀取單房間含欄位（免登入，供簽名端讀取）
* `POST /api/event/:eventId/slots` — 更新 A/B 欄位座標（擁有者或管理者）
* `DELETE /api/event/:eventId` — 刪單筆（擁有者或管理者）
* `POST /api/events/clear` — 刪**自己的**全部（需登入）
* `POST /api/admin/events/clear-all` — 管理者刪全站

### WebSocket（Socket.IO）

* `join:event` — { eventId, role, slotSide, slotIndex }
* `stroke` — { points, size, color, sourceWidth, sourceHeight, slotSide, slotIndex }
* `clear` — { slotSide, slotIndex }（**只清該欄**）
* 伺服器廣播：

  * `stroke`（管理端繪到對應欄位）
  * `clear`（該欄位局部清空與標籤重繪）
  * `slots:update`（管理端更新欄位排版時）
  * `event:deleted`（該房間被刪除時通知簽名端/管理端）

---

## 🖥️ 畫布與版面

* 畫布尺寸可自由設定（預設 1000×1000）。
* 欄位自動排版：兩排（上排甲方、下排乙方），等寬分配，邊距與間距固定，避免現場跑版。
* 簽名端會把手機畫布點位等比例映射到管理端對應欄位。

---

## 🛡️ 安全建議

* 更換 `JWT_SECRET`、限制管理端可見範圍。
* 正式環境配合 Nginx/Apache 反代與 HTTPS，並開啟適當的 CORS 政策。
* 管理者初始 PIN 僅用於播種；建立後請**立即重設**該帳戶密碼。

---

## 🧰 疑難排解

* **手機掃不到或無法連線**：

  * 確保手機與伺服器同一個 Wi‑Fi/網段。
  * 用伺服器 IP（非 `localhost`）開啟 `admin.html`，QR 也會帶入正確來源。
* **畫筆有延遲**：

  * 儘量使用同區網；若跨網請確保延遲 < 100ms。
* **欄位數與排版亂**：

  * 修改欄位數後記得按 `套用欄位數（自動排版）`，再重新產生 QR。
* **忘記管理者密碼**：

  * 用 `POST /api/forgot` 取得驗證碼，再 `POST /api/reset` 重設。
* **資料庫毀損/轉移**：

  * 停止服務後備份 `data.db`；跨機移轉時把檔案帶著即可。

---



