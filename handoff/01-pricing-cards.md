# Prompt 1 — Pricing cards area (hero, tabs, toolbar, plan cards)

> 給 RD：把整份檔案（從下一行起）貼給 AI coding 工具（Claude Code / Cursor 等），並附上 `assets/` 內列出的 SVG。
> 參考實作：repo 根目錄的 `index.html`（單檔 HTML/CSS/JS），所有數值以它為準。

---

You are updating the **pricing page** of our Next.js site (`/pricing`). Rebuild the area **between the site header and the "Compare All Plans" section** so it matches the spec below exactly — every size, color, state and animation. Adapt the markup to our existing components / CSS modules, but do not change any values. Leave the global site header as it is.

A complete working reference is in `index.html` (search for the class names used below). When this spec and the reference disagree, the reference wins.

## 0. Global rules

- Fonts: **Roboto** 400/600/700/800/900 (body) and **Wix Madefor Display** 400/600/800 (display titles: page title, plan names, discount badge).
- Use our design tokens (`design-tokens.css`). Resolved values are listed so you can check them:

  | Token | Value |
  |---|---|
  | `--fill-brand-strong` (brand) | `#03ade2` |
  | `--fill-brand-weak` | `#03ade20d` |
  | `--stroke-brand-weak` | `#03ade233` |
  | `--fill-hover` | `#11181a0a` |
  | `--fill-weak` | `#11181a0a` |
  | `--fill-weaker` | `#11181a05` |
  | `--fill-disabled` | `#11181a33` |
  | `--fill-strong` | `#09161af2` |
  | `--fill-inverse-strong` | `#ffffff` |
  | `--fill-badge` | `#f23b77` |
  | `--icon-error` | `#de0000cc` |
  | `--stroke-strong` | `#11181a33` |
  | `--stroke-weak` | `#11181a1a` |
  | `--stroke-inverse-strong` | `#ffffff99` |
  | `--text-strong` | `#09161af2` |
  | `--text-weak` | `#11181aa6` |
  | `--text-weaker` | `#11181a73` |
  | `--text-inverse-weak` | `#ffffffc7` |
  | `--font-size-tiny` / `--line-height-tiny` | `0.875rem` / `1.25rem` (14px / 20px) |

- **Orange gradient** (used by the Pro CTA, the PRO tag and the slider): `linear-gradient(88.46deg, #ffb802 0%, #ffa51c 32.565%, #ff8800 99.797%)`.
- **Hover rule:** every text link and ghost button gets a **gray** hover (`--fill-hover`), never a blue one. Filled buttons use `filter: brightness(1.05)` on hover.
- **Focus ring:** `:focus-visible { outline: 2px solid var(--fill-brand-strong); outline-offset: 2px; }` — brand blue, never the browser's orange.
- **Never scroll sideways:** `html, body { overflow-x: clip; }` (fall back to `hidden` without `overflow: clip` support). Hidden tooltips must be `display: none`, not `visibility: hidden`, so they never widen the page.
- **Reduced motion:** every animation and transition below has a `prefers-reduced-motion: reduce` fallback (no motion, final state).
- **Card shadows** use `box-shadow`, not `filter: drop-shadow` — iOS Safari tiles tall filtered elements and shows a white band.

## 1. Page container and hero

- `.page`: `max-width: 1440px; margin: 0 auto; padding: 0 83px 64px;` flex column, centered.
- `.hero`: `padding: 64px 110px 0; gap: 32px;` flex column, centered.
- **Title (h1):** Wix Madefor Display, 800, 40px / 40px, letter-spacing −0.5px, `#212529`, centered.
  - Subscription tab: **"Choose the Right Plan for you"**
  - Pay As You Go tab: **"Choose Your Perfect Pack"**
  - Both texts sit in **one grid cell** (`display: grid` + `grid-area: 1/1`) and **crossfade** (`opacity` + `visibility`, `.25s ease`) on tab change, so the height never jumps.

## 2. Main tabs (Subscription / Pay As You Go)

