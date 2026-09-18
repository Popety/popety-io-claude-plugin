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

Summarise current market conditions (asking prices and transaction trends) for <municipality>. entity_stats on listings/transactions is SECTION-based: each call computes ONE curated section id and costs 0.50 credits (the 4 core calls below ≈ 2 credits; the optional 5th makes it 2.50). Listings section filters accept ONLY municipality / district / canton / postal_code / bbox + property_type (an ARRAY) + deal_type ("sale" | "rent", REQUIRED) + date_from / date_to (ISO with T+Z) + active (boolean) — no rooms/price filters. For CURRENT market levels add active: true; for a recent-window read add date_from (e.g. 12 months back); omit both for the full 2006–present history. Follow these steps exactly:

1. RENT LEVELS BY ROOM COUNT (one call, 0.50 cr)
   Call entity_stats with:
   { entity_type: "listings", filters: { municipality: "<municipality>", deal_type: "rent", active: true },
     section: "price_per_room" }
   (active: true = the CURRENT asking market. If the section returns too few listings for a small commune, retry without active but with date_from 12 months back.)
   Extract: all-rooms median/average rent, median rent per room band, and the property-type mix (per-type counts).

2. SALE PRICE LEVELS PER M² (one call, 0.50 cr)
   Call entity_stats with:
   { entity_type: "listings", filters: { municipality: "<municipality>", deal_type: "sale", active: true },
     section: "city_analysis" }
   Extract: price/m² stats (avg/std-dev), the p5–p95 percentile fan, and the per-property-type median. (city_analysis works at municipality level; price_geography does NOT — it only drills below a canton or district filter.)

3. NEW-BUILD VS RESALE (one call, 0.50 cr)
   Call entity_stats with:
   { entity_type: "listings", filters: { municipality: "<municipality>", deal_type: "sale" },
     section: "new_vs_resale" }
   Extract: new vs existing counts, median price/m² for each (the new-build premium), and the recent quarterly counts.

4. TRANSACTION TREND (coverage-gated — one call, 0.50 cr)
   Land-registry transactions are indexed ONLY for French-speaking cantons and Ticino (VD, GE, VS, FR, NE, JU, TI). If <municipality> is not in one of these cantons, SKIP steps 4 and 5 and omit the transactions section entirely (do not report it as missing data). Registered prices are published only in GE, NE and JU — everywhere else report volume and type mix only, never price statistics.
   Call entity_stats with:
   { entity_type: "transactions", filters: { municipality: "<municipality>" },
     section: "price_dynamics" }
   Extract: per-year deal count and, in GE/NE/JU only, the median/average sold price and total volume.

5. TRANSACTION TYPE MIX (optional, same coverage gate — one call, 0.50 cr)
   Call entity_stats with:
   { entity_type: "transactions", filters: { municipality: "<municipality>" },
     section: "transaction_types" }
   Extract: share of true market sales vs inheritance/transfer types.

6. THE DOCUMENT
   Sections to cover:
   - Rental market: median rent by rooms, all-rooms median/average, type mix
   - Sale market: price/m² stats + percentile fan, per-type medians, new-build share + premium
   - Transactions (covered cantons only): count per year (last 5), price trend (GE/NE/JU only), mix of sale vs inheritance/transfer types
   Read the latest quarters for the current picture — section figures span the full listing history. Notes-line material: non-disclosure canton, sparse data, asking prices are advertised, not realised. Next-step candidates: /popety:find-development-sites to source under-exploited parcels, or /popety:investment-yield for a specific property — offer the more relevant one.

PRESENTATION — how to write the final answer (applies to every step above):
- The answer is a client-ready document that will be shared as-is. Not a chat log, not an analysis diary.
- Open with a title line (subject + commune + date) and a one-line verdict a reader would pay for. Never open with method, tool narration, or what you are about to do.
- 3–6 titled sections, ordered by what matters most to the reader; one idea per section. Prefer short prose with embedded figures; use a table only when comparing 3+ items across 2+ dimensions, max ~6 rows, one comparison per table. Select the figures that change the reader's decision — do not dump every number you retrieved.
- Swiss formats: CHF 1'250'000 (apostrophe thousands), m², CHF/m²; official Swiss real-estate terminology in the reader's language. Write the whole document in the language the user wrote in.
- Caveats: at most ONE short "Notes" line at the end of the document (e.g. coverage gaps, asking ≠ realised prices). Never inline a disclaimer after a figure. Never mention tools, credits, section ids, API mechanics, or observations "for the platform team" inside the document — if you have a genuine data/product observation, put it after the document under a separate "---" divider, in one or two lines.
- Close the document with at most ONE "Next step" line offering the single most relevant follow-up — an offer, not a menu.
- In clients that support artifacts/canvas, render the document as a styled artifact (clean typography, generous spacing, no emojis) and keep the chat reply to a two-line summary; otherwise use clean markdown with real headings.

---

Requires the Popety connector (bundled with this plugin — authenticate on first use). Follow the workflow's cost gates: free/preview steps before any paid call.
