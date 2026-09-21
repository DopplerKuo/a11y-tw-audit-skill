# 台灣網站無障礙檢測 Skill：Claude Code / Codex

![GitHub Release](https://img.shields.io/github/v/release/DopplerKuo/a11y-tw-audit-skill?label=Release)
![License: MIT](https://img.shields.io/badge/License-MIT-yellow)

a11y-tw 結合靜態分析與瀏覽器驗證，依 WCAG 2.2 AA 與台灣在地檢查項目產出報告並輔助修復。Claude Code 與 Codex 使用相同的流程、60 條檢查清單及元件範例，分別提供 Claude Code 與 Codex 專用版本。

本專案為社群開發的**非官方工具**，與 Anthropic、OpenAI、W3C、數位發展部均無隸屬或背書關係。

## 選擇版本

|              | Claude Code                 | Codex                             |
| ------------ | --------------------------- | --------------------------------- |
| 呼叫方式     | `/a11y-tw`                  | `$a11y-tw`                        |
| 要複製的目錄 | `claude-code/a11y-tw/`      | `codex/a11y-tw/`                  |
| 修復防護     | Claude hooks + 共用安全檢查 | 共用安全檢查，不使用 Claude hooks |
| 觸發設定     | 手動呼叫                    | 可明確呼叫，也保留自動探索設定    |
| 全域安裝位置 | `~/.claude/skills/a11y-tw`  | `~/.codex/skills/a11y-tw`         |

兩版的修復後再驗證要求相同。文字流程需由代理遵循，不代表 Codex 會執行 Claude hooks。

## 安裝：直接複製目錄

### Claude Code

將 repo 裡的 **`claude-code/a11y-tw/`** 複製到本機 **`~/.claude/skills/`**：

```text
~/.claude/skills/a11y-tw/
├── SKILL.md
├── references/
├── README.md
└── LICENSE
```

使用 `/a11y-tw` 呼叫。

### Codex

將 repo 裡的 **`codex/a11y-tw/`** 複製到本機 **`~/.codex/skills/`**：

```text
~/.codex/skills/a11y-tw/
├── SKILL.md
├── agents/openai.yaml
├── references/
├── README.md
└── LICENSE
```

使用 `$a11y-tw` 呼叫。

`~` 是你的使用者家目錄。若 `skills` 資料夾不存在，先建立它；兩版都已包含完整參考文件，不需要複製 repo 內其他目錄，也不需要連結到另一版。請保留 `a11y-tw` 這層目錄，避免複製成 `skills/SKILL.md` 或 `a11y-tw/a11y-tw/SKILL.md`。

更新時以新目錄取代原有的 `a11y-tw`，避免混入舊檔案。若之前曾裝在其他技能搜尋位置（例如 `~/.agents/skills/a11y-tw`）或專案內，請移除重複版本，避免載入到不同內容。安裝後開啟新工作階段，確認能找到技能。

瀏覽器驗證需要可操作瀏覽器的工具及能啟動的目標專案；相關執行環境依工具與專案本身需求準備。沒有瀏覽器工具或伺服器無法啟動時，仍可提供靜態結果，runtime 項目會明確標成未驗證。

## 檢查與更新技能（選用）

另外提供 `a11y-tw-upgrade`，讓 Claude Code 或 Codex 幫你比較官方 repo 與本機的檔案，並依指示更新。它是獨立 skill，由 AI 使用現有 Git 與檔案工具執行流程，不是背景自動更新程式；使用時需要 Git 及網路。

安裝方式同樣是直接複製，放在 `a11y-tw` 旁邊：

| 使用工具 | 額外複製的目錄 | 放到本機的位置 |
|---|---|---|
| Claude Code | `claude-code/a11y-tw-upgrade/` | `~/.claude/skills/a11y-tw-upgrade/` |
| Codex | `codex/a11y-tw-upgrade/` | `~/.codex/skills/a11y-tw-upgrade/` |

先檢查（不修改本機檔案）：

```text
/a11y-tw-upgrade --check
```

```text
$a11y-tw-upgrade --check
```

要求更新：

```text
/a11y-tw-upgrade --update
```

```text
$a11y-tw-upgrade --update
```

不加參數也只會檢查。更新預設使用官方 repo 的預設分支，會一併更新同一安裝位置的 `a11y-tw` 與 `a11y-tw-upgrade`。若需要指定分支或 tag，可加 `--ref <名稱>`；不會自動切到開發分支。

偵測到本機修改，或第一次使用而無法辨識哪些差異是本機客製內容時，會先列出差異並詢問是否覆蓋。流程會記錄成功更新的來源及檔案指紋，供下次比對；不會自行更新其他專案或全域設定。更新成功後請開啟新工作階段。

**指定的遠端分支必須已有對應的兩個完整 skill 目錄。** 若功能尚未合併到預設分支，檢查會說明尚不支援並保留本機內容；不會把原本只有 Claude Code 的舊版結構直接覆蓋到 Codex。

## 使用方式

在 Claude Code 中輸入：

```text
/a11y-tw 幫我檢查所有頁面的無障礙
/a11y-tw src/components/Modal.tsx
/a11y-tw --criteria 1.4.3,2.4.7
/a11y-tw src --exclude "src/admin,**/*.test.tsx"
```

在 Codex 中輸入：

```text
$a11y-tw 幫我檢查所有頁面的無障礙
$a11y-tw src/components/Modal.tsx
$a11y-tw --criteria 1.4.3,2.4.7
$a11y-tw src --exclude "src/admin,**/*.test.tsx"
```

`--exclude` 接受專案根目錄相對路徑或 glob，多項以逗號分隔，掃描、import 展開、修復與再驗證都維持排除範圍。Skill 會先提供報告，再依授權及修復類型處理；視覺變更需列明影響。

## 功能與限制

- 範圍確認 → 靜態分析 → 瀏覽器驗證 → 報告 → 修復 → 再驗證。
- 60 個既有檢查條目：WCAG 2.2 AA 55 條與台灣在地 5 條。
- 動態圖片及 CMS 替代文字、複雜圖表等價說明、實際前景／背景對比。
- 焦點裁切、手機／縮放後導覽、modal 與非 modal 的鍵盤行為。
- 必填說明、iframe／按鈕名稱與輪播暫停控制。
- 無 JavaScript 情境只在專案明示要求時另行檢查，不新增通用 TW-06 失敗項。

**通過本工具的檢查不等於符合無障礙規範。** 自動化及 AI 判讀不能取代螢幕閱讀器測試、真人使用者測試或專業稽核；缺乏 runtime 證據的項目不計入通過。本工具不提供法律或合規保證。

### 螢幕閱讀器手動測試步驟

1. 用 Tab 走完互動元素，確認順序、焦點可見性與退出方式。
2. 測試 Access Key 是否到達對應區塊。
3. 操作 Modal、Tab、Accordion，確認狀態變更被朗讀。
4. 觸發表單驗證，確認錯誤訊息及焦點處理。
5. 確認控制項名稱、動態訊息及跨頁導覽一致性。

可搭配 Freego、axe DevTools、NVDA + Firefox、VoiceOver + Safari；工具通過不代表已完成所有人工檢查。

## 目錄與維護

```text
a11y-tw-audit-skill/
├── claude-code/
│   ├── a11y-tw-upgrade/     # 選用：檢查與更新技能
│   └── a11y-tw/             # 可直接複製的 Claude Code 版
│       ├── SKILL.md
│       ├── references/      # 流程、清單與範例
│       ├── README.md
│       └── LICENSE
└── codex/
    ├── a11y-tw-upgrade/     # 選用：檢查與更新技能
    └── a11y-tw/             # 可直接複製的 Codex 版
        ├── SKILL.md
        ├── agents/openai.yaml
        ├── references/      # 相同的流程、清單與範例
        ├── README.md
        └── LICENSE
```

兩版各自附上完整的 `references/`，選擇對應目錄即可直接複製使用。修改規則與執行測試的說明，請見 [貢獻指南](CONTRIBUTING.md)。

## 技術文件

以下連到 Claude Code 版的文件；Codex 版內容相同。

| 文件                                                          | 內容                          |
| ------------------------------------------------------------- | ----------------------------- |
| [workflow.md](claude-code/a11y-tw/references/workflow.md)     | 共用流程、修復安全檢查與報告  |
| [checklist.md](claude-code/a11y-tw/references/checklist.md)   | 60 個檢查條目與台灣實務       |
| [contrast.md](claude-code/a11y-tw/references/contrast.md)     | 色彩解析與對比計算            |
| [patterns.md](claude-code/a11y-tw/references/patterns.md)     | 互動元件、動態內容與 SSR 指引 |
| [playwright.md](claude-code/a11y-tw/references/playwright.md) | 瀏覽器驗證與證據限制          |

## 參考資料

- [數位發展部無障礙規範 110.07](https://accessibility.moda.gov.tw/Accessible/Guide/68)
- [PDIS Freego Implementation Report](https://github.com/PDIS/Freego-Implementation-Report)
- [WCAG 2.2 Quick Reference](https://www.w3.org/WAI/WCAG22/quickref/?versions=2.2&levels=aaa)
- [WAI-ARIA Authoring Practices](https://www.w3.org/WAI/ARIA/apg/)

## Contributors

<table>
  <tr>
    <td align="center">
      <a href="https://github.com/dopplerkuo">
        <img src="https://wsrv.nl/?url=github.com%2Fdopplerkuo.png&amp;w=128&amp;h=128&amp;fit=cover&amp;mask=circle&amp;output=png" width="64" height="64" alt="dopplerkuo 的 GitHub 頭像"><br>
        <strong>@dopplerkuo</strong>
      </a><br>
      開發與維護
    </td>
    <td align="center">
      <a href="https://github.com/youxiaaa">
        <img src="https://wsrv.nl/?url=github.com%2Fyouxiaaa.png&amp;w=128&amp;h=128&amp;fit=cover&amp;mask=circle&amp;output=png" width="64" height="64" alt="youxiaaa 的 GitHub 頭像"><br>
        <strong>@youxiaaa</strong>
      </a><br>
      Review 與測試回饋
    </td>
  </tr>
</table>

## 授權

本專案採用 [MIT License](LICENSE) 授權。