- Container: `width: 580px; max-width: 100%; background: #f0f0f0; border-radius: 47px; padding: 8px;` flex. Gap below the tabs is 20px.
- Each tab: equal width, `padding: 16px; border-radius: 41px;` Roboto 600, 24px / 32px, `#000`. The active tab's text is `#fff`, with `transition: color .3s`.
- **Sliding pill indicator:** a single pseudo-element sits behind the buttons and moves under the active tab.
  - JS measures the active button and sets `--ind-x / --ind-y / --ind-w / --ind-h`.
  - Transition: `transform .3s cubic-bezier(.4,0,.2,1), width .3s cubic-bezier(.4,0,.2,1)`.
  - No transition on first paint: add a `ready` class one frame after the first measurement.
  - Re-measure on resize (ResizeObserver) and after fonts load.
  - Pill fill: `border-radius: 41px; background-image: linear-gradient(0deg, var(--stroke-brand-weak) 10.937%, rgba(0,104,215,0.2) 100%), linear-gradient(90deg, #03ade2, #03ade2);`
- ARIA: `role="tablist"`, `role="tab"`, `aria-selected`.
- On tab change, the new tab's cards ease in with `@keyframes panel-in { from { opacity: 0; transform: translateY(8px); } }` and `.3s ease-out`.
- **The selected currency is kept when switching tabs.** Never reset it.

## 3. Row under the tabs (shares one grid cell, crossfades like the title)

### 3a. Subscription → Billing toggle (Monthly / Yearly)

- Container: `background: #f0f0f0; border-radius: 12px; padding: 4px;` flex.
- Buttons:
  - Layout: `min-width: 120px; padding: 8px 12px; border-radius: 8px; gap: 6px;`
  - Text: Roboto 600, 18px / 26px, `#212529`; the active button uses `#15171f`.
  - ARIA: `role="radiogroup"` on the container, `role="radio"` + `aria-checked` on each button.
- Uses the same sliding-pill technique as the tabs. Pill: `background: #fff; border-radius: 8px; box-shadow: 2px 2px 2px rgba(124,124,124,0.25);`
- **Default is Yearly.**
- "Yearly" carries a **Save badge**: "Save 60%".
  - Style: `background: #f23b77; border: 1px solid #ffffff99; border-radius: 32px; padding: 2px 6px;` 600, 14px / 20px, white.
  - Percentage: `round(1 − proYearly[0] / (proMonthly[0] × 12))` = 1 − 95.99 / 239.88 → **60**.

### 3b. Pay As You Go → Subtitle

- Text: **"Pay-as-you-go credit pack could only be used in limited Generative-AI tools."** followed by an info icon (`icon-info.svg`, 16.815px, flipped with `transform: scaleY(-1)`).
- Wix Madefor Display, 18px / 26px, `--text-strong`, centered, gap 5px.
- The icon is a link with a 3px round hit area that turns `--fill-hover` on hover.
- **Click:** smooth-scroll to the **Compare All Plans** section (`#compare`), offset by the sticky site header height (64px). Use an instant jump under reduced motion.

## 4. Toolbar (currency + API link)

- `.plans` section: `padding: 20px 0 40px; gap: 20px;`
- Toolbar: full width, `justify-content: space-between`. It has 3 children: an **invisible copy of the API link** (`visibility: hidden`, `aria-hidden`, keeps the currency dropdown centered), the **currency dropdown**, then the **"See API Pricing →"** link.
- **See API Pricing** → `https://yce.perfectcorp.com/ai-api/api-pricing`.
  - Style: 600, 18px / 26px, brand blue, height 42px, `padding: 0 12px; margin: 0 -12px; border-radius: 32px;`
  - Arrow: `arrow-right.svg`, 20px.
  - Hover: gray pill background, and the arrow slides `translateX(3px)` over `.2s`.
- **Currency dropdown button:** `106×32px; border: 1px solid #cacce0; border-radius: 6px; padding: 8px 12px; background: #fff;` 13px, `#000`. Inside, in order:
  - The code, 55px wide with ellipsis.
  - A 1×16px separator, `#cacce0`.
  - `chevron.svg`, 7×16px.
- **Menu:**
  - Placement: absolute, `top: calc(100% + 4px)`, full button width.
  - Box: `max-height: 196px` (about 6 items, the rest scroll); `padding: 4px 0; border: 1px solid #dcdde3; border-radius: 8px; box-shadow: 0 6px 16px rgba(0,0,0,0.08);`
  - Items: `padding: 6px 16px;` 13px / 20px, `#212529`. Selected item background `#e6f6fc`; hover `--fill-hover`.
  - Scrollbar: 6px wide, transparent track, thumb `--stroke-weak` with radius 3. Use `scrollbar-width: thin` in Firefox.
  - ARIA: `role="listbox"` / `role="option"` + `aria-selected`; the button has `aria-haspopup` / `aria-expanded`.
  - Closing: outside click closes it. `Esc` closes it and returns focus to the button. Opening scrolls the selected item into view.
