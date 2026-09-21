---
name: a11y-tw
description: >
  台灣網站無障礙審計工具。用於掃描 React、Next.js、HTML 前端，
  依 WCAG 2.2 AA 與台灣在地規範產出審計報告、分類修復項目，並輔助安全修復。
---

# 台灣無障礙檢測（Codex）

保留 `agents/openai.yaml` 的技能探索設定。本版不使用 Claude hooks，修復安全檢查需依文字流程執行，不能宣稱 Codex 會自動強制執行這些檢查。

## 使用方式

```text
$a11y-tw 幫我檢查所有頁面的無障礙
$a11y-tw src/components/Modal.tsx
$a11y-tw --criteria 1.4.3,2.4.7
$a11y-tw src --exclude "src/admin,**/*.test.tsx"
```

## 執行流程

啟用後先完整讀取安裝目錄下的 `references/workflow.md`，再依其中流程確認範圍、載入適用的檢查清單、執行驗證、產出報告並處理已授權修復。參考文件均位於本 skill 目錄，不要到受審計專案尋找同名文件。

每次修改都須執行共用流程中的修復安全檢查、Step 5c 品質審查及 Step 6 再驗證；工具或伺服器不可用的項目保持未驗證，不可當成通過。套用 `--exclude` 的範圍必須維持至修復與再驗證。
