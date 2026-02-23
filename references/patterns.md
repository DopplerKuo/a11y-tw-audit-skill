# React / Next.js 無障礙元件模式

> 每個模式都經過台灣無障礙審計實務驗證。遵循這些模式可避免常見扣分點。

## ARIA 使用原則

> **No ARIA is better than bad ARIA.** — W3C WAI

1. **優先用原生 HTML**：`<button>` 自帶 `role="button"`，不需額外標註。
2. **台灣審計例外**：TW-04 要求顯式標註 landmark role（如 `<header role="banner">`），為向後相容舊版輔助技術。
3. **不改變原生語意**：`<button role="heading">` ❌ — 需要標題就用 `<h3>`。
4. **互動元素必須可操作**：`role="button"` 的 `<div>` 必須有 `tabIndex={0}` + `onKeyDown`。
5. **`<a role="button">` 會改變語意**（不再被朗讀為連結），慎用。

## 1. Hover & Focus 對比度 (1.4.11)

**關鍵要求**：互動元件在 `hover` 和 `focus` 狀態下的顏色變化，其與背景的對比度仍需滿足 3:1。

```css
/* ✅ 確保 hover/focus 狀態的對比度 */
.button-primary {
  background-color: #0055CC; /* vs white = 7.1:1 ✅ */
  color: white;
}

.button-primary:hover,
.button-primary:focus {
  background-color: #003380; /* vs white = 12.5:1 ✅ */
  outline: 2px solid #002255; /* 焦點外框 */
}

/* ❌ 常見錯誤：hover 顏色過淺 */
.button-bad:hover {
  background-color: #80BFFF; /* vs white = 1.8:1 ❌ */
}
```