- **Currencies (in this order):** USD, GBP, CAD, AUD, TWD, HKD, JPY, KRW, EUR, CHF, MXN, CLP, BRL, TRY, THB, IDR, RUB, CNY, VND, PLN. Default: **USD**.
- **Price formatting:**
  - Symbol + amount with en-US grouping, e.g. `$1,199.99`, `NT$2,300`.
  - TWD / JPY / KRW / CLP / IDR / VND show **no decimals** on totals.
  - Per-credit prices **always** show 2 decimals.
  - A **letter prefix** before the symbol (NT, HK, A, C, MX, CLP, R, CN, Rp, CHF) is wrapped in `<span class="cur">` and shrunk: 24px inside big prices, 12px inside pack prices.
  - The reference uses placeholder FX rates. **Use our real per-currency price tables.**

## 5. Cards row (both tabs)

- `.cards`: flex, `gap: 32px; align-items: stretch; justify-content: center;`
- `.card-col`: `flex: 1 1 0; padding-top: 20px;` (room for the corner badge), `position: relative`. Paid columns: `min-width: 360px; max-width: 420px;`
- `.card`:
  - Box: `background: #fff; border-radius: 32px; padding: 32px;` flex column, `gap: 14px` (Subscription cards use `gap: 16px` and stretch to equal height).
  - Shadow: `box-shadow: 0 12px 15px rgba(0,0,0,0.25)`; the Pro card uses `0 12px 30px rgba(0,0,0,0.25)`.
- **Plan title block:** `padding: 12px 0; gap: 4px;` centered.
  - Plan name (Free card): Wix Madefor Display 600, 20px / 28px.
  - Price row: flex, baseline-aligned, centered, `gap: 6px`, wraps.
  - **Price:** Roboto 800, 56px / 64px, letter-spacing −1px, `tabular-nums`, `#191919`. The **Pro price is `--text-strong` (black)**, never blue or orange.
  - **/mo:** 600, 22px / 28px, same color as its price.
  - **Crossed-out price:** 600, 24px / 32px, `--text-weaker`, `padding: 0 4px`. A 2px line image (`cross-line.svg`) is absolutely centered over it at full width. When this element wraps onto its own line, add `.is-wrapped` to the row (JS check on resize / fonts ready) and pull it up with `margin-top: -12px`.
- **Buttons (`.btn`):**
  - Base: full width, height 48px, `border-radius: 32px; padding: 0 28px;` Roboto 600, 20px / 28px.
  - Active: `transform: scale(0.98)`. Transitions: `filter / transform / background-color .15s`.
  - Outline: `border: 1px solid var(--stroke-strong); background: rgba(255,255,255,0.01); color: --text-strong;` hover `--fill-hover`.
  - Primary: brand-blue background, white text, hover `brightness(1.05)`.
  - Pro: orange gradient background, white text, hover `brightness(1.05)`.
- **Feature list:**
  - List: `padding: 8px; gap: 12px;` no bullets.
  - Item: flex, top-aligned, `gap: 8px;` 600, 16px / 24px, `--text-strong`.
  - Icon: 20px (`circletick.svg`, blue `#03ADE2`), nudged down with `margin-top: 2px`.
  - Highlight words `<b class="hl">` are 600, brand blue.
- **Tier tags:**
  - **PLUS:** Roboto 700, 20px, line-height 1, white, `padding: 3.5px 8px 2.5px; border-radius: 100px; opacity: .8;` `background-image: linear-gradient(31deg, #44e3ed 2.43%, #03ade2 67.21%)`.
  - **PRO:** Roboto 900, 20px, line-height 1, white, `padding: 2.5px 12px 1.5px; border-radius: 22px; opacity: 1;` `background-image: linear-gradient(89.37deg, #ffb802 0%, #ffa51c 32.565%, #ff8800 99.797%), linear-gradient(180deg, #ffc700 0%, #f7a300 100%)`.

## 6. Subscription tab — the three cards

Desktop order is Free · Plus · Pro, left to right. On mobile the order is reversed (see §9).

### Free
- Plan name "Free Plan"; price `$0` (no /mo).
- Outline button **"Start for Free"**.
- Features: "Include Starter **FREE** Credits" · "Download Images with Watermarks" · "No Credit Card Needed" · "512 MB Cloud Storage".
- Then the AI Agent box (§6.4).

