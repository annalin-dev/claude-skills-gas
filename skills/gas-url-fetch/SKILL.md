---
name: GAS 通用觀念（URL 讀取 + 推送邏輯）
description: GAS 通用知識：(1) 使用者貼 Web App URL 直接 WebFetch 讀內容 (2) clasp push 不必再「建立新版本 → 部署」，HEAD 部署已自動同步。適用所有 GAS 專案。
disable-model-invocation: false
---

# GAS 通用觀念

兩個跟「哪個專案」無關的 GAS 通用知識：URL 怎麼讀、push 完到底要不要再 deploy。

---

## 一、push 完不必再「建立新版本 → 部署」

### GAS 的部署機制

GAS 的 Web App 部署分兩種：

| 類型 | 行為 | 觸發方式 |
|---|---|---|
| **HEAD 部署**（測試/最新） | 每次 `clasp push` 後**自動更新**，使用者重整就看到新版 | clasp push |
| **正式版本部署** | 凍結某個版本當穩定版，不會被後續 push 影響 | GAS 編輯器 → 部署 → 管理部署 → 建立新版本 → 部署 |

### 日常開發只需要 push

```bash
# 改完程式碼，這一行就夠了
clasp push
# 或專案內的 build script，例如：
node build.js --push
```

push 完，使用者**重整 Web App 頁面**就會看到變更。**不需要**每次都進 GAS 編輯器手動「建立新版本」。

### 什麼時候才需要「建立新版本 → 部署」

- 要把穩定版發給特定群體（不希望被開發中的程式碼影響）
- 要凍結某個版本當作正式發行
- GAS Web App URL 換新（很罕見）
- doGet/doPost 的「執行身份」或「存取權限」設定變動

### 排查「push 完但畫面沒變」

依序檢查：
1. **使用 HEAD 部署的 URL 嗎**：Web App URL 對應的是 HEAD 還是正式版本？正式版本要重新建立版本才會更新
2. **clasp push 有錯誤訊息嗎**：例如 API 未啟用、權限不足
3. **瀏覽器快取**：強制重整（Cmd+Shift+R）
4. **GAS 編輯器內看到新檔案了嗎**：打開 GAS 編輯器確認程式碼已更新

---

## 二、貼 GAS 網址即可直接讀取

### 觸發時機

使用者貼出 GAS Web App URL（例如 `https://script.google.com/macros/s/AKfycb.../exec`），不管是：
- 「這個 GAS 沒更新」
- 「你看看這頁長怎樣」
- 「為什麼這頁沒顯示 XXX」
- 或單純把網址貼出來

→ **直接用 WebFetch 工具讀取該 URL**，不要叫使用者另外提供截圖或內容。

### 操作方式

```
WebFetch(url=使用者貼的網址, prompt="...想知道什麼...")
```

例：
```
WebFetch("https://script.google.com/macros/s/AKf.../exec",
         "頁面上有顯示哪些按鈕和欄位？")
```

### 用途

1. **檢查線上渲染結果**：確認 GAS 端真的有什麼 HTML/JSX 被輸出
2. **驗證 doGet/doPost 回應**：拿到 JSON 或 HTML 回應內容
3. **排查推送沒生效**：使用者推送後說「沒看到變更」，可能是：
   - Web App URL 對應**正式版本部署**（凍結版本），不是 HEAD
   - 推送有錯誤但沒注意到
   - 快取問題
4. **核對前後端是否一致**：對照 GAS 線上輸出 vs 本地原始碼

### 注意事項

- **網址帶不同的 query string** 會得到不同回應（doGet 可吃參數）
- 部署可能要登入 Google 帳號才能存取，**WebFetch 拿不到需要登入的頁面**
  → 若該 URL 設定為「只有特定使用者能存取」，會回到登入頁
  → 公開設定為「Anyone」才能用 WebFetch
- GAS Web App URL 通常以 `/exec` 結尾（正式部署）或 `/dev` 結尾（開發測試）

---

## 與其他 skill 的關係

- 推送 GAS 程式碼 → 用 `crm-gas-deploy` 或 `gas-jsx-deploy`
- 推送後驗證線上結果 → 本 skill
- 「為何 push 完沒看到變更」的疑問 → 本 skill 的部署機制段落
