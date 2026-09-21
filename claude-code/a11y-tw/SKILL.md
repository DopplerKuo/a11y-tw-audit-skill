---
name: a11y-tw
disable-model-invocation: true
argument-hint: "[path] [--criteria ids] [--exclude paths]"
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

# 台灣無障礙檢測（Claude Code）

保留手動呼叫政策及 Claude hooks。Hooks 是輔助防護，不代表修改必然正確。

## 使用方式

```text
/a11y-tw 幫我檢查所有頁面的無障礙
/a11y-tw src/components/Modal.tsx
/a11y-tw --criteria 1.4.3,2.4.7
/a11y-tw src --exclude "src/admin,**/*.test.tsx"
```

## 執行流程

啟用後先完整讀取安裝目錄下的 `references/workflow.md`，再依其中流程確認範圍、載入適用的檢查清單、執行驗證、產出報告並處理已授權修復。參考文件均位於本 skill 目錄，不要到受審計專案尋找同名文件。

每次修改都須執行共用流程中的修復安全檢查、Step 5c 品質審查及 Step 6 再驗證；工具或伺服器不可用的項目保持未驗證，不可當成通過。套用 `--exclude` 的範圍必須維持至修復與再驗證。