### Plus
- PLUS tag; price `$3.33` /mo when Yearly, `$9.99` /mo when Monthly.
- Primary button **"Get Plus"**.
- Features: "**100 Credits** / per month" · "Watermark-free, High-quality Downloads" · "Batch Photo Editing" · "10 GB Cloud Storage".
- Then the AI Agent box.

### Pro (orange theme)
- PRO tag, then price + /mo + (crossed-out price, only when discounted).
- Pro button **"Get Pro Pack"**.
- Credit slider box (§7).
- Features, using **orange tick icons** (`circletick-orange.svg`, `#FF8800`) and **highlights in `#ea7d00`**:
  - "**{N} Credits** / per month" — N follows the slider: 300, 800, 1,500…
  - "Watermark-free, High-quality Downloads"
  - "Batch Photo Editing"
  - "100 GB Cloud Storage"
  - "**Daily FREE** AI Agent Chats for Quick Edits"
  - "**40% OFF** Additional Credit Purchases"
  - "Commercial Use Rights Included"
- Then the AI Agent box. **Its links stay brand blue.**
- **Card background** (light orange glow from the top-left that fades to white along a diagonal):
  ```css
  background-color: #fff;
  background-size: 100% 100%;
  background-image:
    linear-gradient(130deg, rgba(255,255,255,0) 15%, #fff 52%),
    url("data:image/svg+xml;utf8,<svg viewBox='0 0 380 608' xmlns='http://www.w3.org/2000/svg' preserveAspectRatio='none'><rect x='0' y='0' height='100%' width='100%' fill='url(%23grad)' opacity='0.15'/><defs><radialGradient id='grad' gradientUnits='userSpaceOnUse' cx='0' cy='0' r='10' gradientTransform='matrix(38.033 33.421 -24.429 27.256 -0.33454 4.7205)'><stop stop-color='%23ffe94d' offset='0'/><stop stop-color='%23ffd23f' offset='0.25'/><stop stop-color='%23ffb63a' offset='0.5'/><stop stop-color='%23ff9b45' offset='0.75'/><stop stop-color='%23ff8a5c' offset='1'/></radialGradient></defs></svg>");
  ```

### 6.4 AI Agent box (bottom of each Subscription card)
- Box: `background: var(--fill-weaker); border-radius: 16px; padding: 12px 20px;` **no border**.
- Two rows, each `min-height: 32px`, flex, `gap: 4px`. The name is 600, 14px / 20px, `--text-strong`, no wrap.
  1. **"YouCam AI Agent"** with the link **"Usage details ↓"** (`arrow-down.svg`, 16px).
     - **Click:** smooth-scroll to the **AI Agent** group of Compare All Plans (`#cmp-ai-agent`). If that group is collapsed, expand it first.
     - Scroll offset = sticky top (64px) + height of the sticky plan header + 16px, so the group title lands just under both sticky bars.
  2. **"AI Agent Connector" ⓘ** with the link **"Explore ↗"** (`external-link.svg`, 16px). Opens `https://yce.perfectcorp.com/products/youcam-for-ai-assistants` in a new tab (`rel="noopener"`).
- Links:
  - Style: brand blue, 600, 14px / 20px, underlined (`text-underline-position: from-font`).
  - Box: height 32px, `padding: 0 12px 0 16px; border-radius: 32px;` icon 16px with `margin-left: 4px`.
  - Hover: gray pill (`--fill-hover`).
- **ⓘ button:** 12px icon (`ic-info.svg`), 4px padding with −4px margin (hit area), round. It turns `--fill-hover` on hover and while open (`aria-expanded="true"`).
- **Connector tooltip** (YCO "Tooltip / Large"): **click** toggles it.
  - Position: above the row, `bottom: calc(100% + 8px)`, `width: min(324px, 100%)`.
  - Box: `padding: 8px 12px; border-radius: 8px; background: var(--fill-strong); box-shadow: 0 8px 24px -4px rgba(0,0,0,0.16);`
  - Text: 14px / 20px, `--text-inverse-weak`. Title: white 600, 4px below.
  - Arrow: 8px, pointing down, its x set from JS to the icon's center (`--arrow-x`).
  - Content: title **"Create directly in your AI assistant"**; body "Connect YouCam AI Agent to ChatGPT, Claude, and Gemini to create and edit visual content through conversation."
  - Closing: opening one closes the others; an outside click or `Esc` closes it.

