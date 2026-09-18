---
name: purchase-market-overview
description: "Asking sale prices per m²: distribution, per-type medians and multi-year trend for a Swiss commune. Use when the user asks about sale prices, asking prices, purchase market, price per m2 or price trends in a commune."
disable-model-invocation: false
---

<!-- GENERATED FROM api/popety-mcp/src/prompts.ts — do not edit; run pnpm --filter @popety-io/mcp generate:integrations -->

# Purchase market overview

Asking sale prices per m²: distribution, per-type medians and multi-year trend for a Swiss commune.

**Arguments to collect from the user:**

- `municipality` — Municipality name, e.g. "Lausanne" or "Zürich". (required)

## Workflow

Produce the purchase-market document for <municipality>, from live listings. entity_stats on listings is SECTION-based (0.50 credits per call; the 2 calls = 1.00 credit). Listings section filters accept ONLY municipality / district / canton / postal_code / bbox + property_type (an ARRAY) + deal_type ("sale" | "rent", REQUIRED) + date_from / date_to (ISO with T+Z) + active (boolean) — no rooms/price filters. active: true = the CURRENT asking market; if a section returns too few listings for a small commune, retry without active but with date_from 12 months back. Follow these steps exactly:

1. TODAY'S OFFER (0.50 cr)
   Call entity_stats with:
   { entity_type: "listings", filters: { municipality: "<municipality>", deal_type: "sale", active: true },
     section: "city_analysis" }
   Extract: price/m² stats (avg/std-dev), the p5–p95 percentile fan, the CHF-500 histogram bands, and the per-property-type medians (apartments vs houses). (city_analysis works at municipality level; price_geography does NOT — it only drills below a canton or district filter.)

2. PRICE TREND (0.50 cr)
   Call entity_stats with:
   { entity_type: "listings", filters: { municipality: "<municipality>", deal_type: "sale", property_type: ["apartment"] },
     section: "price_m2_evolution" }
   Quarterly median asking price/m² for newly listed apartments. Read the last ~6 years; identify the peak quarter and compute the 12-month change for the KPI row.

3. THE DOCUMENT
   Sections: verdict + KPI row (median CHF/m², 12-month change, priced live ads, apartment vs house medians) · Today's offer (distribution with the middle half shaded and the median marked; where the top 5% starts) · Price trend since 2020 (peak annotated, last quarter emphasised). Notes-line material: asking prices, not realised; thin per-type samples. Next step: /popety:investment-yield for a specific property, or /popety:find-development-sites to source parcels — offer the more relevant one.

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
