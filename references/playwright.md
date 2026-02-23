# Playwright MCP 無障礙自動驗證參考

> 本文件供 a11y-tw Skill Step 3 Hybrid 掃描使用。掃描各準則時依下方「準則對應表」同步執行對應檢查。

---

## 前置條件

- Playwright MCP 工具可用（`browser_navigate` 等）
- 開發伺服器已啟動（預設 `http://localhost:3000`）
- 連線方式：`browser_navigate` → 確認頁面載入成功
- 連線失敗則跳過所有 runtime 驗證，在報告中註記

## 準則 → Runtime 檢查對應表

| 準則 | Runtime 檢查 | 章節 |
|------|-------------|------|
| 綜合（掃描開始時執行） | axe-core 全面掃描 | §1 |
| 1.4.3 / 1.4.11 | Computed Style 對比度 | §2 |
| 1.4.10 | 響應式重排 | §8 |
| 2.1.2 | 鍵盤困陷 | §4 |
| 2.4.3 | Tab 順序 | §4 |
| 2.4.7 | 焦點可見性 | §5 |
| 2.4.11 | 焦點遮蔽 | §6 |
| 2.5.8 | 目標大小 | §7 |
| 4.1.2 / 4.1.3 | 無障礙樹驗證 | §3 |
| TW-03 | 3-Tab 導覽 | §9 |

## 1. axe-core 全面掃描

使用 `browser_run_code` 注入 axe-core 並執行：

```js
async (page) => {
  await page.addScriptTag({
    url: 'https://cdnjs.cloudflare.com/ajax/libs/axe-core/4.10.3/axe.min.js'
    // ⚠️ 版本固定於 4.10.3（cdnjs 最新）。更新前需確認新版規則集與本 Skill checklist 不衝突。
  });
  return await page.evaluate(() => window.axe.run());
}
```

**解讀**：`violations` 陣列中的每個 item 包含 `id`（規則名）、`impact`（等級）、`nodes`（問題元素）。將結果對應至 checklist 準則，合併至主報告。

## 2. Computed Style 對比度驗證

使用 `browser_evaluate` 取得所有文字元素的 computed 前景/背景色：

```js
() => {
  const els = document.querySelectorAll(
    'p,span,div,h1,h2,h3,h4,h5,h6,a,button,label,li,td,th,dt,dd,summary,legend,figcaption,blockquote'
  );
  return Array.from(els).map(el => {
    const s = getComputedStyle(el);
    return {
      tag: el.tagName,
      text: el.textContent?.substring(0, 30),
      color: s.color,
      bg: s.backgroundColor,
      fontSize: s.fontSize,
      fontWeight: s.fontWeight
    };
  });
}
```

拿到 `rgb()` 值後，用 `references/contrast.md` 的公式計算對比度。此步驟可驗證靜態分析的結果，並補足無法靜態解析的色彩組合。

## 3. 無障礙樹驗證

使用 `browser_snapshot` 取得完整 accessibility tree。

**檢查項目**：
- 缺少 accessible name 的互動元素（按鈕、連結、表單控制項）
- 缺少 role 的自訂元件
- `aria-expanded` / `aria-selected` 狀態是否正確反映 UI 狀態

## 4. Tab 順序驗證

連續 `browser_press_key("Tab")`，每次之間 `browser_snapshot` 記錄 focused 元素。最多按 **50 次 Tab** 或偵測到焦點循環（回到已訪問過的元素）即停止，避免無限迴圈。

**確認**：
- 順序符合視覺邏輯
- 無鍵盤困陷（同一元素連續出現 2 次以上）
- Modal 中額外按 `browser_press_key("Escape")` 確認可關閉
- 若 50 次 Tab 後仍未完成循環，報告為 🟡 警告（Tab 序列過長）

## 5. 焦點可見性驗證

Tab 至各元素 → `browser_take_screenshot` → 確認有可見的 focus outline。

逐一檢查：
1. 按 Tab 移動焦點
2. 截圖記錄焦點外觀
3. 確認 outline 或其他焦點指示器清楚可見

## 6. 焦點遮蔽驗證（2.4.11）

Tab 至 sticky header/footer 附近的元素 → `browser_take_screenshot` → 確認焦點元素未被遮蓋。

特別注意：
- Sticky/fixed header 下方的元素
- Cookie banner 覆蓋區域
- Floating action button 附近

## 7. 目標大小驗證（2.5.8）

使用 `browser_evaluate` 取得所有互動元素的 bounding rect：

```js
() => {
  const els = document.querySelectorAll(
    'a,button,input,select,textarea,[role="button"],[tabindex]'
  );
  return Array.from(els)
    .map(el => ({
      tag: el.tagName,
      text: el.textContent?.substring(0, 20),
      w: el.getBoundingClientRect().width,
      h: el.getBoundingClientRect().height
    }))
    .filter(r => r.w < 24 || r.h < 24);
}
```

回傳的元素均為尺寸不足 24×24 CSS px 的互動目標。

## 8. 響應式重排驗證（1.4.10）

```
browser_resize(320, 768)
→ browser_evaluate(() => document.documentElement.scrollWidth > 320)
```

若回傳 `true` → 存在水平溢出 → 失敗。搭配 `browser_take_screenshot` 留存證據。

驗證完畢後 `browser_resize` 恢復原始大小。

## 9. 3-Tab 導覽驗證（TW-03）

從頁頂開始：

1. `browser_press_key("Tab")` × 3
2. 每次 Tab 後 `browser_snapshot` 記錄焦點位置
3. 確認第 3 次 Tab 焦點已到達導覽連結

若 3 次 Tab 後焦點仍未到達導覽連結 → 失敗。

## 10. 截圖分析指引

截圖由 AI 即時分析，兼作報告證據。

### 焦點可見性（2.4.7）
- 確認元素周圍有清楚可辨的 outline/border/shadow
- 若 outline 與背景色對比不足 → 🔴 失敗
- 若完全無焦點指示器 → 🔴 失敗

### 焦點遮蔽（2.4.11）
- 確認焦點元素未被 sticky/fixed 元素「完全」覆蓋
- AA 允許部分遮蔽，完全遮蔽 → 🔴 失敗

### 響應式重排（1.4.10）
- 320px 寬度下確認無水平捲動條
- 文字未被截斷或溢出容器
- 互動元素仍可操作

### 截圖命名慣例
- 格式：`a11y-{criterion}-{description}.png`
- 範例：`a11y-2.4.7-focus-button-submit.png`、`a11y-1.4.10-reflow-320px.png`