### 6.5 Holiday discount (config flag, Pro only)
- Config `holidayDiscount`: `0` = off (**default / current**), `0.3` = 30% off.
- When > 0:
  - Discounted price = `floor(round(list × 100) × (1 − d)) / 100`, i.e. whole cents rounded down. Example: $7.99 → **$5.59**.
  - The regular price is shown crossed out after /mo.
  - The corner badge shows **"30% off"**.
- **The price stays black when discounted.**
- **Corner badge:**
  - Placement: absolute in the Pro column, `top: 2px; left: calc(50% + 111.83px); transform: translateX(-50%);` `z-index: 1`. At ≤600px: `left: auto; right: 20px; transform: none`.
  - Box: `background: #de0000cc; border: 1px solid #ffffff99; border-radius: 32px; padding: 4px 12px;`
  - Text: white, Wix Madefor Display 800, 18px / 28px, no wrap.
- The prototype has a floating "Discount On / Off" switch (`?discount=on`) for review only. **Do not ship it.**

### 6.6 Subscription prices (USD, from the PM table)

| Pro credits / month | 300 (default) | 800 | 1,500 | 2,500 | 5,000 | 10,000 |
|---|---|---|---|---|---|---|
| Monthly ($/mo) | 19.99 | 34.99 | 59.99 | 89.99 | 164.99 | 319.99 |
| Yearly, billed once | 95.99 | 249.99 | 429.99 | 649.99 | 1,199.99 | 2,399.99 |
| **Yearly shown as $/mo** | **7.99** | **20.83** | **35.83** | **54.17** | **99.99** | **199.99** |

Plus: Monthly $9.99 · Yearly $39.99 (shown as **$3.33**/mo). Free: $0.

Re-render the prices whenever the slider, billing period, currency or discount changes.

## 7. Pro credit slider (orange)

### 7.1 Box
- `display: flex; flex-direction: column; gap: 8px; padding: 32px 16px 12px; border: 2px solid rgba(255,136,0,0.2); border-radius: 16px; background: rgba(255,136,0,0.05);`
- The 32px top padding leaves room for the tooltip, which pokes **~8px above the box border** and stays **8px clear of the Get Pro Pack button**.
- Children: the slider row (`padding-top: 6px`), then the label row. There is **no** separate credit-count row under the slider; the tooltip shows the count.

### 7.2 Track
- Stops: **300 · 800 · 1,500 · 2,500 · 5,000 · 10,000**, default **300**. The handle **snaps** to stops.
- Track element: height 26px, `margin: -6px 0` (the handle overflows the 14px row); `touch-action: none; cursor: pointer;`
  - ARIA: `role="slider"`, `tabindex=0`, `aria-label="Credit amount"`, `aria-valuemin/max/now`, `aria-valuetext="1,500 credits"`.
- Stop positions: stop *i* (of n = 6) sits at `calc(8px + (100% − 16px) × i/(n−1))`, stored as `--pos`.
- Bar: 6px high, vertically centered, radius 19px, inset 4px on both sides, `--fill-disabled`.
- Progress: same shape, from 4px to `--pos`, **orange gradient**. `transition: width .18s ease` except while dragging.
- Handle: `handle-orange.svg`, 26×26, centered on `--pos`, `z-index: 1`.
  - The SVG is a white dot (r = 5) inside an orange ring (stroke `#FF8800`, opacity .8, width 8, r = 9).
  - `transition: left .18s ease, transform .15s` (no `left` transition while dragging).
  - `scale(1.08)` on hover / drag.
  - Keyboard focus ring: `box-shadow: 0 0 0 3px rgba(255,136,0,.35)` on a round handle.
- **Stop dots:**
  - 10px circles centered on each stop. Default fill `#b9b9b9` with a `0 0 0 1px rgba(255,255,255,0.6)` ring.
  - Passed stops (before the handle): `#ff8800`, no ring.
  - The current stop and the first stop are hidden (opacity 0).
  - `transition: background-color/box-shadow/opacity .18s ease`.

### 7.3 Labels
- One button per stop, under the track: 12px / 16px, `--text-weaker`. The active label and hovered labels are `--text-strong`.
- Widths: first label 36px, left-aligned; last label 37px, right-aligned; the middle labels share the rest equally and are centered.
- Clicking a label selects that stop. Labels have `tabindex=-1`; the track owns keyboard input.

