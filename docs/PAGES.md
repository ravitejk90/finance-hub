# Page-by-page reference

A granular, screen-by-screen tour of the Finance Hub. Every page follows the
same template:

- **Purpose** — what question the page answers.
- **Layout** — the components top to bottom, as they appear on screen.
- **Elements** — a table of each element: what it shows and which snapshot
  field feeds it (see `docs/DATA_MODEL.md` for the schema).
- **Rules** — behavior and edge cases that are easy to miss.

All figures below are **synthetic examples**. Labels in `code` are the literal
UI strings.

> How to read the source column: `snapshot.investments_total` means the
> top-level `investments_total` key of the hub snapshot. Anything marked
> *computed* is derived in the UI, not stored.

---

## Global chrome (present on every screen)

**Purpose.** Orientation and navigation; the passcode gate keeps the whole hub
private.

| # | Element | Shows | Source |
|---|---|---|---|
| 1 | Header avatar + status | Owner avatar, `Updated {Mon D} at {h:MM AM/PM}` freshness stamp | `snapshot.as_of` |
| 2 | `Refresh` button | Triggers the daily-refresh pipeline on demand | action |
| 3 | Gear icon | Opens the settings popover (account visibility toggles live here, next to the gear — no scrolling) | `snapshot.hidden_accounts`, user-managed |
| 4 | Tab bar | `Overview · Assets · Debts · Spending · Goals · Plan`; active tab is highlighted | navigation state |
| 5 | Passcode gate | `Enter Passcode` / `Your financial picture is protected`; numeric keypad; attempt counter | app-level lock |

**Rules.**

- The freshness stamp is the snapshot's `as_of`, not "now" — it tells you how
  stale the data is at a glance.
- Overlays (dialogs, settings, confirmations) open in place at the current
  scroll position and restore scroll + focus on close.

---

## 1. Overview

**Purpose.** "Where do I stand?" — net worth and its trend in one glance.

**Layout.**

1. `NET WORTH` hero card
2. (Below the fold on small screens: asset/debt summary tiles — same fields as
   the hero breakdown line)

**Elements.**

| Element | Shows | Source |
|---|---|---|
| Eyebrow label | `NET WORTH` | static |
| Hero value | Total net worth, large | *computed:* `assets_total − debts_total` |
| Status pill | `Positive net position` (or negative variant) | *computed* from sign of net worth |
| Breakdown line | `{financial assets} financial assets · {home equity} home equity · {cards} cards` | *computed* from investments+cash, `home.equity`, Σ cards |
| Progress row | `Net worth progress` … `+{P/L}` … `Period P/L · {range}` | net-worth history log |
| Range selector | `1M · 3M · 6M · 1Y · All` | controls the chart window |
| Trend chart | Net-worth history for the selected range | net-worth log (appended by each refresh) |

**Rules.**

- The chart's data comes from an append-only history log, not from the
  snapshot — a refresh never rewrites history, only appends.
- Example values: net worth `$512,340.55`; breakdown `$402,150.75 investments ·
  $45,230.10 cash · $65,371.90 home equity`.

---

## 2. Assets

**Purpose.** Everything owned, broken into four lenses plus research.

**Layout.** A sub-tab bar under the main tab bar:

`[Investments] [Cash] [Income] [Home]` and a `Research` entry point.

### 2a. Investments

**Purpose.** Portfolio value, its trend, and every account/holding.

| Element | Shows | Source |
|---|---|---|
| Eyebrow + hero | `PORTFOLIO` … `{investments_total}` large | `snapshot.investments_total` |
| Helper text | e.g. `Connect an account to inspect its positions` (empty state) | static |
| Progress row | `Portfolio progress` … `+{P/L}` … `Period P/L · {range}` | history log |
| Range selector | `1M · 3M · 6M · 1Y · All` | chart window |
| Trend chart | Portfolio history | history log |
| Group cards | Owner groupings, e.g. `Mine` … `{value}`, `Family` … `{value}` | *computed* by grouping accounts |
| Account cards | Institution, account label + masked suffix, value | `snapshot.investments[]` |
| Holding rows | Ticker, name, value, shares, price | `snapshot.investments[].holdings[]` |

**Rules.**

- Every investment account returned by the bank sync appears — the list is
  never hardcoded. A `$0` balance is valid data and still renders.
- A cash/margin line inside an account is kept as a holding so account totals
  reconcile.

### 2b. Cash

**Purpose.** Liquid cash across banks, with per-account detail.

| Element | Shows | Source |
|---|---|---|
| Hero row | `Combined cash` … `{cash_total}` … `+{P/L} · {range}` | `snapshot.cash_total`, history |
| Range selector + chart | `1M · 3M · 6M · 1Y · All`, cash history | history log |
| `ACCOUNTS` section | One row per account: bank icon, `{name} …{last4}`, `{institution} · {note}`, `{value}` right-aligned | `snapshot.cash[]` |

