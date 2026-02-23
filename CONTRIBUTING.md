# 貢獻指南

感謝你有興趣參與 a11y-tw！以下說明如何回報問題與提交貢獻。

## 回報問題

1. 先搜尋 [Issues](https://github.com/DopplerKuo/a11y-tw-audit-skill/issues) 確認是否已有相同問題
2. 使用 Issue 模板建立新 issue，盡量提供：
   - 使用的 Claude Code 版本
   - 重現步驟
   - 預期行為 vs 實際行為
   - 相關錯誤訊息或截圖

## 提交 PR

1. Fork 本 repo
2. 建立分支：`git checkout -b feat/your-feature`
3. 修改後確認：
   - SKILL.md 的審計流程邏輯是否一致
   - references/ 下的檢查清單編號是否正確對應
   - README.md 是否需要同步更新
4. Commit message 格式：`feat:` / `fix:` / `docs:` / `chore:`
5. 開 PR 並描述變更內容

## 可以貢獻什麼

- **檢查清單補充**：發現 checklist.md 遺漏的 WCAG 準則或台灣規範
- **元件模式**：在 patterns.md 新增無障礙元件實作範例
- **對比度演算法**：改善 contrast.md 的色彩解析邏輯
- **Bug 修復**：修正 SKILL.md 流程中的邏輯錯誤
- **文件改善**：修正錯字、補充說明、改善可讀性

## 開發注意事項

- 本專案是 Claude Code Skill，沒有可執行的程式碼，主要內容是 Markdown
- 測試方式：在 Claude Code 中實際執行 `/a11y-tw` 驗證流程是否正常
- references/ 下的檔案會被 Skill 在審計過程中按需載入，修改時注意不要破壞既有的標題結構

## 行為準則

請保持友善、尊重的溝通方式。我們歡迎所有程度的貢獻者。