### 7.4 Interaction
- **Pointer:** pointerdown on the track captures the pointer, adds `.dragging`, and jumps to the nearest stop, computed as `round(clamp((x − left − 8)/(width − 16), 0, 1) × (n−1))`. pointermove updates while captured; pointerup / pointercancel removes `.dragging`.
- **Keyboard:** ← / ↓ = previous stop; → / ↑ = next stop; Home = first; End = last.

### 7.5 Tooltip on the handle (always visible)
- Style: Figma "Tooltip / Type: Top centre".
- Content: `coin.svg` (16px) + "**300 credits**", gap 4px. It updates on every change.
- Position: absolute, `bottom: calc(100% + 6px); left: var(--pos); transform: translateX(-50%);` `z-index: 3`. It follows the handle with `transition: left .18s ease` (none while dragging). The arrow stays centered; do **not** clamp the tooltip to the box edges.
- Box: `display: inline-flex; padding: 8px 12px; border-radius: 8px; background: #fff;` no pointer events.
- Text: `--text-strong`, 14px / 20px, 600, no wrap.
- Shadow: `filter: drop-shadow(0 8px 12px rgba(0,0,0,0.16))`, so one shadow wraps both bubble and arrow.
- Arrow: a 6px CSS triangle under the center, white.

### 7.6 Swipe hint animation (finger + gradient trail)
- **Trigger:** the first time the track is ≥60% visible (IntersectionObserver, threshold 0.6), wait **400ms**, then add `.hinting` to the track.
- **Finger:** `finger-pointer.svg`, **42×42px**, white-filled hand.
  - Placement: absolute, `z-index: 2; top: 12px; margin-left: -15.75px` (puts the fingertip, at ~37.5% of the icon width, on the stop).
  - `filter: drop-shadow(0 2px 3px rgba(0,0,0,0.2)); opacity: 0;` no pointer events.
  - Icon credit: iconpacks.net #10507.
- **Animation:** `swipe-hint 2.4s ease-in-out`, **3 iterations**; one round trip is one play. Each play:
  1. The finger appears at 300 and "presses" (scale .92).
  2. It swipes to 10,000.
  3. It fades out there.
  4. It returns to the start invisibly.
  ```css
  @keyframes swipe-hint {
    0%   { opacity: 0; left: 8px;               transform: scale(1); }
    10%  { opacity: 1; left: 8px;               transform: scale(0.92); }
    60%  { opacity: 1; left: calc(100% - 8px);  transform: scale(0.92); animation-timing-function: linear; }
    70%  { opacity: 0; left: calc(100% - 8px);  transform: scale(1); }
    100% { opacity: 0; left: 8px; }
  }
  ```
- **Gradient trail** behind the finger:
  - Element: same shape as the bar (6px, radius 19, left 4px), `z-index: 1`, so it sits **above the dots** and below the handle.
  - Fill: `linear-gradient(90deg, rgba(255,136,0,0) 0%, rgba(255,165,28,.25) 40%, #ff8800 100%)`.
  - Same timing as the finger:
  ```css
  @keyframes hint-trail {
    0%   { opacity: 0; width: 4px; }
    10%  { opacity: 1; width: 4px; }
    60%  { opacity: 1; width: calc(100% - 12px); animation-timing-function: linear; }
    70%  { opacity: 0; width: calc(100% - 12px); }
    100% { opacity: 0; width: 4px; }
  }
  ```
- **Dots under the trail:** while hinting, a `requestAnimationFrame` loop hides each stop dot (opacity 0, with the .18s fade) once the trail's right edge passes the dot's center **and** the trail opacity is > 0.4. When the trail fades out, the dots come back. Clear everything when hinting ends.
- **Stop:** remove `.hinting` on the finger's `animationend`, or immediately on any `pointerdown` / `keydown` on the track.
- Reduced motion: no hint at all.

### 7.7 Pro price roll (when the price changes)
- Applies to the Pro price whenever its text changes (slider, billing, discount).
- Compare the old and new strings **right-aligned** (so decimals line up). Unchanged characters stay still. Each changed character becomes a 1-line-tall clipping box (`display: inline-block; height: 1lh; overflow: hidden; vertical-align: top`) holding a column: **[new char, old char]**.
- **Slide down:** the column animates `translateY(-50%) → 0` over **400ms ease-in-out**, so the new digit drops in from above while the old one exits downward.
- Mark the old character `aria-hidden="true"`.
- If the new string is longer, the missing old characters are blank (`&nbsp;`).
- No animation on first render, or when the letter prefix changes (e.g. switching to NT$). Under reduced motion, show the final value with no animation.

