---
name: development-activity
description: "Construction permits, transactions and new-build pulse of a commune. Use when the user asks about construction permits, building activity, new construction pulse of a commune."
disable-model-invocation: false
---

<!-- GENERATED FROM api/popety-mcp/src/prompts.ts — do not edit; run pnpm --filter @popety-io/mcp generate:integrations -->

# Development activity

Construction permits, transactions and new-build pulse of a commune.

**Arguments to collect from the user:**

- `municipality` — Municipality name, e.g. "Lausanne" or "Zürich". (required)

## Workflow

Assess construction and market activity in <municipality>. Follow these steps exactly:

1. PERMIT VOLUME AND TYPE
   Call entity_stats with:
   { entity_type: "permits", filters: { municipality: "<municipality>" },
     aggs: { by_year: { date_histogram: { field: "inquiry_start_date", calendar_interval: "year" } },
            by_type: { terms: { field: "regbl_classification_general.keyword", size: 10 } } } }
   NOTE: permit STATUS labels are only populated in Romandie — volume and type breakdowns work everywhere, so do not filter by status here.

2. TRANSACTION VOLUME BY YEAR
   Call entity_stats with:
   { entity_type: "transactions", filters: { municipality: "<municipality>" },
     aggs: { by_year: { date_histogram: { field: "transaction_date", calendar_interval: "year" } } } }
   CAVEAT: transaction records cover French-speaking cantons and Ticino only — if <municipality> is in DE-CH, skip this step and say so.

3. CURRENT SALE INVENTORY
   Call entity_stats with:
   { entity_type: "listings", filters: { municipality: "<municipality>", deal_type: "sale", active: true },
     aggs: { by_new_construction: { terms: { field: "new_construction" } } } }
   Extract: active sale listing count and, where populated, the new-construction share.

4. NARRATIVE
   Answer: is construction activity rising or falling (permit trend)? Is transaction activity rising or falling (where covered)? What type of development dominates (new construction vs transformation)? How much is currently for sale, and how much of it is new-build? Cite the data gaps that apply (DE-CH transactions, permit status).

---

Requires the Popety connector (bundled with this plugin — authenticate on first use). Follow the workflow's cost gates: free/preview steps before any paid call.