**Rules.**

- The `note` field carries useful context (e.g. `2.8% APY`) and is
  user-editable; the refresh never overwrites it.
- Example row: `Accelerate Savings …0000` · `Example Bank · 2.8% APY` ·
  `$38,500.00`.

### 2c. Income

**Purpose.** What came in this period, split by business vs. personal.

| Element | Shows | Source |
|---|---|---|
| Eyebrow + hero | `INCOME · {MONTH YEAR}` … `{total}` large | *computed* from income records |
| Subtitle | `Income-category deposits + final invoices` | static (defines the income rule) |
| Period selector | `Month · 3 months · Year · All` | aggregation window |
| `Through month` + month pill | Which month the view covers, e.g. `October 2026` | selected period |
| Split cards | `Total` … `{total}` · `Business` … `{business}` (`Final invoices`) · `Personal` … `{personal}` (`Income-category deposits`) | *computed* |
| `LINE ITEMS` section | `{N} income records` — the underlying deposits/invoices | income records |

**Rules.**

- Income definition: payroll/interest and similar deposits only. Loans,
  refunds/travel credits, own-account transfers, and business→personal
  transfers are excluded (no double-counting).
- Business vs. personal split follows the invoice/deposit source.

### 2d. Home (real estate)

**Purpose.** Home equity and its trend.

| Element | Shows | Source |
|---|---|---|
| Eyebrow | `REAL ESTATE` | static |
| Location | City, ST (e.g. `Example City, ST`) | `snapshot.home.location` |
| Label + hero | `Estimated equity` … `{home.equity}` large | *computed:* `home.estimated_value − mortgage.balance` |
| Progress row | `Equity progress` … `+{P/L}` … `Period P/L · {range}` | history log |
| Range selector + chart | `1M · 3M · 6M · 1Y · All`; equity history with hover tooltip (`{value}` / `{date}`) | history log |
| `Estimated value` card | Home value estimate | `snapshot.home.estimated_value` (manual) |
| `Mortgage` card | Balance, rate, monthly payment | `snapshot.mortgage` |

**Rules.**

- The home value estimate is **manual** (e.g. from a valuation site, refreshed
  occasionally) — the bank sync never touches it.
- Equity is always estimate minus mortgage balance; both are shown so the
  math is checkable.

### 2e. Research

**Purpose.** "What do I own and what should I know about it?" — one card per
held stock, following the portfolio automatically.

| Element | Shows | Source |
|---|---|---|
| Intro card | `PORTFOLIO RESEARCH · {date}` … `Practical keep view` … `Risk buckets, not trade instructions. ({N} companies covering the owner's current stock holdings.)` | `snapshot.research.as_of`, company count |
| Bucket cards | Title (`Stronger core` / `Watch closely` / `Highest risk`), ticker chips, one-line description | `snapshot.research.buckets[]` |
| Company card header | Ticker, company name, bucket tag, price + day change | research data |
| Summary | 2–4 sentence business + financial-state summary | `company.summary` |
| Metrics grid | 8 metrics, each: label, value, context, source link — trailing P/E, forward P/E, PEG, revenue, revenue growth, net margin, debt/equity, dividend yield | `company.metrics[]` |
| `Key risks` | 2–3 current, specific risks | `company.risks[]` |
| Sources | Named links underpinning the figures | `company.sources[]` |
| Next earnings | `{Mon D, YYYY} · {before/after close}` — hidden entirely when unknown | `company.next_earnings` |

**Rules.**

- The universe is **dynamic**: distinct individual-stock tickers with combined
  position value ≥ the threshold (e.g. $500). New qualifiers get a full card;
  sold positions drop off. No hardcoded ticker list.
- Excluded: ETFs, mutual funds, crypto, cash/money-market, options, dust.
- Every figure carries a source link; unknown earnings dates stay unknown
  (never invented); non-meaningful metrics show `N/M`, never `0`.
- Buckets describe risk, not instructions — the UI copy says so explicitly.

---

## 3. Debts

**Purpose.** Everything owed — cards and mortgage — with the same hero +
trend treatment as assets.

| Element | Shows | Source |
|---|---|---|
| Hero card | Total debt (`mortgage.balance + Σ cards.value`), progress row, range selector, trend chart | *computed*, history log |
| Card rows | Institution, card label + masked suffix, current balance | `snapshot.cards[]` |
| Mortgage card | Balance, rate, monthly payment, due date, autopay status, extra principal | `snapshot.mortgage` |

**Rules.**

- Every credit account returned by the sync appears, including `$0` balances.
- `debts_total` is always recomputed as `mortgage.balance + Σ(cards.value)` —
  never stored independently.

---

## 4. Spending

**Purpose.** "Where did it go?" — cash flow, trends, category breakdown,
transactions, budgets.

**Layout** (top to bottom):