## 8. Pay As You Go tab

Cards, left to right: Free account · Standard Pack · Pro Exclusive Pack. On mobile the order is reversed (§9).

### Free account
- Plan name "Free account"; price `$0` in the selected currency (no decimals).
- Outline button **"Sign up free"**.
- Features: "Sign up for free credits" · "Save image with watermark" · "No credit card needed".

### Standard Pack
- Plan name "Standard Pack"; price = total of the selected pack.
- Primary button **"Buy now"**.
- **Tools box:**
  - Box: `background: var(--fill-brand-weak); border-radius: 16px; padding: 14px 16px; gap: 6px;`
  - Head: "**Included Tools by Plan**" (600, 14 / 20) + info icon (16px, flipped `scaleY(-1)`).
  - Rows (14px / 22px, space-between, numbers 700 18px):
    - "Subscribers — **37+** tools"
    - "Free & Pay-as-You-Go — **18** AI tools"
  - **Info tooltip:**
    - Opens on hover and focus on desktop; **tap toggles** it on touch.
    - Position: below the icon, `top: calc(100% + 10px)`. Size: width 324px, `max-width: calc(100vw − 32px)`.
    - Style: same as the connector tooltip; the arrow points **up**.
    - JS centers it under the icon, keeps it 16px inside the viewport, and sets the arrow x to the icon's center.
    - Entrance: `tip-in .15s ease-out` (from `opacity 0, translateY(-4px)`).
    - Hidden state is `display: none`.
    - Content: title "**Subscription is required for these tools:**"; body "Photo Enhancer, Object Removal, AI Replace, etc. See the complete list in the pricing table below."
- **Pack list** (radio group), 8 packs: **100, 200, 500, 1000, 2000, 5000, 10000, 20000** credits.
  - The first 5 are visible; the rest appear with **Show more**. Default selection: **500**.
  - Row: `min-height: 48px; padding: 8px 12px; gap: 12px; border-bottom: 1px solid var(--stroke-weak);` hover `--fill-hover` (`.15s`).
    - Left: a custom radio, 22px circle, `border: 1.5px solid rgba(17,24,26,0.3)`. Checked: brand fill with a white inner dot (inset 6px). Focus: `0 0 0 3px rgba(3,173,226,.35)`.
    - Then "{N} credits" (15px / 22px).
    - Right: per-credit price — **bold 600 18px** amount + "/credit" (13px).
  - Selecting a pack updates the card's big price to that pack's total.
- **Show more / Show less** button: 600, 15px / 22px, `--text-strong`, centered, `padding: 6px 10px; margin-top: -4px`. Its chevron rotates 180° (`.2s`) when expanded; `aria-expanded`.
- Footer: "These credits can be used at any time within one year" (12px / 16px, centered, pinned to the card bottom).

### Pro Exclusive Pack
- Plan name "Pro Exclusive Pack"; price = discounted total; the Standard total is shown crossed out next to it.
- Primary button **"Subscribe now"**.
- Same pack list, with these differences:
  - Rows are 60px tall.
  - The selected row has a `rgba(255,255,255,0.7)` background.
  - Each row shows the Standard per-credit price crossed out (12px / 16px, `--text-weaker`, line-through) above the Pro per-credit price.
  - Pro prices are teal `#00998a` (both the big price and the per-credit prices).
  - **Per-pack promo tag (optional):** a pack row can carry a small tag right after "{N} credits" (gap 8px), e.g. **"30% off"** on the 2000-credit Pro pack. Tags are configured per plan and pack. Style (Figma 2387-19578): `padding: 2px 6px; border: 1px solid var(--stroke-inverse-strong); border-radius: 32px; background: var(--icon-success)` (`#067a57cc`); text white (`--text-inverse-strong`), Roboto 600, 10px / 16px; 22px tall in total.
- Same footer text.
- **Corner badge** "{x}% OFF", where x = `round((1 − pro / standard) × 100)`, i.e. 40% OFF.
  - Same position and type as the Subscription badge.
  - Background: the **orange gradient** `linear-gradient(88deg, #ffb802 0%, #ffa51c 33%, #ff8800 100%)`; no border.

