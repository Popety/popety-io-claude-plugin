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

Summarise current market conditions (asking prices and transaction trends) for <municipality>. Each entity_stats call costs 1 flat credit and accepts SEVERAL named aggregations — bundle them as below (3 calls total). Follow these steps exactly:

1. ACTIVE RENTAL MARKET (one call, three aggs)
   Call entity_stats with:
   { entity_type: "listings", filters: { municipality: "<municipality>", deal_type: "rent", active: true },
     aggs: { rent_per_m2: { stats: { field: "price_per_square_meter" } },
             by_rooms: { terms: { field: "rooms_nb", size: 12 },
                         aggs: { median_rent: { percentiles: { field: "price", percents: [50] } } } },
             by_category: { terms: { field: "property_category.keyword", size: 8 } } } }
   Extract: inventory count, asking rent per m² (avg/min/max), median rent per room count, property-type mix.

2. ACTIVE SALE MARKET (one call, four aggs)
   Call entity_stats with:
   { entity_type: "listings", filters: { municipality: "<municipality>", deal_type: "purchase", active: true },
     aggs: { sale_per_m2: { stats: { field: "price_per_square_meter" } },
             by_rooms: { terms: { field: "rooms_nb", size: 12 },
                         aggs: { median_price: { percentiles: { field: "price", percents: [50] } } } },
             by_category: { terms: { field: "property_category.keyword", size: 8 } },
             new_vs_resale: { terms: { field: "new_construction" },
                              aggs: { median_per_m2: { percentiles: { field: "price_per_square_meter", percents: [50] } } } } } }
   Extract: inventory count, asking price per m², median price per room count, type mix, new-construction share and its per-m² premium vs resale.

3. TRANSACTION ACTIVITY (coverage-gated — one call, two aggs)
   Land-registry transactions are indexed ONLY for French-speaking cantons and Ticino (VD, GE, VS, FR, NE, JU, TI). If <municipality> is not in one of these cantons, SKIP this step and omit the transactions section entirely (do not report it as missing data). Registered prices are published only in GE, NE and JU — everywhere else report volume and type mix only, never price statistics.
   Call entity_stats with:
   { entity_type: "transactions", filters: { municipality: "<municipality>" },
     aggs: { by_year: { date_histogram: { field: "transaction_date", calendar_interval: "year" },
                        aggs: { price_stats: { stats: { field: "transaction_total_price" } } } },
             by_type: { terms: { field: "transaction_step_name", size: 12 } } } }

4. SUMMARY — present as a compact market dashboard
   - Rental: active inventory, asking rent/m² range, median rent by rooms, type mix
   - Sale: active inventory, asking price/m² range, median price by rooms, new-build share + premium
   - Transactions (covered cantons only): count per year (last 5), price trend (GE/NE/JU only), mix of sale vs inheritance/transfer types
   Cite data gaps where applicable (non-disclosure canton, sparse data; asking prices are advertised, not realised). Offer deeper dives: /popety:development-activity for the construction pipeline, /popety:investment-yield for a specific property.

---

Requires the Popety connector (bundled with this plugin — authenticate on first use). Follow the workflow's cost gates: free/preview steps before any paid call.
