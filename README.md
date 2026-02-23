# 台灣網站無障礙檢測 Skill for Claude Code

台灣網站無障礙檢測 Skill for Claude Code

![GitHub Release](https://img.shields.io/github/v/release/DopplerKuo/a11y-tw-audit-skill?label=Release "最新版本")
![License: MIT](https://img.shields.io/badge/License-MIT-yellow "MIT License")
![Claude Code Skill](https://img.shields.io/badge/Claude_Code-Skill-6B4FBB "Claude Code Skill")

本專案為社群開發的**非官方工具**，與 Anthropic、W3C、數位發展部均無隸屬或背書關係。

a11y-tw 是一個 [Claude Code](https://docs.anthropic.com/en/docs/claude-code) Skill，在 Claude Code 環境中以 `/a11y-tw` 指令呼叫。它結合靜態分析與 Playwright runtime 雙軌驗證，針對 WCAG 2.2 AA 55 條準則與台灣在地規範 5 條（共 60 條檢查項目）進行掃描，產出結構化報告並輔助修復。

## 重要聲明

> **自動化工具只能涵蓋約 30% 的 WCAG 準則。**
>
> 本工具**不能取代**：
> - 螢幕閱讀器測試（NVDA、VoiceOver）
> - 真人使用者測試
> - 專業無障礙稽核
>
> **通過本工具的檢測 ≠ 符合無障礙規範。**
>
> 定位：開發階段的輔助檢查，以及 Freego 機器檢測前的預掃工具。
>
> **免責聲明**：本工具依據公開規範提供自動化檢測建議，不構成法律或合規意見。使用者應自行判斷檢測結果的適用性，作者不對因使用本工具產生的任何損失負責。

## 快速開始

### 前置條件

- **Claude Code**（需有 Claude Pro / Max / Team / Enterprise 訂閱）
- **Node.js 18+**（供 Playwright runtime 驗證使用）

### 安裝

將 `a11y-tw/` 目錄複製到你的專案 `.claude/skills/` 下：

```
your-project/
└── .claude/
    └── skills/
        └── a11y-tw/
            ├── SKILL.md
            ├── references/
            │   ├── checklist.md
            │   ├── contrast.md
            │   ├── patterns.md
            │   └── playwright.md
            └── README.md
```

### 使用方式

在 Claude Code 中輸入：

```
/a11y-tw 幫我檢查所有頁面的無障礙
/a11y-tw src/components/Modal.tsx
/a11y-tw --criteria 1.4.3,2.4.7
```

## 功能特色

- **6 步驟結構化審計流程**：範圍確認 → 資源載入 → 靜態分析 → Runtime 驗證 → 報告 → 修復
- **60 條檢查項目**：WCAG 2.2 AA 55 條 + 台灣在地規範 5 條
- **靜態 + Runtime 雙軌驗證**：Playwright 瀏覽器自動化 + axe-core 全頁掃描
- **結構化報告**：統計摘要、失敗 / 警告 / 通過逐條列出、台灣專區獨立呈現
- **三級修復分類**：🔧 可自動修復 / 🎨 需確認（視覺變更）/ 📝 需手動處理（提供指引）
- **修復後自動再驗證**：修復完成後自動重新檢查失敗項目
- **PostToolUse Hook**：每次修復自動審查，防止引入回歸（如意外替換設計 token、引入不必要的 DOM 操作）

## 涵蓋範圍與限制

| 能做 | 不能做 |
|------|--------|
| 靜態 HTML / ARIA 屬性檢查 | 螢幕閱讀器實際朗讀效果 |
| CSS 色彩對比計算（含 oklch、CSS 變數鏈） | 跨頁面導覽一致性（3.2.3 / 3.2.4） |
| Tab 序列與焦點可見性驗證 | 影片字幕與音訊描述品質 |
| Reflow 320px 測試 | 認知層面的主觀判斷 |
| 台灣定位點（`:::`）、Access Key 檢查 | 非 Web 技術（原生 App、桌面應用） |
| axe-core 全頁掃描 | |

## 檔案結構

```
a11y-tw/
├── SKILL.md              # Skill 主體：審計流程、報告模板、修復規則、Hook 定義
├── references/
│   ├── checklist.md      # 60 條完整檢查清單（WCAG 2.2 AA 55 條 + 台灣在地 5 條）+ 台灣法規附錄
│   ├── contrast.md       # 色彩對比靜態分析演算法：CSS 變數解析、oklch 轉換、WCAG 對比度公式
│   ├── patterns.md       # 17 個 React / Next.js 無障礙元件模式 + SSR / CSR 指引
│   └── playwright.md     # Playwright MCP 自動驗證操作參考（10 項 runtime 檢查）
└── README.md
```

## 搭配工具

本 Skill 的靜態分析 + Playwright 驗證可涵蓋大部分自動化可偵測的問題。以下工具用於 Skill 無法覆蓋的面向：

| 工具 | 用途 |
|------|------|
| **Freego** | 台灣官方機器檢測（申請標章必用，僅 Windows） |
| **axe DevTools** | 瀏覽器擴充自動偵測 |
| **NVDA + Firefox** | 螢幕閱讀器測試（台灣審查必用） |
| **VoiceOver + Safari** | macOS 螢幕閱讀器測試 |
| **jest-axe** | CI 整合：單元測試中加入 axe-core 檢查 |
| **eslint-plugin-jsx-a11y** | 開發時即時 linting |

### 螢幕閱讀器手動測試步驟

1. 用 Tab 鍵走完所有互動元素，確認焦點順序與可見性
2. 測試 Access Key（Alt+U/C/Z）跳到對應區塊
3. 操作 Modal / Tab / Accordion，確認狀態變更被朗讀
4. 觸發表單驗證錯誤，確認錯誤訊息被通知、焦點移至第一個錯誤欄位
5. 確認 `aria-label`、`aria-live` 的實際朗讀效果

## 技術細節

技術文件位於 `references/` 目錄：

| 檔案 | 內容 |
|------|------|
| [`checklist.md`](references/checklist.md) | 60 條檢查清單，含每條準則的檢查方式、自動化程度分級、台灣法規對應 |
| [`contrast.md`](references/contrast.md) | 色彩對比靜態分析演算法，含 CSS 變數鏈解析與 oklch 色彩空間轉換 |
| [`patterns.md`](references/patterns.md) | 常見互動元件的無障礙實作模式，含 Modal、Tab、Accordion、Form 等 |
| [`playwright.md`](references/playwright.md) | Playwright runtime 驗證的操作步驟與判斷邏輯 |

## 參考資料

- [數位發展部無障礙規範 110.07](https://accessibility.moda.gov.tw/Accessible/Guide/68)
- [PDIS Freego Implementation Report](https://github.com/PDIS/Freego-Implementation-Report)
- [WCAG 2.2 Quick Reference](https://www.w3.org/WAI/WCAG22/quickref/?versions=2.2&levels=aaa)
- [WAI-ARIA Authoring Practices](https://www.w3.org/WAI/ARIA/apg/)
- [W3C Web Accessibility Laws & Policies — Taiwan](https://www.w3.org/WAI/policies/#taiwan)
- [Freego 官方網站](https://accessibility.moda.gov.tw/)

## Contributors

- [@dopplerkuo](https://github.com/dopplerkuo) — 開發與維護
- [@youxiaaa](https://github.com/youxiaaa) — Review 與測試回饋

## 授權

本專案採用 [MIT License](LICENSE) 授權。