### PAYG pack totals

| Credits | 100 | 200 | 500 | 1000 | 2000 | 5000 | 10000 | 20000 |
|---|---|---|---|---|---|---|---|---|
| USD Standard | 17.99 | 31.99 | 72.99 | 129.99 | 214.99 | 449.99 | 699.99 | 999.99 |
| USD Pro | 10.79 | 19.19 | 43.79 | 77.99 | 128.99 | 269.99 | 419.99 | 599.99 |
| TWD Standard | 569 | 1040 | 2300 | 3970 | 6780 | 14750* | 26000* | 46000* |
| TWD Pro | 341 | 624 | 1380 | 2380 | 4060 | 8850* | 15600* | 27600* |

Per-credit = total ÷ credits. \* = placeholder; confirm with PM. Other currencies: use the real price tables.

## 9. Responsive behavior (cards area)

- **≤1280px (tablet / small laptop):**
  - Page and hero:
    - `.page` padding `0 32px 56px`
    - hero padding `40px 0 0`, gap 24
    - h1 34px / 40px
  - Toggles and subtitle:
    - tabs width 480px, padding 6; tab 20px / 28px, padding 12
    - billing buttons min-width 104, 16px / 24px, padding `6px 12px`; Save badge 13px / 18px
    - subtitle 16px / 24px
  - **Cards become a horizontal scroller** when three no longer fit:
    - `gap: 16px; justify-content: flex-start; overflow-x: auto; scroll-snap-type: x mandatory; scroll-padding: 0 32px;`
    - The scroller bleeds to the screen edges: `width: calc(100% + 64px); margin: -20px -32px -40px; padding: 20px 32px 40px;` so shadows and badges aren't clipped. When the cards fit, there is no padding gap on the sides during scroll.
    - Columns `flex: 1 0 344px; min-width: 344px;` with `scroll-snap-align: start`.
  - Card and type sizes:
    - card padding 24, radius 28, gap 12; card-body gap 14
    - plan-title padding `6px 0`; plan name 18px / 26px
    - price 46px / 54px; /mo 18px; crossed-out price 20px / 28px
    - tier tag 17px; buttons 44px tall, 18px / 26px
    - features gap 10, padding 4; items 15px / 22px; icons 18px
    - corner badge 16px / 24px
- **≤768px (stacked):**
  - No scroller. Cards stack in **`column-reverse`**, so the **most expensive plan is on top** (Pro → Plus → Free; Pro Exclusive → Standard → Free).
  - `gap: 32px`; each column is full width, `max-width: 460px`, centered. The toolbar is also `max-width: 460px`.
- **≤600px (phone):**
  - Page and hero: `.page` padding `0 16px 48px`; hero padding-top 32, gap 24; h1 28px / 34px.
  - Tabs and subtitle:
    - tabs padding 6; tab 18px / 24px, padding `12px 8px`
    - subtitle becomes `display: block`, 14px / 20px; its icon is 15px and inline (`vertical-align: -6px`)
  - Toolbar: the invisible API-link spacer is removed (`display: none`); the API link is 16px.
  - Cards: card padding `24px 20px`, radius 24; price 48px / 56px; /mo 18px; crossed-out price 20px; the corner badge sits `right: 20px`.
- **≤360px:** slider labels 11px.
- Verify there is **no horizontal page scroll** at 375px and 320px, with tooltips open and closed.

## 10. Acceptance checklist
- [ ] Switching tabs crossfades the title / sub-row with no height jump. The pill slides. The selected currency is unchanged.
- [ ] Yearly is the default and shows "Save 60%". Monthly / Yearly updates the Plus and Pro prices.
- [ ] The Pro slider snaps through the 6 stops by drag, click, label click and keyboard. The tooltip, the "{N} Credits" line and the price all update; the price digits roll down in 400ms.
- [ ] On first view, the finger swipes 300 → 10,000 three times with the orange trail covering the dots, then disappears. Touching the slider stops it at once.
- [ ] With `holidayDiscount = 0.3`: $7.99 shows as $5.59, $7.99 is crossed out, and the red "30% off" badge appears. The price stays black.
- [ ] "Usage details" scrolls to the expanded AI Agent group under the sticky headers. The PAYG info icon scrolls to Compare All Plans.
- [ ] All hovers are gray. The focus ring is brand blue. Reduced motion disables every animation.
