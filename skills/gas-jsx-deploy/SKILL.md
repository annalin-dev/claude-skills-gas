---
name: GAS JSX 部署（通用）
description: 適用任何 React + Google Apps Script 新專案的初始化與 clasp 部署流程
disable-model-invocation: false
---

# GAS JSX 部署通用流程

適用所有使用 React（JSX）+ Google Apps Script 的專案。
JSX 需要預先用 Babel 編譯成純 JS，再透過 clasp 推送到 GAS。

## Anna 的 Mac 環境

- Node：`/usr/local/bin/node`
- Clasp：`~/.local/bin/clasp`（已用 anna.lin@ichef.com.tw 登入）
- npm：`/usr/local/bin/node /usr/local/lib/node_modules/npm/bin/npm-cli.js`
- Apps Script API：已在 script.google.com/home/usersettings 開啟

## 新專案初始化步驟

### 1. 安裝依賴
```bash
cd 新專案資料夾
/usr/local/bin/node /usr/local/lib/node_modules/npm/bin/npm-cli.js init -y
/usr/local/bin/node /usr/local/lib/node_modules/npm/bin/npm-cli.js install @babel/core @babel/preset-react
```

### 2. 複製 build.js
從 CRM 專案複製 `build.js`，修改 `PAGE_FILES`、`PARTIAL_FILES`、`BACKEND_FILE` 對應新專案的檔案清單。

### 3. 設定 .clasp.json
```json
{
  "scriptId": "貼上 GAS Script ID",
  "rootDir": "./gas_output"
}
```
Script ID：GAS 編輯器 → 專案設定 → 指令碼 ID

### 4. 首次推送
```bash
node build.js --push
```
**首次**推送後才需要：GAS 編輯器 → 部署 → 管理部署 → 建立新版本 → 部署（取得 Web App URL）

## 日常使用

```bash
# 編譯全部 + 推送
node build.js --push

# 只編譯特定檔案（關鍵字篩選，不推送）
node build.js FileName
```

**推送後不需要重新部署。** GAS 的 HEAD 部署在每次 `clasp push` 後自動更新，使用者重整頁面即可看到最新程式碼。只有以下情況才需要「建立新版本」：
- 凍結穩定版本，避免開發中程式碼影響使用者
- Web App URL 需要換新

## 貼 GAS Web App URL 給 Claude 讀

使用者貼 GAS Web App URL（`script.google.com/.../exec`）時，Claude 可直接用 WebFetch 讀取頁面內容：
- 檢查目前線上頁面的渲染結果
- 驗證 doGet/doPost 回應
- 排查「推送後沒看到變更」（確認該 URL 對應 HEAD 還是正式版本部署）

## 頁面 vs Partial 差異

| 類型 | 說明 | 輸出 |
|------|------|------|
| 頁面（Page） | 完整 HTML，含 `<script type="text/babel">` | JSX 編譯為 `<script>`，移除 Babel CDN |
| Partial | 純 JSX，無 HTML 包裝 | 直接輸出純 JS，**不加** `<script>` 標籤 |

Partial 透過 `<?!= include('FileName') ?>` 嵌入頁面，已在頁面 `<script>` 區塊內。

## appsscript.json 必要設定

build.js 自動產生，**不可刪除 webapp 欄位**，否則 Web App URL 失效：
```json
{
  "timeZone": "Asia/Taipei",
  "dependencies": {},
  "exceptionLogging": "STACKDRIVER",
  "runtimeVersion": "V8",
  "webapp": {
    "executeAs": "USER_DEPLOYING",
    "access": "ANYONE"
  }
}
```

## GAS LockService 原則

LockService **不可重入**。批次寫入操作：
- 最外層函數取得一次 lock
- 內部呼叫 unlocked helper（函數名慣例加底線 `_`）
- finally 統一 releaseLock

## 常見錯誤

**`Unexpected token '<'`** — Partial 還是 JSX 版本，重新 `node build.js --push`

**Web App URL 失效** — `appsscript.json` 缺少 `webapp`，或推送後未建立新版本部署

**clasp push 失敗：API 未啟用** — 前往 script.google.com/home/usersettings 開啟 Google Apps Script API
