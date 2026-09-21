# 貢獻指南

感謝你有興趣參與 a11y-tw！以下說明如何回報問題與提交貢獻。

## 回報問題

1. 先搜尋 [Issues](https://github.com/DopplerKuo/a11y-tw-audit-skill/issues) 確認是否已有相同問題
2. 使用 Issue 模板建立新 issue，盡量提供：
   - 使用的工具（Claude Code / Codex）與版本
   - 重現步驟
   - 預期行為 vs 實際行為
   - 相關錯誤訊息或截圖

## 提交 PR

1. Fork 本 repo
2. 建立分支：`git checkout -b feat/your-feature`
3. 修改後確認：
   - 兩版 references/workflow.md 與其他共用文件是否一致
   - references/ 下的檢查清單編號是否正確對應
   - README.md 是否需要同步更新
4. Commit message 格式：`feat:` / `fix:` / `docs:` / `chore:`
5. 開 PR 並描述變更內容

## 可以貢獻什麼

- **檢查清單補充**：發現 checklist.md 遺漏的 WCAG 準則或台灣規範
- **元件模式**：在 patterns.md 新增無障礙元件實作範例
- **對比度演算法**：改善 contrast.md 的色彩解析邏輯
- **Bug 修復**：修正共用流程或 Claude Code／Codex 專用設定中的邏輯錯誤
- **文件改善**：修正錯字、補充說明、改善可讀性

## 開發注意事項

- 兩個可直接複製的完整版本分別位於 claude-code/a11y-tw/ 與 codex/a11y-tw/；共同規則須同步至兩版 references/，工具專用設定只放在各自的 SKILL.md 與 agents/openai.yaml
- 行為測試：安裝對應套件，分別以 Claude Code 的 `/a11y-tw` 與 Codex 的 `$a11y-tw` 驗證。檔案測試不能取代 Claude Code、Codex 或瀏覽器的實際操作測試
- references/ 下的檔案會被 Skill 在審計過程中按需載入，修改時注意不要破壞既有的標題結構

## 維護者檢查

- 確認 Claude Code 與 Codex 的 a11y-tw/references/ 內容相同，兩版 a11y-tw-upgrade/references/upgrade.md 也維持一致。
- 將各個 skill 目錄複製到隔離的測試位置，確認必要文件及參考路徑完整；不要用正式安裝測試覆蓋與復原。
- 更新流程需檢查：只讀檢查、內容相同、遠端缺少新目錄、網路失敗、首次無紀錄、本機修改、來源 ref 切換，以及更新中途失敗時兩個目錄與紀錄一起還原。
- 確認遠端快照內容不會被當成操作指令執行，且指定分支尚未提供新結構時不會改裝另一工具的版本。

以上是維護者的文件與行為檢查，一般使用者只需複製目錄。檔案完整性不代表已驗證實際網站的無障礙表現。

## 行為準則

請保持友善、尊重的溝通方式。我們歡迎所有程度的貢獻者。
