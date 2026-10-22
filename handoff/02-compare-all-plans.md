# Prompt 2 — Compare All Plans (table section)

> 給 RD：把整份檔案（從下一行起）貼給 AI coding 工具，並附上 `assets/` 內列出的 SVG。
> 參考實作：repo 根目錄的 `index.html`，搜尋 `Compare All Plans` 區塊，所有數值以它為準。

---

You are updating the **"Compare All Plans"** section of our Next.js pricing page (`/pricing`), which sits right below the plan cards. Rebuild it to match the spec below exactly. Adapt the markup to our components / CSS modules, but keep every value.

**This is a layout / interaction change only.** The table content (groups, tools, credit costs and per-plan limits) is managed in our **back office by a dedicated team** and loaded from the existing data source. Keep reading rows from that source, never hard-code them, and render whatever it returns with the layout below. §7 shows sample content only.

A complete working reference is in `index.html` (the `.compare` section; its `COMPARE` array is hard-coded sample data standing in for the back-office data). When this spec and the reference disagree, the reference wins.

## 0. Global rules
- Fonts: Roboto (body), Wix Madefor Display (section title).
- Tokens (resolved values):

  | Token | Value |
  |---|---|
  | `--fill-brand-strong` | `#03ade2` |
  | `--stroke-brand-strong` | `#03ade2cc` |
  | `--fill-strong` | `#09161af2` |
  | `--fill-weak` | `#11181a0a` |

- The site header is **sticky, 64px tall**. Everything sticky in this section sits under it.
- Reduced motion: no collapse animation and no smooth scrolling.
- Never cause horizontal page scroll.

## 1. Section
- `<section id="compare">`: `background: #f5f5f5; padding: 80px 32px;`
- Inner wrapper: `max-width: 1200px; margin: 0 auto;`
- Title (h2) **"Compare All Plans"**: Wix Madefor Display 800, 40px / 40px, letter-spacing −0.5px, `#212529`, centered, `margin-bottom: 40px`.
- Other page elements link here:
  - The PAYG subtitle info icon scrolls to `#compare`.
  - The cards' "Usage details" links scroll to the AI Agent group, which must have `id="cmp-ai-agent"`.

## 2. Sticky plan header (with gradient mask)
The plan header row **sticks under the site header** while the table scrolls.

