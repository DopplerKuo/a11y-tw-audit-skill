# a11y-tw

本目錄是可獨立安裝的完整 skill，保留 SKILL.md、references/ 與本目錄內的其他檔案即可。

Claude Code 版放到 `~/.claude/skills/a11y-tw`，以 `/a11y-tw` 呼叫；Codex 版放到 `~/.codex/skills/a11y-tw`，以 `$a11y-tw` 呼叫。請依下載的版本選擇位置。

## 螢幕閱讀器手動測試步驟

1. 用 Tab 走完互動元素，確認順序、焦點可見性與退出方式。
2. 測試 Access Key 是否到達對應區塊。
3. 操作 Modal、Tab、Accordion，確認狀態變更被朗讀。
4. 觸發表單驗證，確認錯誤訊息及焦點處理。
5. 確認控制項名稱、動態訊息及跨頁導覽一致性。

可搭配 NVDA + Firefox 或 VoiceOver + Safari。自動化結果不能取代真人測試與完整無障礙稽核。