1. Cash-flow hero card
2. Period-comparison chart card
3. `Where it went` breakdown card
4. Transactions list
5. Budgets

**Elements.**

| Element | Shows | Source |
|---|---|---|
| Cash-flow hero | `CASH FLOW · {MONTH YEAR}` … `{total spend}` large … `Total spend` | *computed* from transactions in period |
| Period selector | `Week · Month · Quarter · Year` | aggregation window |
| `PERIOD COMPARISON` card | `Month over month` grouped bars: `Income` vs `Total spend` per month (`May`–`Oct`), footer `{Month Year}` | income records + transactions |
| `BREAKDOWN` / `Where it went` | Donut chart by category | *computed* |
| `Categories · Tags` toggle | Switches the breakdown dimension | view state |
| Donut center | `{total}` … `Total spend in {Month Year}` | *computed* |
| Category rows | Color dot, name, `{amount}`, `{share %}`, chevron | *computed* per category |
| Transactions list | Merchant, date, category, amount | synced transactions |
| Category control | Dropdown of existing categories (no free text) | category list |
| Budget rows | Category, spent vs. budget, progress bar | budgets + transactions |

**Rules.**

- The hero total always equals the sum of the listed transactions for the
  selected period — if they disagree, that's a bug, not rounding.
- Budgets recompute from current-period transactions after every refresh and
  every recategorization.
- The category dropdown prevents typos from corrupting the data; categories
  are a fixed list.
- Transfers between the owner's own accounts never count as income or
  spending.

---

## 5. Bills

**Purpose.** The actual household bills the owner pays — nothing else.

| Element | Shows | Source |
|---|---|---|
| Bill rows | Name, amount, billing period, due date, status (`Autopay` / `Due` / `Paid`) | `snapshot.bills[]` |
| Bills total | Plain number (not a tile) | *computed* |
| Period controls | `Month · 3 months · Year · All` + month picker; defaults to current month | view state |

**Rules.**

- Bills = utilities, car payments, and genuine household bills the owner pays.
- Subscriptions live only under Subscriptions — never mixed in.
- Escrow-paid items (e.g. insurance paid through the mortgage) are
  informational, not bills.
- The bill archive sync **replaces** the whole table each run — always send
  the complete record set, never deltas.

---

## 6. Subscriptions

**Purpose.** Recurring charges, auto-detected and tracked.

| Element | Shows | Source |
|---|---|---|
| Subscription rows | Merchant name, cost (`$X.XX/mo`), `Auto-detected` note for new finds | `snapshot.subscriptions[]` |

**Rules.**

- New recurring merchants are added as `Auto-detected` (with a chat alert);
  costs update when the charge changes.
- Nothing is ever auto-removed (a missing stream can mean an annual plan or a
  different payment method); user-removed entries are never re-added.
- Excluded from detection: investment funding, transfers, card payments, loan
  payments.

---

## 7. Goals

**Purpose.** Savings goals with live progress.

| Element | Shows | Source |
|---|---|---|
| Goal cards | Name, target date, progress as **percentage/milestone** (no dollar amounts in titles), purpose line | `snapshot.savings_goals[]` + live cash |
| Progress detail | Current pace vs. required pace, projected date | *computed* from cash history |

**Rules.**

- Goal configuration (target, date, funding source) is user-managed — the
  daily refresh recomputes progress but never overwrites the config.
- Funding sources reference live balances (e.g. total cash), so progress moves
  with reality.

---

## 8. Plan

**Purpose.** The standing money plan — recurring investments and intentions.

| Element | Shows | Source |
|---|---|---|
| Plan items | Title + detail lines (e.g. `{amount}/week` auto-invests) | `snapshot.plan[]` |

**Rules.**

- Informational only — the hub never executes anything on this list.

---

## Data-source key

Quick lookup: where each prominent number comes from.

| UI number | Snapshot path / derivation |
|---|---|
| Net worth | `assets_total − debts_total` (computed) |
| Total assets | `investments_total + cash_total + home.estimated_value` (computed) |
| Total debt | `mortgage.balance + Σ cards.value` (computed) |
| Portfolio value | `investments_total` |
| Combined cash | `cash_total` |
| Home equity | `home.estimated_value − mortgage.balance` (computed) |
| Period spend | Σ transactions in selected period (computed) |
| Research universe | distinct stock tickers in holdings with value ≥ threshold (computed) |
| Bill amounts | `bills[]` (user/archive managed) |
| Goal progress | `savings_goals[]` config × live balances (computed) |

## Conventions used on every page

- Money is stored as a plain number; formatting (`$`, commas) happens in the UI.
- Optional fields are omitted when not applicable — the UI hides the row
  instead of showing a dash or zero.
- Charts read from append-only history logs; the snapshot itself carries only
  the current state.
- Every automated number is recomputed from source fields on each refresh —
  nothing is cached in a way that can go stale silently.
