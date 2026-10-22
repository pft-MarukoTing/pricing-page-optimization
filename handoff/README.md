# Pricing page 改版 — RD 交付包

這個資料夾把 prototype（repo 根目錄 `index.html`）的所有畫面與互動細節整理成兩份可以直接貼給 AI coding 工具的 prompt，讓 RD 能套用到官網 `https://yce.perfectcorp.com/pricing`。

## 內容

| 檔案 | 範圍 |
|---|---|
| [01-pricing-cards.md](01-pricing-cards.md) | 上半部：標題、Subscription / Pay As You Go 分頁、Monthly / Yearly、幣別下拉、See API Pricing、所有方案卡片（含 Pro 橘色 credit slider、手指滑動提示、價格滾動動畫、Holiday discount、AI Agent 區塊、PAYG 點數包）與 RWD |
| [02-compare-all-plans.md](02-compare-all-plans.md) | 下半部：Compare All Plans 表格的版型與互動（sticky 方案列 + 漸層遮罩、可收合群組、手機兩行版型、Usage details 錨點）。表格內容由後台團隊設定，沿用現有資料來源，不寫死 |

## 怎麼用

1. 在官網 repo 開一個分支。
2. 把需要的 prompt 檔**從 `---` 之後**整段貼給 Claude Code / Cursor，並把下方列出的 SVG 一併放進專案。
3. 兩份可以分開做、分開上線；唯一的連動是卡片上的 **Usage details** 會捲到表格的 `#cmp-ai-agent`，以及 PAYG 副標的 info icon 會捲到 `#compare`。
4. 有疑問時以 `index.html` 為準：用瀏覽器直接打開就能操作，所有數值都在裡面的 `<style>` 與 `<script>`。
5. 用每份 prompt 最後的 **Acceptance checklist** 驗收。

## 需要的素材（`assets/`）

| 檔案 | 用途 |
|---|---|
| `design-tokens.css` | 設計 token（官網已有同一套就不用重複引入） |
| `circletick.svg` / `circletick-orange.svg` | 卡片功能列表勾勾（藍 / Pro 卡橘） |
| `handle-orange.svg` | Pro slider 把手 |
| `coin.svg` | slider tooltip 的 credits icon |
| `finger-pointer.svg` | 滑動提示手指（iconpacks.net #10507） |
| `cross-line.svg` | 原價刪除線 |
| `arrow-right.svg` | See API Pricing 箭頭 |
| `arrow-down.svg` / `external-link.svg` | AI Agent 區塊連結 icon |
| `ic-info.svg` | AI Agent Connector info icon |
| `icon-info.svg` | PAYG 副標與 Included Tools 的 info icon |
| `chevron.svg` | 幣別下拉箭頭 |
| `chevron-section.svg` | 表格群組收合箭頭 |
| `table-tick.svg` / `table-minus.svg` | 表格 ✓ / – |

## 待 PM 確認

- 各幣別的正式價目表（prototype 的匯率是暫時的換算）。
- TWD 5,000 / 10,000 / 20,000 點數包價格（目前是 placeholder）。
- Holiday discount 開關：目前預設關閉（`holidayDiscount = 0`），開啟為 30%。prototype 右下角的 Discount On/Off 只是給審稿用，**不要上線**。