- Sticky wrapper:
  - `position: sticky; top: 64px; z-index: 10; isolation: isolate;`
  - Define `--gap: 19px` and `--side` (= the section's side padding: 32px, or 20px on mobile).
  - `margin: calc(-1 * var(--gap)) calc(-1 * var(--side)) 0; padding: var(--gap) var(--side) 0;` This leaves a 19px gap between the site header and the plan header and spans the full section width.
- **Gradient mask** (Figma "Gradient Mask"): a pseudo-element behind the header.
  - `inset: 0 0 calc(-1 * var(--gap)) 0; z-index: -1; pointer-events: none;`
  - `background: linear-gradient(#f5f5f5, rgba(245,245,245,0));`
  - It starts at the top of the gap and ends 19px below the header. Rows scrolling up are almost hidden in the gap and fade in just under the header. **The solid end is at the top; the faded end is at the bottom.**
- Header row (white bar):
  - Layout: `display: grid; grid-template-columns: repeat(5, minmax(0,1fr)); gap: 8px; padding: 10px; align-items: stretch;` (all plan boxes share one height even if a subtitle wraps).
  - Look: `border-radius: 36px; background: #fff; box-shadow: 0 14px 10px rgba(0,0,0,0.15);`
- Columns, in order:
  1. Label **"Tools"**: 400, 20px / 28px, `#000`, `padding-left: 28px`, vertically centered.
  2. Label **"Credit Cost"**: 400, 18px / 26px, centered.
  3–5. **Plan boxes** (below).
- Plan box: flex column, `gap: 8px; padding: 12px 16px; border-radius: 24px; background: var(--fill-weak);`
  - Name row: flex, space-between, `gap: 6px;` 600, 18px / 26px, `#000`. The name is on the left, a mini tag on the right.
  - Subtitle: 16px / 24px, `#212529`, `margin-top: -6px`.
  - Button: pinned to the bottom (`margin-top: auto`).
    - Box: height 40px, `border-radius: 32px;`
    - Text: white, 600, 18px / 26px, no wrap.
    - Hover: `filter: brightness(1.08)` with `.15s`.

  | Plan | Name / tag | Subtitle | Button |
  |---|---|---|---|
  | Free | "Free" (no tag) | "5 free credits upon sign-up" | **"Sign Up"**, background `var(--fill-strong)` (near black) |
  | Plus | "Plus" + PLUS tag | "100 credits per month" | **"Subscribe"**, brand blue `#03ade2` |
  | Pro | "Pro" + PRO tag | "300 credits per month" | **"Subscribe"**, `linear-gradient(88.8deg, #ffb802 0%, #ffa51c 32.565%, #ff8800 99.797%)` |

- **Mini tags:**
  - Base: `padding: 5px 8px 4px; border-radius: 100px;` white, 12px, line-height 1, 700.
  - PLUS: `background-image: linear-gradient(34deg, rgba(68,227,237,0.8) 2.43%, #03ade2cc 67.21%), linear-gradient(#fff, #fff);`
  - PRO: `padding: 5px 9px 4px; border-radius: 22px;` 900, `background-image: linear-gradient(89.5deg, #ffb802 0%, #ffa51c 32.565%, #ff8800 99.797%);`

## 3. Groups (collapsible white panels)
- Groups stack in a column, `gap: 24px`, starting 24px below the header.
- Group panel: `padding: 14px 0; border-radius: 40px; background: #fff; overflow: clip;`
- **Group title button:**
  - Box: full width, height 60px, `padding: 0 40px; gap: 10px;` no border or background, left-aligned.
  - Text: Roboto 600, 20px / 28px, `#000`.
  - Chevron: `chevron-section.svg`, 28px, after the text.
    - Expanded: points **up** (`transform: scaleY(-1)`).
    - Collapsed: points down.
    - `transition: transform .25s ease`.
  - ARIA: `aria-expanded`, `aria-controls` → body id.
  - **All groups start expanded.**
- **Collapse animation:** the body is a grid, `grid-template-rows: 1fr` → `0fr` when collapsed, `transition: grid-template-rows .3s ease`. The inner wrapper has `overflow: hidden; min-height: 0`.
- The first group ("Watermark free" only) has **no title**.

## 4. Rows and cells
- Row: same 5-column grid as the header, `align-items: center; min-height: 60px;`
- Every cell: `padding: 14px 20px;` 16px / 24px, `#15171f`, centered.
- **Tool name** (column 1): 600, left-aligned, `padding-left: 40px`.
- **Cell content:**
  - `✓` → `table-tick.svg` (20px, alt "Included")
  - `–` → `table-minus.svg` (20px, alt "Not included")
  - Anything else → plain text
  - Cells are flex-centered with `gap: 6px`.
- Credit Cost cells keep a number with its unit (e.g. "1 Image") on one line: use a non-breaking space between a digit and a following word.
- **Singular / plural units (required).** Never show "(s)" (e.g. "2 use(s) / day", "1 credit(s)"). The front end decides each unit from the number in front of it:
  - The number is exactly 1 and is **not** a range → singular: "1 use / day", "1 credit", "1 sec", "1 Image".
  - Anything else → plural: "2 uses / day", "0 credits", "1.5 credits", ranges like "1 - 2 credits" and "2 - 3 secs", "20 images = 30 credits".
  - A unit with no number in front → plural ("credit(s)" → "credits").
  - Each number–unit pair in a string is decided on its own: "From 1 credit / 1 Image", "From 2 credits / 1 sec".
  - If the back office gives structured data (count + unit), pluralize from the count. If it only gives text with "(s)", convert the text:

  ```js
  // "From 2 credit(s) / 1 sec(s)" → "From 2 credits / 1 sec"
  const PLURAL = { use: 'uses', credit: 'credits', sec: 'secs', image: 'images', video: 'videos' };
  const plural = w => {
    const p = PLURAL[w.toLowerCase()] ?? `${w}s`;
    return w[0] === w[0].toUpperCase() ? p[0].toUpperCase() + p.slice(1) : p; // keep "Image" capitalized
  };
  function formatUnits(text) {
    return text
      // "<number>[ - <number>] <unit>(s)"
      .replace(/(\d[\d,.]*)(\s*-\s*\d[\d,.]*)?(\s+)([A-Za-z]+)\(s\)/g, (_, n, range, sp, unit) =>
        `${n}${range ?? ''}${sp}${!range && Number(n.replace(/,/g, '')) === 1 ? unit : plural(unit)}`)
      // a "<unit>(s)" with no number in front → plural
      .replace(/\b([A-Za-z]+)\(s\)/g, (_, unit) => plural(unit));
  }
  ```

  | Back office | Shown |
  |---|---|
  | `1 use(s) / day` | 1 use / day |
  | `2 use(s) / day` | 2 uses / day |
  | `2 use(s) / lifetime` | 2 uses / lifetime |
  | `1 credit(s) / 1 sec(s)` | 1 credit / 1 sec |
  | `1 credit(s) / 2 - 3 sec(s)` | 1 credit / 2 - 3 secs |
  | `From 1 credit(s) / 1 Image(s)` | From 1 credit / 1 Image |
  | `From 12 credit(s) / 1 video(s)` | From 12 credits / 1 video |
  | `1 - 2 credit(s)` | 1 - 2 credits |
  | `20 image(s) = 30 credit(s)` | 20 images = 30 credits |

- Otherwise show the back office's text as-is.
- Map cell values to icons: the back office's "included" value → tick icon, "not included" → minus icon, anything else → plain text.

## 5. Bottom note
Below the groups, `margin-top: 24px; padding: 0 8px;` 16px / 24px, `#15171f`:
"Users on the Plus/Pro plan can use subscription features without credit consumption and download without watermark."

## 6. Mobile (≤900px; Figma mobile table)
- Section: `padding: 40px 20px 48px`; sticky `--side: 20px`. Title 28px / 40px, `margin-bottom: 20px`.
- Header:
  - **3 columns** (the Tools / Credit Cost labels are hidden), `border-radius: 20px`.
  - Plan box: `padding: 8px; border-radius: 12px; justify-content: space-between;`
  - Text: name 16px / 24px (gap 3px); subtitle 14px / 20px.
  - Mini tag: `padding: 4px 6px;` 8.5px, 800 (PRO too).
  - Button: 32px tall, 14px / 20px.
- Groups: gap 20px, starting 20px below the header. Panel `padding: 4px 0 16px; border-radius: 20px;` Title `padding: 0 16px;` 18px / 26px, chevron 24px.
- **Each tool takes two lines:**
  - **Line 1** spans all 3 columns as a flex row (`justify-content: space-between; gap: 12px; padding: 0 12px`):
    - The tool name on the left: `flex: 0 0 auto; max-width: 60%; padding: 8px 0 6px`.
    - The credit cost on the right: `flex: 1 1 0; min-width: 0; text-align: right; padding: 8px 0 6px`. The cost takes whatever width the name leaves; it is not a fixed third.
  - **Line 2** holds the 3 plan cells (Free / Plus / Pro) in a 3-column grid. Row padding `0 4px`; cell padding `8px 4px 16px`; 14px / 20px.

## 7. Sample content (for layout testing only)
**Do not copy these rows into code.** Real rows come from the back office. They are listed so you can check the layout with realistic lengths: long tool names, long costs, and "- -" placeholders.

Columns: **Tool · Credit Cost · Free · Plus · Pro**. `✓` = included, `–` = not included. "- -" is literal text.

Rendering rules that must hold for any back-office data:
- A group without a title (like the first one) renders with no title button and cannot collapse.
- The group whose title is **"AI Agent"** gets `id="cmp-ai-agent"` (the cards' "Usage details" target). Prefer a stable key from the back office over matching the title text.
- Groups and rows appear in the order the back office returns them.

**(no title)**

| Tool | Credit Cost | Free | Plus | Pro |
|---|---|---|---|---|
| Watermark free | *(empty)* | – | ✓ | ✓ |

**AI Photo Editing**

| Tool | Credit Cost | Free | Plus | Pro |
|---|---|---|---|---|
| Photo Enhancer | - - | 2 uses / day | 5 uses / day | ✓ |
| Photo Repair | - - | 2 uses / day | 5 uses / day | ✓ |
| AI Colorize | - - | 3 uses / day | ✓ | ✓ |
| AI Color Correction | - - | 3 uses / day | ✓ | ✓ |
| AI Lighting | - - | 2 uses / day | 200 uses / day | 250 uses / day |
| AI Image Extender | - - | 1 use / day | 200 uses / day | 200 uses / day |
| AI Replace | - - | 2 uses / day | ✓ | ✓ |
| Object Removal (Standard) | - - | 3 uses / day | ✓ | ✓ |
| Object Removal (Advanced) | - - | 1 use / day | 1 use / day | 150 uses / day |
| Remove Background, Blur Background, Change Background | - - | 3 uses / day | 5 uses / day | ✓ |
| Crop | - - | ✓ | ✓ | ✓ |
| Flip & Rotate | - - | ✓ | ✓ | ✓ |
| Resize | - - | ✓ | ✓ | ✓ |
| Adjust Light & Color | - - | ✓ | ✓ | ✓ |
| Effects | - - | Limited | ✓ | ✓ |
| AI Image Generator | From 1 credit / 1 Image | ✓ | ✓ | ✓ |
| Image Templates | From 1 credit / 1 Image | ✓ | ✓ | ✓ |

**AI Portrait**

| Tool | Credit Cost | Free | Plus | Pro |
|---|---|---|---|---|
| AI Makeup Virtual Try On | - - | – | ✓ | ✓ |
| AI Face Reshape | - - | – | ✓ | ✓ |
| AI Face Retouch | - - | – | – | ✓ |
| AI Body Reshape | - - | – | Limited | ✓ |
| AI Hair Color | - - | 1 use / day | ✓ | ✓ |
| AI Hairstyle Generator | 2 credits | ✓ | ✓ | ✓ |
| AI Face Swap | 1 - 2 credits | ✓ | ✓ | ✓ |
| AI Clothes | 2 credits | ✓ | ✓ | ✓ |
| AI Beard Generator | 2 credits | ✓ | ✓ | ✓ |
| AI Makeup Transfer | 2 credits | ✓ | ✓ | ✓ |
| AI Headshot Generator | 20 images = 30 credits | ✓ | ✓ | ✓ |
| AI Studio Generator | 10 images = 10 credits | ✓ | ✓ | ✓ |
| AI Avatar Generator | 10 images = 10 credits | ✓ | ✓ | ✓ |

**AI Video Editing**

| Tool | Credit Cost | Free | Plus | Pro |
|---|---|---|---|---|
| Video Enhancer | 1 credit / 1 sec | ✓ | ✓ | ✓ |
| Video Face Swap | 1 credit / 2 - 3 secs | ✓ | ✓ | ✓ |
| Video Style Transfer | 4 credits / 1 sec | ✓ | ✓ | ✓ |
| Image to Video | From 2 credits / 1 sec | ✓ | ✓ | ✓ |
| Text to Video | - - | ✓ | ✓ | ✓ |
| Video Object Removal | From 2 credits / 1 sec | ✓ | ✓ | ✓ |
| AI Video Editor | From 4 credits / 1 sec | ✓ | ✓ | ✓ |
| Character Motion Swap | From 4 credits / 1 sec | ✓ | ✓ | ✓ |
| Video Templates | From 1 credit / 1 sec | ✓ | ✓ | ✓ |

**AI Agent** — `id="cmp-ai-agent"` (the cards' "Usage details" target)

| Tool | Credit Cost | Free | Plus | Pro |
|---|---|---|---|---|
| General Chat | 5 credits | 2 uses / lifetime | 2 uses / lifetime | 5 uses / day |
| Multi Image Upload | 8 - 10 credits | ✓ | ✓ | ✓ |
| Advanced Editing and Generation | 12 credits | ✓ | ✓ | ✓ |
| Video Generation | - - | ✓ | ✓ | ✓ |

**Others**

| Tool | Credit Cost | Free | Plus | Pro |
|---|---|---|---|---|
| Batch Photo Enhancer | - - | – | 1 use / day | ✓ |
| Batch Image Converter | - - | – | ✓ | ✓ |
| Commercial use allowed | - - | – | – | ✓ |

> The back office decides the credit costs. If a value is missing (for example it would render as "null"), show "- -" instead of the raw text.

## 8. "Usage details" deep link (from the plan cards)
When a card's "Usage details ↓" is clicked:
1. Prevent the default jump.
2. If the AI Agent group is collapsed, expand it.
3. Smooth-scroll so the group's top lands at `64px (sticky top) + sticky plan-header height + 16px` from the viewport top. Use an instant jump under reduced motion.

## 9. Acceptance checklist
- [ ] Scrolling the table keeps the white plan header 19px under the site header. Rows fade out behind the gradient in that gap, with no hard edge.
- [ ] The plan boxes are equal height; the buttons stay at the bottom; the hover brightens them.
- [ ] Every group collapses and expands with a smooth height animation, and the chevron flips.
- [ ] At ≤900px: 3 plan columns; each tool shows name + cost on line 1 and the three plan cells on line 2; no horizontal scroll at 320px.
- [ ] No "(s)" anywhere in the table: every unit is singular after exactly 1 and plural otherwise, including ranges.
- [ ] "Usage details" from any card lands on the expanded AI Agent group, fully visible under the sticky bars.
