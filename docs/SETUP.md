# Setup — rebuild it for yourself

This repo documents the project; it is not a runnable app. This guide explains
how to build your own instance from the architecture.

## 1. Prerequisites

- **Plaid** (or another aggregation API): a development/production app with
  access to `accounts`, `holdings`, `liabilities`, `transactions`, and
  recurring-transactions. **Request read-only scopes.** Your app must never be
  able to move money.
- **A hosting/runtime** for a TypeScript full-stack app with a JSON document
  store (any database works — the design only needs one versioned snapshot
  document plus an append-only history log).
- **A scheduler** for the automations (cron or equivalent).
- **Public data sources** for research: any quote/fundamentals provider for the
  8 metrics, plus an earnings-calendar source. Record the source URL with every
  figure — never invent one.

## 2. Build order

1. **Snapshot store + actions.** Implement `get_snapshot`, `update_snapshot`
   (replace semantics), and the targeted patch actions. Add the
   read-modify-write-verify loop from `ARCHITECTURE.md` §2 before anything else.
2. **Overview + Investments UI.** Render accounts and holdings from the snapshot;
   compute the derived totals exactly as in `DATA_MODEL.md`.
3. **Plaid sync.** Map institutions to snapshot sections (§3 of ARCHITECTURE.md).
   Include *every* account the API returns — never hardcode the list.
4. **Research pipeline.** Implement the dynamic universe rule, the 8-metric
   profile shape, risk buckets, and the earnings-date merge.
5. **Spending.** Transaction sync, fixed category list, budgets computed from
   effective categories, period totals.
6. **Bills + Subscriptions.** Keep the two strictly separate per the rules in
   `FEATURES.md`.
7. **Goals.** Progress from cash balances; never let the refresh overwrite
   goal configuration.
8. **Automations.** Daily refresh, read-only health check, earnings reminders —
   silent on success, precise on failure.

## 3. Privacy checklist (do this before connecting real accounts)

- [ ] Bank/brokerage connections are read-only; verify at the provider.
- [ ] The app has no code path that can transfer money, pay bills, or trade.
- [ ] The hub is behind authentication (passcode at minimum).
- [ ] No financial figures are logged, cached, or rendered anywhere unintended
      (notifications, goal titles, exports).
- [ ] Backups/exports exclude credentials and tokens.

## 4. What this repo intentionally omits

- Real account data, balances, holdings, bills — everything here is synthetic.
- API keys, tokens, and credentials of any kind.
- Any single-vendor internals — the architecture is described generically so it
  can be rebuilt on any stack.
