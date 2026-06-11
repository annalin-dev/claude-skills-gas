# claude-skills-gas

Claude Code skills for Google Apps Script (GAS) projects.

## Skills

### gas-url-fetch
GAS Web App URL 讀取與部署觀念。

- 使用者貼出 GAS Web App URL 時，直接用 WebFetch 讀取，不需要截圖
- `clasp push` 完不需要再手動「建立新版本 → 部署」，HEAD 部署自動同步
- 說明什麼情況才需要建立正式版本

### gas-jsx-deploy
React + GAS 新專案初始化與 clasp 部署流程。

- JSX 用 Babel 編譯成純 JS 再推送到 GAS 的完整設定
- clasp 初始化、`.claspignore`、build script 設定
- 首次部署與日常推送流程

## 安裝

在 Claude Code 對話框輸入：

```
skills add https://github.com/annalin-dev/claude-skills-gas
```

兩個 skills 會一起安裝。
