# Architecture

How the Finance Hub is put together. This is a reference design — adapt the
pieces to your own stack. All examples use synthetic data.

## 1. Big picture

```
┌─────────────┐     ┌──────────────┐     ┌─────────────────┐
│  Bank /      │     │  Snapshot    │     │  Hub UI         │
│  brokerage   │────▶│  store       │────▶│  (tabs)         │
│  (Plaid,     │ read│  (one JSON   │ read│  Overview /     │
│  read-only)  │-only│  document)   │-only│  Investments /  │
└─────────────┘     └──────────────┘     │  Spending /     │
       ▲                    ▲            │  Bills / Goals  │
       │                    │            └─────────────────┘
┌─────────────┐     ┌──────────────┐
│ Public web  │     │ Automations  │
│ (research,  │────▶│ (scheduled   │
│ earnings,   │     │  jobs)       │
│ treasury)   │     │              │
└─────────────┘     └──────────────┘
```

Three principles:

1. **One snapshot.** All hub state lives in a single JSON document
   (`docs/DATA_MODEL.md`). The UI renders from it; jobs update it.
2. **Read-only money.** Bank/brokerage access is strictly read-only
   (balances, holdings, transactions). The app cannot move money, pay bills,
   or place trades — by design, not just by policy.
3. **Automation with guardrails.** Scheduled jobs refresh data and watch for
   problems, but destructive or outward-facing actions (publishing, emailing)
   always stop at a human review step.

## 2. State and actions

The snapshot is updated through typed actions, not direct edits:

| Action | Scope |
|---|---|
| `get_snapshot` | Read the full current snapshot (always start here) |
| `update_snapshot` | **Replace** the whole snapshot — any omitted top-level section is wiped. Always rebuild from `get_snapshot`, then verify every section after writing |
| `set_research` | Patch only the research section |
| `set_treasury10y` | Patch only the treasury-yield section |
| `get_spending` / `refresh_spending_from_plaid` | Read / trigger async transaction sync |
| `upsert_bill` | Add or replace a single bill by `(name, period)` |

Rules that prevent data loss:

- Never push a partial snapshot through `update_snapshot`.
- Preserve user-managed sections verbatim on every write (settings, hidden
  accounts, savings-goal configuration, home-value estimate, bills ledger).
- After every write, read back and verify all sections are present.

## 3. Bank sync (Plaid, read-only)

Reads used: accounts, investment holdings, liabilities, transactions, and
recurring-transaction streams. Mapping is by institution + masked account
suffix (e.g. `…4412`), never by hardcoded account lists where avoidable:

- Every credit account found appears in the debt section (new cards show up
  automatically; a $0 balance is valid data, not a reason to hide it).
- Every depository account appears under cash; every investment account under
  investments.
- Investment holdings carry ticker, name, value, shares, and price.
- Subscriptions are auto-detected from recurring outflow streams, merged with
  the stored list: new merchants get added as "Auto-detected"; existing ones
  get cost updates; **nothing is ever auto-removed** (a missing stream can mean
  an annual plan or a different payment method); user-removed entries are never
  re-added. Investment funding, transfers, card payments, and loan payments are
  excluded from subscriptions.

## 4. Research pipeline

The stock-research tab follows the portfolio, not a fixed watchlist:

- **Universe** = distinct individual-stock tickers across all holdings with
  total position value ≥ a threshold (e.g. $500). ETFs, mutual funds, crypto,
  cash/money-market, options, and dust positions are excluded.
- **New qualifiers** get a full profile: business summary, 2–3 risks, sources.
- **Departed tickers** (sold below threshold) are dropped.
- **Metrics** — every company carries 8 standardized metrics, each with a
  display value, context, and a source link (never invented):
  trailing P/E, forward P/E, PEG, revenue, revenue growth, net margin,
  debt-to-equity, dividend yield.
- **Risk buckets** — `core` (profitable franchises), `watch` (cycle/valuation
  sensitive), `high` (execution-dependent). Buckets describe risk, not
  buy/sell instructions.
- **Earnings dates** merge from a separate curated file on every refresh;
  unknown dates stay unknown (never invented), and the card hides the line when
  absent.

## 5. Automations

| Job | Cadence | What it does |
|---|---|---|
| Daily refresh | Every morning | Pulls Plaid (accounts, holdings, transactions), rebuilds investments/cash/cards, refreshes research + treasury data, recomputes totals, appends to the net-worth log. Silent on success |
| Health check | After refresh | **Read-only watchdog.** Verifies transaction freshness (< 36h), every linked account is represented, bills and subscriptions are non-empty. Silent when healthy; one precise alert when not. Never repairs |
| Earnings reminders | Daily | Verifies next-day earnings for held stocks, sends a researched brief |
| Bill filing | Weekly | Files utility/mortgage bills from email and portals into document storage; feeds the bills ledger and balances into the hub |
| Spending sync | With refresh | Async transaction pull + freshness verification |

Notification routing is per-topic by user choice (e.g. earnings alerts to one
channel, trading-related reminders to another) — never broadcast everything
everywhere.

## 6. Privacy and safety

- Bank access is read-only and disconnectable at any time.
- The hub is passcode-protected; goal/progress surfaces avoid raw dollar
  amounts where the user prefers percentages.
- Full-snapshot writes are the main data-loss risk — mitigated by the
  read-modify-write-verify loop in §2.
- Automations never send money, place trades, or email anyone without an
  explicit human review step (drafts, not sends).
