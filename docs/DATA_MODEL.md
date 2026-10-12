# Data model

The hub's entire state is one JSON snapshot. This is the schema, with
**synthetic example values** — every figure here is invented for illustration.

```jsonc
{
  "as_of": "2026-10-12",          // snapshot date (YYYY-MM-DD)
  "currency": "USD",
  "net_worth": 512340.55,         // assets_total - debts_total
  "assets_total": 980120.44,
  "debts_total": 467779.89,       // mortgage.balance + sum(cards.value)

  "investments_total": 402150.75,
  "investments": [
    {
      "institution": "Example Brokerage",
      "account": "Individual …1234",
      "value": 88120.40,
      "holdings": [
        { "ticker": "EGCO", "name": "Example Growth Co", "value": 15230.10,
          "shares": 210.5, "price": 72.35 },
        { "ticker": "EETF", "name": "Example Index Fund", "value": 42100.00 }
      ],
      "note": "Taxable brokerage"
    }
  ],

  "cash_total": 45230.10,
  "cash": [
    { "institution": "Example Bank", "account": "Savings …5678",
      "value": 38500.00, "note": "2.8% APY" },
    { "institution": "Example Bank", "account": "Checking …9012",
      "value": 6730.10 }
  ],

  "cards": [
    { "institution": "Example Bank", "account": "Rewards Card …3456",
      "value": 412.20 }
  ],

  "mortgage": {
    "institution": "Example Mortgage Co",
    "balance": 467367.69,
    "rate": "5.5%",
    "monthly_payment": 3200.00,
    "due": "1st",
    "autopay": true,
    "extra_principal": 500
  },

  "home": {
    "location": "Example City, ST",
    "estimated_value": 532739.59,   // manual estimate; equity = value − balance
    "estimate_source": "Example estimator · Oct 2026",
    "purchase_price": 550000,
    "equity": 65371.90
  },

  "bills": [
    { "name": "Example Power Co", "amount": 85.40, "period": "2026-10",
      "due_date": "2026-10-20", "status": "Autopay" },
    { "name": "Example Internet", "amount": 70.00, "period": "2026-10",
      "due_date": "2026-10-07", "status": "Autopay" }
  ],
  // Bills = only real household bills the user pays (utilities, car
  // payments). Subscriptions live separately. Escrow-paid items are
  // informational, not bills.

  "subscriptions": [
    { "name": "Example Streaming", "cost": "$15.99/mo" },
    { "name": "Example Cloud Storage", "cost": "$2.99/mo",
      "note": "Auto-detected" }
  ],

  "research": {
    "as_of": "2026-10-12",
    "note": "Risk buckets, not trade instructions.",
    "buckets": [
      { "id": "core",  "title": "Stronger core",
        "detail": "Profitable franchises.", "tickers": ["EGCO"] },
      { "id": "watch", "title": "Watch closely",
        "detail": "Cycle/valuation sensitivity.", "tickers": ["ECYC"] },
      { "id": "high",  "title": "Highest risk",
        "detail": "Execution-dependent.", "tickers": ["ESPC"] }
    ],
    "companies": [
      {
        "ticker": "EGCO",
        "name": "Example Growth Co",
        "bucket": "core",
        "summary": "Concise (~500–700 chars): what the business does today and its current financial state.",
        "metrics": [
          { "key": "trailing_pe",    "value": "24.5x",   "context": "vs 5-yr avg 22x",
            "source_label": "Example provider", "source_url": "https://example.com/egco" },
          { "key": "forward_pe",     "value": "21.0x",   "context": "On next-12-mo estimates",
            "source_label": "Example provider", "source_url": "https://example.com/egco" },
          { "key": "peg",            "value": "1.4x",    "context": "Directional only",
            "source_label": "Example provider", "source_url": "https://example.com/egco" },
          { "key": "revenue",        "value": "$48.2B TTM", "context": "FY2026",
            "source_label": "Example provider", "source_url": "https://example.com/egco" },
          { "key": "revenue_growth", "value": "+18% YoY", "context": "Latest quarter",
            "source_label": "Example provider", "source_url": "https://example.com/egco" },
          { "key": "margin",         "value": "29.8%",    "context": "Net margin TTM",
            "source_label": "Example provider", "source_url": "https://example.com/egco" },
          { "key": "debt_to_equity", "value": "0.31x",   "context": "Conservative balance sheet",
            "source_label": "Example provider", "source_url": "https://example.com/egco" },
          { "key": "dividend_yield", "value": "0.9%",     "context": "$X.XX annualized",
            "source_label": "Example provider", "source_url": "https://example.com/egco" }
        ],
        "risks": ["Two or three specific, current risks."],
        "sources": [
          { "label": "Example provider", "url": "https://example.com/egco" }
        ],
        "caveat": "Any open questions or data limitations.",
        "next_earnings": { "date": "2026-10-28", "timing": "after_close",
                           "source_url": "https://example.com/egco/earnings" }
        // Omit next_earnings entirely when the date is unknown — never invent one.
      }
    ]
  },

  "treasury10y": {
    "as_of": "2026-10-08",
    "current_yield": 4.20,            // percent, not seasonally adjusted
    "unit": "percent",
    "frequency": "Monthly averages · latest daily observation",
    "history": [ { "date": "2026-09-30", "value": 4.18 } ],
    "source_label": "FRED · DGS10",
    "source_url": "https://fred.stlouisfed.org/series/DGS10/"
  },

  "savings_goals": [
    {
      "id": "opportunity-fund",
      "name": "Opportunity fund",
      "target_amount": 100000,
      "target_date": "2027-06-30",
      "purpose": "Cash reserve for market dislocations.",
      "funding_source": { "kind": "total_cash" },
      "fallback_monthly_rate": 3000,
      "created_at": "2026-01-15T00:00:00.000Z",
      "updated_at": "2026-10-12T00:00:00.000Z"
    }
  ],

  "plan": [
    { "title": "Example recurring investment", "detail": "$100/week" }
  ],

  "hidden_accounts": []   // user-managed visibility toggles; preserved verbatim
}
```

## Conventions

- **Money** is a number (no currency symbols); display formatting happens in the UI.
- **Omit** optional keys (`note`, `detail`, `shares`, `price`, `caveat`,
  `next_earnings`) when not applicable.
- **Metrics** use `"N/M"` or `"None"` for not-meaningful values (e.g. P/E for an
  unprofitable company, dividend yield for a non-payer) — never `0`, which
  implies a measured zero.
- **Derived totals** are recomputed on every write:
  `debts_total = mortgage.balance + Σ cards.value`;
  `assets_total = investments_total + cash_total + home.estimated_value`;
  `net_worth = assets_total − debts_total`;
  `home.equity = home.estimated_value − mortgage.balance`.
