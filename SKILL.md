---
name: a11y-tw
disable-model-invocation: true
argument-hint: "[path] [--criteria ids]"
description: >
  台灣網站無障礙審計工具。掃描 React/Next.js/HTML，
  依 WCAG 2.2 AA + 台灣在地規範產出審計報告並輔助修復。
hooks:
  PostToolUse:
    - matcher: "Edit"
      hooks:
        - type: prompt
          prompt: |
            An a11y audit fix was applied. Review the edit in $ARGUMENTS.
            Flag (ok=false) if ANY of these are true:
            1. A named CSS class/variable/token was REPLACED with a hardcoded value (e.g., bg-accent-teal → bg-[#0b7d86])
            2. DOM manipulation APIs were introduced (MutationObserver, document.title=, window.addEventListener) for something a framework likely handles natively
            3. An HTML element type was changed (e.g., <p> → <legend>, <div> → <section>) — these have different browser default styles and may break layout
            4. New code reimplements something visible in the old_string context (e.g., custom focus trap when existing component already handles it)
            Simple attribute additions (alt, aria-label, role, lang, scope, autocomplete, aria-*) are always ok=true.
            $ARGUMENTS
  Stop:
    - hooks:
        - type: agent
          prompt: |
            The a11y-tw audit skill is finishing. Verify completeness:
            1. Read Claude's last message. Did it apply any fixes (Edit calls)?
            2. If fixes were applied, check: was re-verification performed AFTER the fixes?
               Look for evidence of: re-running checks, Playwright re-tests, or a diff table.
            3. If fixes applied but NO re-verification done, return ok=false with reason "Re-verification required after fixes."
            4. If no fixes were applied (report-only), return ok=true.
            5. If fixes AND re-verification were both done, return ok=true.
            $ARGUMENTS
---

# 台灣無障礙檢測工具（a11y-tw）

## 使用方式

```
/a11y-tw 幫我檢查所有頁面的無障礙
/a11y-tw src/components/Modal.tsx
/a11y-tw --criteria 1.4.3,2.4.7
```

## 本 Skill 不適用於

- 螢幕閱讀器實際朗讀效果測試（需 NVDA/VoiceOver 手動測試）
- 跨頁面導覽一致性驗證（3.2.3 / 3.2.4，需人工比對多頁面）
- 影片字幕與音訊描述品質評估（1.2.x）
- 非 Web 技術（原生 App、桌面應用程式）
- 認知層面的主觀判斷（錯誤訊息建議品質、操作指示清晰度）

## ⚠️ 重要：修復流程與優先級

任何修復都必須**先產出報告**，再依據修復類型決定執行方式。

### 修復類型分類

- **🎨 需確認（視覺變更）**：任何可能影響 UI 外觀的修復，**不可直接執行**，必須先以 plan 列出，**等使用者確認**後才執行。
- **🔧 可自動修復（無視覺變更）**：不影響視覺的純語意或屬性修復，可在使用者同意後批次執行。
- **📝 需手動處理（提供指引）**：涉及複雜邏輯、需人工判斷或需跨檔案協調的項目，僅提供修復指引。

### 修復優先級

1. **🔴 高**：阻礙核心功能、違反 A/AA 級準則、台灣審計必查項目。
2. **🟡 中**：影響部分使用者體驗、建議遵循的最佳實踐。
3. **🟢 低**：輕微問題、AAA 級建議、程式碼風格。

| 修復類型 | 常見範例（非窮舉） |
|------|------|
| 🎨 **需確認** | 色彩對比調整、焦點指示器樣式、目標大小調整、新增 Skip Link/定位點、文字間距/行高調整 |
| 🔧 **可自動修復** | 加 `alt=""`、`lang`、`rel`、ARIA 屬性、`scope`、修正標題跳級、加 `autoComplete`、`role`、狀態屬性 |
| 📝 **需手動處理** | 影片字幕、複雜圖表描述、鍵盤/焦點管理流程設計、一致性導覽/識別、**Hover/Focus 狀態的對比度** |

> **分類原則**：不在上表中的修復，依此判斷 — 🎨 會改變畫面上任何可見像素（顏色、大小、間距、佈局、新增/移除視覺元素）？→ 需確認。🔧 純增刪 HTML 屬性或語意標籤、不改變計算後視覺結果？→ 可自動修復。其餘 → 📝 需手動處理。

## 審計流程

### Step 1：確認範圍

解析使用者指令，確定掃描範圍：

| 指令 | 範圍 |
|------|------|
| `/a11y-tw` | `app/` 或 `pages/` 或 `src/` 整個前端目錄 |
| `/a11y-tw <path>` | 該路徑下所有 .tsx/.jsx/.html 檔案 |
| `/a11y-tw <file>` | 單一檔案 + 其 import 的元件 |
| `/a11y-tw --criteria <ids>` | 全部檔案但只查指定準則 |

使用者可能以自然語言描述掃描目標（如「幫我看 enterprise 與 public 下所有頁面」）。此時：
1. 從描述中提取關鍵詞，用 Glob 搜尋匹配的目錄或檔案
2. 將結果列出，請使用者確認掃描範圍再繼續

### Step 2：載入參考資源

依掃描內容按需讀取：

| 場景 | 檔案 |
|------|------|
| **每次必讀**（`--criteria` 模式時僅讀匹配的準則段落） | `references/checklist.md`（60 條完整清單 + 台灣法規附錄） |
| 遇到複雜互動元件（Modal、Tab、Accordion、Carousel、Form、Date Picker 等）或台灣定位點實作 | `references/patterns.md` |
| 掃描包含 1.4.3 / 1.4.11 對比度 | `references/contrast.md` |

#### 錯誤處理

- **無匹配檔案**：回報「指定路徑未找到可掃描的 .tsx/.jsx/.html 檔案」，結束流程
- **CSS 入口檔未找到**：對比度靜態分析標為 🟡 警告，依賴 Step 3b runtime 驗證補強

### Step 3：掃描

> **⚠️ Step 3 包含 3a 和 3b 兩個子步驟。兩者都必須執行。完成 3a 後必須立刻進入 3b，不得跳過 3b 直接進入 Step 4。**

#### Step 3a：靜態分析

依 `references/checklist.md` 排列順序一次掃描全部適用準則：第一原則（可感知）→ 第二原則（可操作）→ 第三原則（可理解）→ 第四原則（穩健性）→ 台灣特有。

對每條準則產出初判結果：
- **純靜態準則（43 條）**：直接產出最終判定（✅ 通過 / 🔴 失敗 / 🟡 警告）
- **混合準則（13 條）**：產出靜態初判，標註 `⚠️ 需 runtime 確認`

> **3a 完成條件**：必須對每條適用準則逐一產出判定（✅/🔴/🟡/⚠️），以結構化清單或表格呈現。僅讀取原始碼檔案不算完成 3a — 讀取是準備工作，逐條準則比對並記錄判定才是 3a 的產出。未見到逐條判定結果，不得進入 3b。

特別強化以下項目（其餘準則仍須依 checklist.md 逐條完整掃描，不得因未列於此處而降低檢查深度）：
- **1.4.11 非文字對比**：檢查互動元件的 Hover & Focus 狀態對比度。
- **2.4.11 焦點未被遮蔽**：檢查 sticky 元素是否遮擋焦點。
- **4.1.2 名稱、角色、值**：檢查自訂元件的 role 和 aria 狀態屬性。
- **TW-03 網站導覽**：驗證「3 次 Tab 內到達導覽」的可操作性。
- **3.3.8 無障礙驗證**：澄清密碼欄位的 `autocomplete` 政策。

##### 靜態色彩對比分析（1.4.3 / 1.4.11）

掃描對比度準則時，依 `references/contrast.md` 執行靜態分析：

1. 讀取專案 CSS 入口檔，建立 **CSS 變數 → 色碼** 對應表（含 oklch 轉換、var() 鏈解析）
2. 掃描元件中的 Tailwind class（`text-*` / `bg-*`）與 inline style，解析前景/背景色
3. 對每組可解析的前景/背景色計算 WCAG 對比度，不足者標為 🔴 失敗
4. CSS 偽類（`:hover` / `:focus`）中宣告的色彩亦同步分析（1.4.11）
5. 無法靜態解析的色彩組合（動態 JS、繼承鏈不完整、半透明合成）標為 `⚠️ 需 runtime 確認`

#### Step 3b：Runtime 驗證（必須執行）

**Step 3a 完成後，立刻執行本步驟。**

##### 1. 偵測開發伺服器 URL

依以下優先順序偵測 dev server URL，取第一個命中的結果：

1. **package.json scripts**：解析 `dev` 或 `start` 腳本中的 port flag（`--port`、`-p`、`PORT=`）
2. **環境變數檔**：依序檢查 `.env.local` → `.env.development` → `.env` 中的 `PORT`
3. **框架設定檔**：依 Step 5a 偵測到的框架讀取對應設定（`next.config.*` / `vite.config.*` / `nuxt.config.*` / `angular.json`）
4. **框架預設值**：Next.js/CRA/Nuxt → 3000、Vite → 5173、Angular → 4200

##### 2. 啟動與連線

嘗試 `browser_navigate` 至偵測到的 URL：
- **成功** → 讀取 `references/playwright.md`，繼續執行下方批次檢查
- **失敗（連線拒絕）** → 嘗試用專案的 dev script 啟動開發伺服器（`npm run dev` 或對應指令），等待就緒後重試
- **啟動後仍失敗** → 全部 runtime 項目標註「僅靜態分析，開發伺服器未啟動」，跳至 Step 4

##### 3. 依檢查類型批次執行

| 批次 | 涵蓋準則 | 操作 |
|------|---------|------|
| axe-core 全頁掃描 | 通用 | CDN 注入 + `axe.run()` |
| 對比度驗證 | 1.4.3, 1.4.11 | `getComputedStyle()` 批次取色 |
| Tab 序列與焦點 | 2.1.2, 2.4.3, 2.4.7, 2.4.11, TW-03 | 連續 Tab 按鍵 + 每步截圖 |
| Reflow 測試 | 1.4.10 | `resize(320)` + 截圖 |
| 無障礙樹快照 | 2.5.3, 4.1.2, 4.1.3, 2.4.4 | `browser_snapshot` |
| 目標大小 | 2.5.8 | `getBoundingClientRect()` 批次量測 |
| 台灣定位點 | TW-01 | Tab 至定位點 + 截圖 |

> 上表依操作類型組織 runtime 檢查，「涵蓋準則」列出各批次的主要目標。單一批次的結果可能同時涉及未列出的相關準則（例如 Tab 序列測試同時觀察 3.2.1 焦點觸發行為）。

##### 4. 合併結果

Runtime 結果覆寫或補充 Step 3a 的靜態初判：
- 靜態 `⚠️ 需 runtime 確認` + runtime ✅ → 最終 ✅ 通過
- 靜態 `⚠️ 需 runtime 確認` + runtime 🔴 → 最終 🔴 失敗
- 靜態 🔴 失敗（純靜態準則）→ 維持 🔴 失敗

##### 錯誤處理

- **axe-core CDN 載入失敗**：跳過 axe 掃描，該項標為 🟡 警告，建議使用者手動用 axe DevTools 檢查
- **Playwright 中途失敗**：已完成的 runtime 結果保留，未完成的項目標註「僅靜態分析，runtime 未驗證」

#### 截圖分析協定

截圖用於兩個目的：**AI 即時判斷**（焦點是否可見、是否被遮蔽、佈局是否正確）+ **報告留存證據**。分析截圖時，重點確認：焦點指示器可見性（2.4.7）、焦點未被 sticky 遮蔽（2.4.11）、320px 無水平溢出（1.4.10）。

### Step 4：產出報告

> **前置條件：Step 3a 和 3b 都已執行完畢。若 3b 未執行，返回執行 3b。**

```markdown
# 無障礙審計報告

**審計日期**：YYYY-MM-DD
**掃描範圍**：[路徑/檔案]
**目標等級**：WCAG 2.2 AA + 台灣在地規範
**審計結果**：X 通過 / Y 警告 / Z 失敗
**驗證覆蓋**：靜態分析 X 條 / runtime 驗證 Y 條 / 僅靜態（runtime 未驗證）Z 條

---

## 📊 統計摘要

| 原則 | 通過 | 警告 | 失敗 |
|------|------|------|------|
| 可感知 | X | X | X |
| 可操作 | X | X | X |
| 可理解 | X | X | X |
| 穩健性 | X | X | X |
| 台灣特有 | X | X | X |
| **合計** | **X** | **X** | **X** |

## 🔴 失敗（必須修復）

### [F001] 準則編號 — 問題摘要
- **檔案**：`path/to/file.tsx:行號`
- **問題**：具體描述
- **影響**：對使用者的影響
- **驗證方式**：靜態分析 / runtime 驗證 / 靜態 + runtime
- **修復**：具體修復方式
- **修復類型**：🔧 可自動修復 / 🎨 需確認（視覺變更）/ 📝 需手動處理
- **修復風險**：🟢 低（純屬性）/ 🟡 中（CSS/佈局）/ 🔴 高（框架 API/架構）
- **優先級**：🔴 高 / 🟡 中 / 🟢 低
- **證據**：[runtime 截圖/結果，僅靜態分析時省略]

## 🟡 警告（建議修復）

### [W001] ...

## 🟢 通過

- ✅ 準則編號：通過描述

## 📋 台灣特有規範

| 項目 | 狀態 | 說明 |
|------|------|------|
| TW-01 定位點（:::） | ✅/🔴 | ... |
| TW-02 Access Key | ✅/🔴 | ... |
| TW-03 網站導覽 / 3-Tab | ✅/🔴 | ... |
| TW-04 Landmark Roles | ✅/🔴 | ... |
| TW-05 Freego 相容性 | ✅/🔴 | ... |
```

### Step 5：修復

報告產出後，詢問使用者是否要修復。

#### 5a. 專案架構探索（修復前必做，同一次審計只做一次）

在動手修任何東西之前，先理解專案的架構慣例：

1. **技術棧偵測**：讀取 `package.json`，識別框架（Next/Nuxt/Vue/Angular/Svelte/Astro）、CSS 方案（Tailwind/Sass/CSS Modules/styled-components）、元件庫（Radix/MUI/Headless UI/Ant Design）
2. **設計系統探索**：找到色彩/間距/字型的 token 定義位置（`tailwind.config.*`、`:root` CSS 變數、SCSS 變數、theme 檔案）。若專案無設計系統，記錄此事實
3. **既有解法搜尋**：對每項修復，Grep 專案中類似問題的現有處理方式

> **原則：修復必須是專案既有架構的自然延伸。用專案已有的方式解決問題。用框架原生 API。找不到既有方式時，參考 `references/patterns.md`。**

#### 5b. 修復執行

**第一批：🔧 可自動修復** — 批次執行，不需逐項確認。

**第二批：🎨 需確認（視覺變更）** — 一次列出所有 🎨 項目的修復計畫：

每項須包含：修復方式、**視覺影響**（改動後畫面上什麼會不同）、影響範圍、修復風險等級。

對每項提供選項：
- **A. 現在修復**（修完後在 5c 摘要中記錄視覺變更）
- **B. 先跳過**（記錄於報告，本次不執行）

使用者可逐項選擇，也可批次回覆「全部 A」或「全部 B」。

**第三批：📝 需手動處理** — 僅提供修復指引。

#### 5c. 修復品質審查（修復後、再驗證前必做）

回顧本次所有修復，以下列原則逐一審查。若任一項不通過，先修正再進入 Step 6。

1. **根因優先**：每項修復是否改動了問題的源頭（定義層），而非在消費端重複修補？若同一類修復散佈 3 個以上檔案，視為警示信號，應考慮是否有更上游的單點可改。

2. **抽象層一致**：新增或修改的樣式值、變數、class 是否透過專案既有的抽象層（設計 token、CSS 變數、共用元件）引入？是否繞過了專案已建立的間接層？

3. **框架原生優先**：修復是否使用了框架提供的原生機制？是否引入了框架已有對應方案的低階 DOM 操作或瀏覽器 API？

4. **跨狀態有效**：修復在所有 viewport、互動狀態（hover/focus/active）、顯示條件（responsive breakpoint、條件渲染）下是否仍然有效？是否有功能仍活躍但修復元素被隱藏的情況？

5. **全域完整**：Grep 全專案，確認同一類問題的所有實例都已處理，而非只修了掃描範圍內的檔案。

審查通過後，若本次有任何 🎨 修復被執行，輸出視覺變更摘要：

| 報告編號 | 準則 | 視覺變更描述 | 影響範圍 |
|---------|------|------------|---------|
| F003 | 2.4.7 | 焦點外框從 2px → 3px | 全站互動元素 |
| ... | ... | ... | ... |

### Step 6：修復後再驗證（自動執行）

> **此步驟為必要步驟。Step 5 修復完成後立即自動執行，不需使用者再次下指令。**

1. 僅重新檢查 Step 4 報告中 🔴 失敗與 🟡 警告的準則
2. 重新執行 Step 3a 靜態分析（僅失敗項）
3. 重新執行 Step 3b runtime 驗證（僅失敗項，若開發伺服器可用）
4. 產出差異表格（修復前 → 修復後 → 驗證方式）
5. 若發現回歸問題，立即通知使用者並暫停

> 使用者可要求 `/a11y-tw` 完整重掃（full re-scan），此時重新執行 Step 1–4。

### 收尾：清理截圖

審計流程結束後（無論是 Step 4 報告產出後使用者不修復，或 Step 6 再驗證完成後），執行：

1. Glob 搜尋工作目錄下的 `a11y-*.png` 與 `page-*.png`
2. 若有找到檔案，列出檔案清單與總數量，詢問使用者是否要刪除
3. 使用者同意後才刪除，不可自動刪除

#### 仍需人工驗證

以下無法由靜態分析或 Playwright 完成：螢幕閱讀器朗讀（NVDA + Firefox / VoiceOver + Safari）、跨頁一致性（3.2.3 / 3.2.4）、影片字幕/音訊描述（1.2.x）、認知層面（錯誤訊息建議品質、操作指示清晰度）。

---

## 搭配工具建議

本 Skill 的靜態分析 + Playwright 驗證可涵蓋大部分自動化可偵測的問題。
以下工具用於 Skill 無法覆蓋的面向：

| 工具 | 用途 |
|------|------|
| **Freego** | 台灣官方機器檢測（申請標章必用，僅 Windows） |
| **axe DevTools** | 瀏覽器擴充自動偵測 |
| **NVDA + Firefox** | 螢幕閱讀器測試（台灣審查必用） |
| **VoiceOver + Safari** | macOS 螢幕閱讀器測試 |
| **jest-axe** | CI 整合：單元測試中加入 axe-core 檢查 |
| **eslint-plugin-jsx-a11y** | 開發時即時 linting |

> 螢幕閱讀器手動測試步驟見 `README.md`。
