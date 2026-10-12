# Features

A tour of each tab. All examples use synthetic data.

## Overview

The landing view: net worth with history, assets vs. debt breakdown, cash and
investment totals. Designed to answer "where do I stand?" in one glance.

## Investments

- Every linked account (brokerage, retirement, 401k, 529, crypto) with its
  holdings: ticker, name, value, shares, price.
- Totals reconcile against the accounts — a cash/margin line is kept as a
  holding so the math ties out.
- **Research tab** — one card per held stock:
  - Business summary in plain language
  - 8 standardized metrics: trailing P/E, forward P/E, PEG, revenue, revenue
    growth, net margin, debt-to-equity, dividend yield — each with context and
    a source link
  - 2–3 current risks, source links, and caveats
  - Next earnings date (when known; hidden when unknown)
  - Risk bucket: **Stronger core** (profitable franchises), **Watch closely**
    (cycle/valuation sensitive), **Highest risk** (execution-dependent)
- The research universe is **dynamic**: it follows actual holdings. Buy a new
  stock above the threshold and a card appears; sell it and the card goes away.
  No hardcoded ticker lists.

## Spending

- Transactions synced from linked banks/cards, refreshed daily.
- **Categories** — a fixed dropdown of your existing categories (no free text,
  so typos can't corrupt the data). Recategorizing updates budgets.
- **Budgets** — per-category budgets whose progress recomputes from current
  transactions whenever data refreshes or a transaction is recategorized.
- **Period totals** — the first tile totals exactly the transactions listed and
  follows the selected time period (month / 3 months / year / all).
- Overlays open in place at the current scroll position and restore it on close.

## Bills

Only genuine household bills the user pays: utilities, car payments, and similar.
Each bill shows amount, billing period, due date, and status (Autopay / Due /
Paid / Escrow).

Deliberate rules:

- **Subscriptions are never bills** — they live only under Subscriptions.
- Escrow-paid items (e.g. insurance paid through a mortgage) are informational,
  not presented as bills the user pays.
- The bills total is a plain number, not a tile.
- Month / 3-month / Year / All views plus a month picker; defaults to the
  current month.

## Subscriptions

Recurring charges auto-detected from transaction history. New merchants appear
as "Auto-detected" (with a chat alert); costs update when the charge changes.
Nothing is ever auto-removed, and user-removed entries are never re-added.

## Goals

Savings goals with live progress computed from cash balances and observed
savings pace — e.g. an opportunity/crash reserve and a rental-property fund.
Goal surfaces use percentages and milestone language rather than raw dollar
amounts where the user prefers it. Goal configuration is user-managed; the
daily refresh never overwrites it.

## Guardrails across the app

- Read-only bank access: the app can never move money, pay bills, or trade.
- Passcode protection on the hub.
- Full-snapshot writes use read-modify-write-verify so no section is ever
  silently dropped.
- Automations are silent on success and precise on failure.
