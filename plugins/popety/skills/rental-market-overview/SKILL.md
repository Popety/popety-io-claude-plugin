---
name: rental-market-overview
description: "Asking rents by rooms and per m², rent trend and inventory mix for a Swiss commune. Use when the user asks about rents, asking rents, rental market, rent levels or rent trends in a commune."
disable-model-invocation: false
---

<!-- GENERATED FROM api/popety-mcp/src/prompts.ts — do not edit; run pnpm --filter @popety-io/mcp generate:integrations -->

# Rental market overview

Asking rents by rooms and per m², rent trend and inventory mix for a Swiss commune.

**Arguments to collect from the user:**

- `municipality` — Municipality name, e.g. "Lausanne" or "Zürich". (required)

## Workflow

Produce the rental-market document for <municipality>, from live listings. entity_stats on listings is SECTION-based (0.50 credits per call; the 3 core calls = 1.50 credits). Listings section filters accept ONLY municipality / district / canton / postal_code / bbox + property_type (an ARRAY) + deal_type ("sale" | "rent", REQUIRED) + date_from / date_to (ISO with T+Z) + active (boolean) — no rooms/price filters. active: true = the CURRENT asking market; if a section returns too few listings for a small commune, retry without active but with date_from 12 months back. Follow these steps exactly:

1. RENT BY ROOM COUNT (0.50 cr)
   Call entity_stats with:
   { entity_type: "listings", filters: { municipality: "<municipality>", deal_type: "rent", active: true },
     section: "price_per_room" }
   Extract: all-rooms median/average, median + p30–p70 per room band, and the property-type mix (apartments vs parking/commercial/rooms counts).

2. RENT PER M² BY ROOM COUNT (0.50 cr)
   Call entity_stats with:
   { entity_type: "listings", filters: { municipality: "<municipality>", deal_type: "rent", active: true },
     section: "price_m2_per_room" }
   The per-m² view inverts the story — small flats pay the premium. Present BOTH units for the by-rooms figures (CHF/month and CHF/m²; in an artifact, a unit toggle).

3. RENT TREND (0.50 cr)
   Call entity_stats with:
   { entity_type: "listings", filters: { municipality: "<municipality>", deal_type: "rent", property_type: ["apartment"] },
     section: "price_evolution" }
   Quarterly median asking rent for newly listed apartments. Read the last ~6 years; compute the 12-month change for the KPI row.

4. ROOMS × SURFACE MATRIX (optional — per-object cost, preview first)
   Only when the user wants the full picture: search_entities listings { area: { municipality: "<municipality>" }, transaction_type: "rent", property_type: "apartment", active: true } with preview: true FIRST (typically ~100–150 ads ≈ 5–8 credits); then compute the median rent per rooms × surface-band cell and present it as a heatmap, fading cells with fewer than 3 ads.

5. THE DOCUMENT
   Sections: verdict + KPI row (median rent, median CHF/m², live apartment ads, 12-month trend) · Rent by rooms (both units, p30–p70 band + median) · Where rooms and surface meet (only if step 4 ran) · Rent trend since 2020 · one line on the non-apartment inventory mix. Notes-line material: asking rents, not contract rents; sparse bands. Next step: /popety:listing-hunter for a client shortlist, or /popety:purchase-market-overview for the sale side — offer the more relevant one.

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
