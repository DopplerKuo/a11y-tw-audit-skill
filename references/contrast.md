# 色彩對比靜態分析參考

> 本文件供 a11y-tw Skill Step 3「靜態色彩對比分析」使用，提供 CSS 變數解析、色彩轉換與 WCAG 對比度計算的完整演算法。

---

## 1. CSS 變數解析流程

1. 讀取專案 CSS 入口檔（`globals.css` 或等效）
2. 提取 `:root` 與 `@theme` 區塊的所有自訂屬性定義
3. 建立 **變數 → 值** 的對應表
4. 解析 `var()` 引用鏈（最多 3 層），例如：
   ```
   --color-foreground: var(--foreground)
   --foreground: oklch(0.145 0 0)
   → 最終值：oklch(0.145 0 0)
   ```
5. 將所有值正規化為 hex

## 2. 色彩格式轉換

### `#hex` → 直接使用

### `rgb(r, g, b)` / `rgba(r, g, b, a)`
- 提取 R、G、B 通道（0–255）
- `rgba` 中 `a < 1` → 標記為半透明，建議 runtime 驗證

### `oklch(L C H)` → hex

轉換路徑：oklch → OKLab → linear sRGB → sRGB → hex

1. **oklch → OKLab**：
   ```
   a = C × cos(H × π / 180)
   b = C × sin(H × π / 180)
   L = L（不變）
   ```
2. **OKLab → linear sRGB**：
   ```
   l_ = L + 0.3963377774 × a + 0.2158037573 × b
   m_ = L - 0.1055613458 × a - 0.0638541728 × b
   s_ = L - 0.0894841775 × a - 1.2914855480 × b

   l = l_³,  m = m_³,  s = s_³

   R_lin =  4.0767416621 × l - 3.3077115913 × m + 0.2309699292 × s
   G_lin = -1.2684380046 × l + 2.6097574011 × m - 0.3413193965 × s
   B_lin = -0.0041960863 × l - 0.7034186147 × m + 1.7076147010 × s
   ```
3. **linear sRGB → sRGB**：
   ```
   sRGB = linear ≤ 0.0031308
     ? linear × 12.92
     : 1.055 × linear^(1/2.4) - 0.055
   ```
4. **sRGB → hex**：`clamp(0, round(sRGB × 255), 255)` → 轉十六進位

### 半透明色（`rgba` / `oklch` 帶 alpha）
- 標記為「需合成背景，建議 runtime 驗證」
- 若背景色已知且不透明，可用 alpha compositing：`result = fg × alpha + bg × (1 - alpha)`

## 3. Tailwind v4 utility 解析

| Class 模式 | 對應 CSS | 變數查表 |
|------------|---------|---------|
| `text-{name}` | `color: var(--color-{name})` | 查對應表取 hex |
| `bg-{name}` | `background-color: var(--color-{name})` | 查對應表取 hex |
| `border-{name}` | `border-color: var(--color-{name})` | 查對應表取 hex |
| `text-{name}/{opacity}` | 含透明度 | 標記需 runtime |
| `bg-{name}/{opacity}` | 含透明度 | 標記需 runtime |

**查表路徑**：`className` → CSS variable name → 變數對應表 → hex 值

## 4. WCAG 對比度公式

### 相對亮度（Relative Luminance）

```
sRGB_channel = value / 255
linear = sRGB ≤ 0.04045
  ? sRGB / 12.92
  : ((sRGB + 0.055) / 1.055) ^ 2.4
L = 0.2126 × R_linear + 0.7152 × G_linear + 0.0722 × B_linear
```

### 對比度（Contrast Ratio）

```
ratio = (L_lighter + 0.05) / (L_darker + 0.05)
```

其中 `L_lighter` 為較亮色的相對亮度，`L_darker` 為較暗色的相對亮度。

### 門檻值

| 情境 | 最低對比度 | 準則 |
|------|----------|------|
| 一般文字（< 18pt） | ≥ 4.5:1 | 1.4.3 |
| 大字（≥ 18pt 或 ≥ 14pt bold） | ≥ 3:1 | 1.4.3 |
| UI 元件邊框/狀態 | ≥ 3:1 | 1.4.11 |
| 有意義的圖形物件 | ≥ 3:1 | 1.4.11 |

> 大字定義：≥ 18pt (24px) 或 ≥ 14pt (18.66px) bold。

## 5. 靜態可解析 vs 需 runtime

### ✅ 可靜態解析

- 明確 `#hex` / `rgb()` / `oklch()` 色值
- CSS 自訂屬性（`var()` 引用鏈 ≤ 3 層）
- Tailwind utility class（不帶透明度修飾）
- CSS 偽類（`:hover` / `:focus`）中直接宣告的色彩

### ⚠️ 需 runtime 驗證

- JavaScript 動態設定的色彩（如 `style.color = ...`）
- 複雜繼承鏈（跨多檔案、依賴 DOM 結構）
- 半透明色彩合成（背景不確定）
- 第三方 CSS / 外部樣式表
- Tailwind 帶透明度修飾（`text-primary/50`）