**檢查工具**：
- [WebAIM Contrast Checker](https://webaim.org/resources/contrastchecker/)
- Chrome / Edge DevTools 內建的色彩選擇器

## 2. Modal / Dialog 彈窗

**關鍵要求**：開啟時 trap focus、關閉時 focus 回到觸發按鈕。

```tsx
import { useRef, useEffect, useCallback, useState, useId } from 'react';

// ✅ 使用方式：由父元件管理 triggerRef
function ModalExample() {
  const [isOpen, setIsOpen] = useState(false);
  const triggerRef = useRef<HTMLButtonElement>(null);

  return (
    <>
      <button
        ref={triggerRef}
        onClick={() => setIsOpen(true)}
        title="開啟聯絡我們彈跳視窗"    {/* ✅ 台灣審計要求 */}
      >
        聯絡我們
      </button>
      <Modal
        isOpen={isOpen}
        onClose={() => setIsOpen(false)}
        triggerRef={triggerRef}
        title="聯絡我們"
      >
        <p>表單內容...</p>
      </Modal>
    </>
  );
}

// Modal 元件：接收 triggerRef 以便關閉時 focus 回去
function Modal({ isOpen, onClose, triggerRef, title, children }) {
  const modalRef = useRef<HTMLDivElement>(null);
  const titleId = useId(); // ✅ 動態 ID 避免多 Modal 衝突

  useEffect(() => {
    if (isOpen) {
      // ✅ 開啟時 focus 到 modal 容器
      modalRef.current?.focus();
    } else if (triggerRef?.current) {
      // ✅ 關閉時 focus 回到觸發按鈕
      triggerRef.current.focus();
    }
  }, [isOpen, triggerRef]);

  const handleKeyDown = useCallback((e: React.KeyboardEvent) => {
    // ✅ Esc 鍵關閉 modal
    if (e.key === 'Escape') {
      onClose();
    }
    // ✅ Focus trap: Tab 在 modal 內循環
    if (e.key === 'Tab') {
      const focusable = modalRef.current?.querySelectorAll(
        'button, [href], input, select, textarea, [contenteditable], audio[controls], video[controls], iframe, details, summary, [tabindex]:not([tabindex="-1"])'
      );
      if (!focusable?.length) return;
      const first = focusable[0] as HTMLElement;
      const last = focusable[focusable.length - 1] as HTMLElement;
      if (e.shiftKey && document.activeElement === first) {
        e.preventDefault();
        last.focus();
      } else if (!e.shiftKey && document.activeElement === last) {
        e.preventDefault();
        first.focus();
      }
    }
  }, [onClose]);

  if (!isOpen) return null;

  return (
    <>
      <div className="modal-overlay" aria-hidden="true" onClick={onClose} />
      <div
        ref={modalRef}
        role="dialog"
        aria-modal="true"
        aria-labelledby={titleId}
        tabIndex={-1} // ✅ 讓 modal 容器可以被 focus
        onKeyDown={handleKeyDown}
      >
        <h2 id={titleId}>{title}</h2>
        {children}
        <button onClick={onClose}>關閉</button>
      </div>
    </>
  );
}
```

## 3. 自訂互動元件 Switch (4.1.2)

**關鍵要求**：使用 `role` 定義元件類型，並用 `aria-checked` 反映狀態，同時支援鍵盤操作。

```tsx
import { useId } from 'react';

function Switch({ label, checked, onChange }) {
  const labelId = useId();

  const handleClick = () => {
    onChange(!checked);
  };

  const handleKeyDown = (e: React.KeyboardEvent) => {
    if (e.key === ' ' || e.key === 'Enter') {
      e.preventDefault();
      onChange(!checked);
    }
  };

  return (
    <div className="switch-container">
      <span id={labelId}>{label}</span>
      <div
        role="switch"              // ✅ 定義 role
        aria-checked={checked}     // ✅ 反映狀態
        aria-labelledby={labelId}  // ✅ 關聯顯式標籤
        tabIndex={0}               // ✅ 讓元件可以被 focus
        onClick={handleClick}
        onKeyDown={handleKeyDown}
      >
        <span className="switch-handle" />
      </div>
    </div>
  );
}
```

## 4. Tab 頁籤

```tsx
function Tabs({ tabs }) {
  const [activeIndex, setActiveIndex] = useState(0);

  const handleKeyDown = (e: React.KeyboardEvent, index: number) => {
    let newIndex = index;
    if (e.key === 'ArrowRight') newIndex = (index + 1) % tabs.length;
    if (e.key === 'ArrowLeft') newIndex = (index - 1 + tabs.length) % tabs.length;
    if (e.key === 'Home') newIndex = 0;
    if (e.key === 'End') newIndex = tabs.length - 1;
    if (newIndex !== index) {
      e.preventDefault();
      setActiveIndex(newIndex);
    }
  };

  return (
    <div>
      <div role="tablist" aria-label="內容分頁">
        {tabs.map((tab, i) => (
          <button
            key={tab.id}
            role="tab"
            id={`tab-${tab.id}`}
            aria-selected={i === activeIndex}       {/* ✅ 台灣審計必查 */}
            aria-controls={`panel-${tab.id}`}
            tabIndex={i === activeIndex ? 0 : -1}   {/* roving tabindex */}
            onKeyDown={(e) => handleKeyDown(e, i)}
            onClick={() => setActiveIndex(i)}
          >
            {tab.label}
          </button>
        ))}
      </div>
      {tabs.map((tab, i) => (
        <div
          key={tab.id}
          role="tabpanel"
          id={`panel-${tab.id}`}
          aria-labelledby={`tab-${tab.id}`}
          hidden={i !== activeIndex}
          tabIndex={0}
        >
          {tab.content}
        </div>
      ))}
    </div>
  );
}
```

## 5. Accordion / 折疊選單

> **替代方案**：若僅需簡單的展開/收合行為，優先使用原生 `<details>` / `<summary>` 元素，無需 JavaScript 即有完整無障礙支援。以下自訂模式適用於需要統一控制（如一次只展開一個）或複雜動畫的場景。

```tsx
function Accordion({ items }) {
  const [openIndex, setOpenIndex] = useState<number | null>(null);

  return (
    <div>
      {items.map((item, i) => (
        <div key={item.id}>
          <h3 id={`heading-${item.id}`}>
            <button
              aria-expanded={i === openIndex}     {/* ✅ 台灣審計必查 */}
              aria-controls={`panel-${item.id}`}
              onClick={() => setOpenIndex(i === openIndex ? null : i)}
            >
              {item.title}
            </button>
          </h3>
          <div
            id={`panel-${item.id}`}
            role="region"
            aria-labelledby={`heading-${item.id}`}
            hidden={i !== openIndex}
          >
            {item.content}
          </div>
        </div>
      ))}
    </div>
  );
}
```

## 6. 下載按鈕

```tsx
// ✅ 台灣審計要求：aria-label 包含檔名、副檔名、檔案大小
<a
  href="/files/report-2026.pdf"
  download
  aria-label="下載 2026年度報告.pdf（檔案大小：2.5MB）"
>
  <IconDownload aria-hidden="true" />
  下載報告
</a>

// 多個下載按鈕時，每個都要有獨立的 aria-label
<a href="/files/q1.xlsx" download
   aria-label="下載 Q1財報.xlsx（檔案大小：1.2MB）">Q1 財報</a>
<a href="/files/q2.xlsx" download
   aria-label="下載 Q2財報.xlsx（檔案大小：1.4MB）">Q2 財報</a>
```

## 7. 表單驗證（完整模式）

```tsx
function ContactForm() {
  const [errors, setErrors] = useState<Record<string, string>>({});
  const formRef = useRef<HTMLFormElement>(null);

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    const newErrors = validate(formData);
    setErrors(newErrors);

    if (Object.keys(newErrors).length > 0) {
      // ✅ focus 到第一個錯誤欄位（台灣審計必查）
      const firstErrorField = formRef.current?.querySelector(
        '[aria-invalid="true"]'
      ) as HTMLElement;
      firstErrorField?.focus();

      // ✅ 通知螢幕閱讀器有錯誤
      announceError(`表單有 ${Object.keys(newErrors).length} 個錯誤，請修正後重新送出`);
    }
  };

  return (
    <form ref={formRef} onSubmit={handleSubmit} noValidate>
      <div>
        <label htmlFor="name">
          姓名 <span aria-hidden="true">*</span>
        </label>
        <input
          id="name"
          type="text"
          required
          aria-required="true"
          aria-invalid={!!errors.name}
          aria-describedby={errors.name ? 'name-error' : undefined}
          autoComplete="name"
        />
        {errors.name && (
          <p id="name-error" role="alert" className="error-text">
            {errors.name}
          </p>
        )}
      </div>

      {/* ✅ 密碼欄位支援密碼管理器（3.3.8 無障礙驗證） */}
      <div>
        <label htmlFor="password">密碼</label>
        <input
          id="password"
          type="password"
          autoComplete="current-password"
          // ❌ 不要加 onPaste={(e) => e.preventDefault()}
        />
      </div>

      {/* ✅ 驗證碼欄位支援 SMS 自動填入（3.3.8） */}
      <div>
        <label htmlFor="otp">驗證碼</label>
        <input
          id="otp"
          type="text"
          inputMode="numeric"
          autoComplete="one-time-code"
          // ❌ 不要拆成多個 input 且禁止貼上
        />
      </div>

      {/* fieldset 群組範例 */}
      <fieldset>
        <legend>偏好聯絡方式</legend>
        <label>
          <input type="radio" name="contact" value="email" />
          電子郵件
        </label>
        <label>
          <input type="radio" name="contact" value="phone" />
          電話
        </label>
      </fieldset>

      <button type="submit">送出</button>
    </form>
  );
}

// Live region 通知工具（SSR 安全）
function announceError(message: string) {
  if (typeof document === 'undefined') return;
  const el = document.getElementById('a11y-announcer');
  if (el) el.textContent = message;
}

// 在 layout 中放置 announcer
// <div id="a11y-announcer" role="status" aria-live="polite" className="sr-only" />
```

## 8. 資料表格

```tsx
<table>
  <caption>2025 年各季營收比較</caption>     {/* ✅ Freego 會檢查 */}
  <thead>
    <tr>
      <th scope="col">季度</th>
      <th scope="col">營收（萬元）</th>
      <th scope="col">成長率</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th scope="row">Q1</th>              {/* 列標題用 scope="row" */}
      <td>1,200</td>
      <td>+5%</td>
    </tr>
  </tbody>
</table>
```

## 9. Toast / 狀態訊息

```tsx
// ✅ 不需 focus 即可被螢幕閱讀器讀到（4.1.3）
<div role="status" aria-live="polite">
  {toastMessage && <p>{toastMessage}</p>}
</div>

// 錯誤級別用 aria-live="assertive"
<div role="alert" aria-live="assertive">
  {errorMessage && <p>{errorMessage}</p>}
</div>
```

## 10. 圖片輪播 / Carousel

```tsx
<div role="region" aria-label="精選圖片" aria-roledescription="carousel">
  <div aria-live="polite">
    <div role="group" aria-label={`第 ${current + 1} 張，共 ${total} 張`}
         aria-roledescription="slide">
      <img src={slides[current].src} alt={slides[current].alt} />
    </div>
  </div>
  <button aria-label="上一張" onClick={prev}><IconChevronLeft /></button>
  <button aria-label="下一張" onClick={next}><IconChevronRight /></button>
  {/* 自動播放必須有暫停按鈕 (2.2.2) */}
  <button aria-label={paused ? '播放' : '暫停'} onClick={togglePause}>
    {paused ? <IconPlay /> : <IconPause />}
  </button>
</div>
```

## 11. Skip Link + Sticky 元素共存（2.4.11）

```css
/* Tailwind 已內建 sr-only，若無 Tailwind 則手動定義 */
.sr-only {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  white-space: nowrap;
  border-width: 0;
}

.sr-only.focus\:not-sr-only:focus {
  position: absolute;
  width: auto;
  height: auto;
  padding: 1rem;
  margin: 0;
  overflow: visible;
  clip: auto;
  white-space: normal;
  z-index: 9999;
  background: white;
  color: black;
  font-weight: bold;
  text-decoration: underline;
}

/* ✅ 2.4.11 焦點未被遮蔽：為 sticky 元素留出 scroll padding */
html {
  scroll-padding-top: 80px;    /* header 高度 */
  scroll-padding-bottom: 60px; /* footer 高度 */
}

main {
  padding-bottom: 80px; /* 為 sticky footer 留空間 */
}
```

## 12. Next.js 特定注意事項

```tsx
// app/layout.tsx - 根 layout 設定
export default function RootLayout({ children }) {
  return (
    <html lang="zh-Hant">                    {/* ✅ 3.1.1 */}
      <head>
        <meta name="viewport" content="width=device-width, initial-scale=1" />
      </head>
      <body>
        {/* Live region announcer（全站共用） */}
        <div id="a11y-announcer" role="status" aria-live="polite" className="sr-only" />

        {/* Skip links */}
        <a href="#main-content" className="sr-only focus:not-sr-only ...">
          跳到主要內容
        </a>

        <header role="banner">...</header>
        <main id="main-content" role="main" tabIndex={-1}>
          {children}
        </main>
        <footer role="contentinfo">...</footer>
      </body>
    </html>
  );
}

// ⚠️ Next.js App Router 路由切換不會自動通知螢幕閱讀器
// 需要在 layout 或 middleware 中處理 route change announcement
// 推薦使用 next-route-announcer 或自訂 useRouteAnnouncer hook
```

```tsx
// 動態頁面標題（Next.js App Router）
// app/about/page.tsx
export const metadata = {
  title: '關於我們 - 網站名稱',    // ✅ 2.4.2 描述性標題
};

// 或動態標題
export async function generateMetadata({ params }) {
  const product = await getProduct(params.id);
  return {
    title: `${product.name} - 產品詳情 - 網站名稱`,
  };
}
```

## 13. 台灣定位點（:::）+ Access Key + 3-Tab

台灣政府網站必須設置「網頁導盲磚」，以三個冒號 `:::` 為視覺標記，搭配鍵盤快速鍵。

```tsx
{/* 定位點完整實作 */}
<body>
  {/* Access Key 說明（螢幕閱讀器可讀） */}
  <div className="sr-only">
    <p>本網站使用鍵盤快速鍵：</p>
    <ul>
      <li>Alt+U：上方功能區塊</li>
      <li>Alt+C：中央內容區塊</li>
      <li>Alt+Z：下方功能區塊</li>
    </ul>
  </div>

  {/* 上方功能區塊定位點 */}
  <a id="U" href="#header" accessKey="U" title="上方功能區塊"
     className="sr-only focus:not-sr-only">:::</a>
  <header id="header" role="banner">
    <nav role="navigation" aria-label="主要導覽">
      {/* 導覽內容 */}
    </nav>
  </header>

  {/* 中央內容區塊定位點 */}
  <a id="C" href="#main-content" accessKey="C" title="中央內容區塊"
     className="sr-only focus:not-sr-only">:::</a>
  <main id="main-content" role="main" tabIndex={-1}>
    {/* 主要內容 */}
  </main>

  {/* 下方功能區塊定位點 */}
  <a id="Z" href="#footer" accessKey="Z" title="下方功能區塊"
     className="sr-only focus:not-sr-only">:::</a>
  <footer id="footer" role="contentinfo">
    {/* 頁尾內容 */}
  </footer>
</body>
```

### Access Key 對應表

| 快速鍵 | 功能 | 必要性 |
|--------|------|--------|
| Alt+U | 上方功能區塊（header） | 必要 |
| Alt+C | 中央內容區塊（main） | 必要 |
| Alt+Z | 下方功能區塊（footer） | 必要 |
| Alt+S | 網站搜尋 | 建議 |
| Alt+L | 左側導覽 | 若有左側選單則建議 |

### 注意事項

- Access Key 在不同瀏覽器/OS 的啟動方式不同：Windows Chrome 是 `Alt+Key`，Mac 是 `Ctrl+Option+Key`
- `:::` 是台灣特有的視覺標記，國際上無此慣例
- 定位點在一般使用者看不到（sr-only），但 focus 時應可見
- `id` 值用大寫字母（U, C, Z）是台灣慣例

### 3-Tab Skip Links

從任何頁面，鍵盤使用者在 **3 次 Tab 內**必須能到達導覽/Access Key 說明：

```tsx
{/* 頁面最頂端的 skip links，確保 3 Tab 可達 */}
<div className="skip-links">
  <a href="#main-content" className="sr-only focus:not-sr-only">
    跳到主要內容
  </a>
  <a href="/sitemap" className="sr-only focus:not-sr-only">
    網站導覽
  </a>
  <a href="/accessibility" className="sr-only focus:not-sr-only">
    無障礙說明
  </a>
</div>
```

**3-Tab 手動測試步驟**：
1. 重新整理頁面，點擊網址列。
2. 按 `Tab` 鍵第一次，焦點應落在「跳到主要內容」的 Skip Link 上。
3. 按 `Tab` 鍵第二次，焦點應落在「網站導覽」的 Skip Link 上。
4. **結果**：2 次 Tab 內可達，符合規範。

## 14. Landmark Roles 顯式標註

現代語意 HTML5 元素自帶 landmark role，但為確保向後相容（台灣審計要求），建議明確標註：

```tsx
<header role="banner">       {/* 頁首，每頁一個 */}
<nav role="navigation">       {/* 導覽，可多個但需 aria-label 區分 */}
<main role="main">            {/* 主要內容，每頁一個 */}
<aside role="complementary">  {/* 側欄/補充內容 */}
<footer role="contentinfo">   {/* 頁尾，每頁一個 */}
<section role="region">       {/* 需有 aria-label 或 aria-labelledby */}
<form role="form">            {/* 需有 aria-label */}
```

多個同類型 landmark 時，用 `aria-label` 區分：
```tsx
<nav role="navigation" aria-label="主要導覽">...</nav>
<nav role="navigation" aria-label="頁尾導覽">...</nav>
```

## 15. 目標大小與拖曳替代（2.5.7 / 2.5.8）

```css
/* ✅ 2.5.8 確保最小點擊區域 24×24px */
button, a, [role="button"], input[type="checkbox"], input[type="radio"] {
  min-width: 24px;
  min-height: 24px;
}

/* ✅ 小 icon 按鈕用 padding 撐大點擊區域 */
.icon-button {
  padding: 8px; /* icon 16px + padding 8px×2 = 32px */
}

/* ✅ 緊密排列的元素之間留足間距 */
.button-group button + button {
  margin-left: 8px;
}
```

```tsx
// ✅ 2.5.7 可排序列表：拖曳 + 按鈕替代
function SortableList({ items, onReorder }) {
  return (
    <ul>
      {items.map((item, index) => (
        <li key={item.id} draggable onDragStart={...} onDrop={...}>
          <span>{item.name}</span>
          {/* ✅ 單點替代方式 */}
          <button
            aria-label={`將 ${item.name} 上移`}
            onClick={() => moveUp(index)}
            disabled={index === 0}
          >
            ↑
          </button>
          <button
            aria-label={`將 ${item.name} 下移`}
            onClick={() => moveDown(index)}
            disabled={index === items.length - 1}
          >
            ↓
          </button>
        </li>
      ))}
    </ul>
  );
}
```

## 16. Breadcrumb 麵包屑導覽

常見扣分：缺少 `nav` + `aria-label`、當前頁面未標記。

```tsx
function Breadcrumb({ items }: { items: { label: string; href?: string }[] }) {
  return (
    <nav aria-label="麵包屑導覽">
      <ol>
        {items.map((item, i) => {
          const isLast = i === items.length - 1;
          return (
            <li key={i}>
              {isLast ? (
                // ✅ 當前頁面用 aria-current="page"
                <span aria-current="page">{item.label}</span>
              ) : (
                <>
                  <a href={item.href}>{item.label}</a>
                  <span aria-hidden="true"> / </span>
                </>
              )}
            </li>
          );
        })}
      </ol>
    </nav>
  );
}
```

**要點**：
- 外層 `<nav aria-label="麵包屑導覽">` 讓螢幕閱讀器識別為導覽區域
- 使用 `<ol>` 表達順序關係（非 `<ul>`）
- 最後一項用 `aria-current="page"` 標記當前位置，不設連結
- 分隔符號用 `aria-hidden="true"` 避免被朗讀

## 17. Date Picker 日期選擇器

**最佳做法**：優先用原生 `<input type="date">`，瀏覽器內建無障礙支援。

```tsx
// ✅ 最簡單且最無障礙的做法
<label htmlFor="birth-date">出生日期</label>
<input
  id="birth-date"
  type="date"
  autoComplete="bday"
  aria-describedby="date-hint"
/>
<p id="date-hint">格式：年/月/日</p>
```

**若必須自訂**，關鍵要求：允許直接鍵入日期、日曆面板用 `role="dialog"` + `aria-modal="true"`、格線用 `role="grid"`、方向鍵移動焦點、Enter 選取、Esc 關閉、選取日期 `aria-selected="true"`、今天 `aria-current="date"`、關閉後 focus 回觸發元素。

---

## SSR / CSR 注意事項

React/Next.js 的 SSR 與 CSR 在無障礙方面有關鍵差異，審計時需特別注意：

### Hydration 無障礙風險

- **狀態不一致**：伺服器渲染的 HTML 可能缺少客戶端才有的 ARIA 狀態（如 `aria-expanded`）。Hydration 完成前，AT 讀到的狀態可能不正確。
- **事件未綁定**：Hydration 完成前，鍵盤事件（`onKeyDown`）尚未綁定，互動元件無法操作。對關鍵互動元件考慮延遲顯示（搭配 `useEffect` 在 mount 後設定 ARIA 狀態），或確保伺服器與客戶端初始狀態一致。
- **`document` / `window` 存取**：在 Server Component 或 SSR 階段呼叫 `document` 會拋錯。所有 DOM API 呼叫須加守衛：`if (typeof document === 'undefined') return;`

### Next.js App Router vs Pages Router

| 面向 | App Router | Pages Router |
|------|-----------|-------------|
| 路由播報 | 無內建播報，需自行實作 `useRouteAnnouncer` 或 live region | 內建 `next/router` events 可搭配播報 |
| `<head>` 管理 | `export const metadata` / `generateMetadata` | `next/head` 元件 |
| Server Component | 預設為 Server Component，不可使用 hooks / event handlers | 全部為 Client Component |

### 審計要點

1. 檢查是否有元件在 SSR 階段直接使用 `document`、`window`、`localStorage`
2. 確認互動元件標記為 `'use client'`，否則鍵盤事件不會生效
3. 路由切換時是否有 live region 通知螢幕閱讀器（特別是 App Router）
4. Hydration 不一致是否造成 ARIA 屬性閃爍（伺服器端與客戶端值不同）

---

## 第三方元件庫審計指引

使用 MUI、Ant Design、Radix、Headless UI 等元件庫時，審計策略有所不同：

### 一般原則

1. **信任但驗證**：主流元件庫通常有基礎無障礙支援，但不保證符合台灣特有需求
2. **聚焦配置層**：檢查開發者傳入的 props 是否正確（如 `aria-label`、`role`），而非元件庫內部實作
3. **DOM 輸出為準**：不論元件庫如何封裝，最終以渲染出的 HTML/ARIA 為審計對象

### 常見需注意項目

- **Landmark 顯式標註（TW-04）**：部分元件庫的 Layout 元件不輸出顯式 `role` 屬性，可能需要手動補上以滿足 Freego 檢測
- **定位點/Access Key（TW-01/02）**：元件庫不會包含台灣特有的 `:::` 定位點與 Access Key，需額外實作
- **焦點管理**：元件庫的 Modal/Drawer/Popover 通常自帶 focus trap，驗證其行為是否符合 WAI-ARIA APG 即可，不需重新實作
- **主題色對比度**：使用元件庫的主題系統時，確認自訂色彩仍滿足 WCAG 對比度要求
