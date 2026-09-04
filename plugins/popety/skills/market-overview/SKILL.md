---
name: market-overview
description: "Asking-price levels and transaction trends for a Swiss commune. Use when the user asks about market prices, asking prices, price or rent trends, market conditions in a commune."
disable-model-invocation: false
---

<!-- GENERATED FROM api/popety-mcp/src/prompts.ts — do not edit; run pnpm --filter @popety-io/mcp generate:integrations -->

# Market overview

Asking-price levels and transaction trends for a Swiss commune.

**Arguments to collect from the user:**

- `municipality` — Municipality name, e.g. "Lausanne" or "Zürich". (required)
- `canton` — ISO 2-letter canton code, e.g. "VD". Used to flag whether transaction prices are disclosed in this canton. (optional)

## Workflow

Summarise current market conditions (asking prices and transaction trends) for <municipality>. Follow these steps exactly:

1. ACTIVE RENTAL ASKING LEVELS
   Call entity_stats with:
   { entity_type: "listings", filters: { municipality: "<municipality>", deal_type: "rent", active: true },
     aggs: { rent_per_m2: { stats: { field: "price_per_square_meter" } } } }
   Extract: count, min, max, avg, stddev of asking rent per m² (CHF).

2. ACTIVE SALE ASKING LEVELS
   Call entity_stats with:
   { entity_type: "listings", filters: { municipality: "<municipality>", deal_type: "purchase", active: true },
     aggs: { sale_per_m2: { stats: { field: "price_per_square_meter" } } } }
   Extract: count, min, max, avg of asking sale price per m² (CHF).

3. TRANSACTION VOLUME AND PRICE TREND BY YEAR
   Call entity_stats with:
   { entity_type: "transactions", filters: { municipality: "<municipality>" },
     aggs: { by_year: { date_histogram: { field: "transaction_date", calendar_interval: "year" },
                        aggs: { price_stats: { stats: { field: "transaction_total_price" } } } } } }

4. SUMMARY
   Present: median asking rent/m², median asking sale price/m², transaction count per year (last 5 years), and whether transaction price trend is rising, stable, or falling. Cite data gaps where applicable (non-disclosure canton, sparse data).

---

Requires the Popety connector (bundled with this plugin — authenticate on first use). Follow the workflow's cost gates: free/preview steps before any paid call.
