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

Produce the purchase-market document for <municipality>, from live listings. entity_stats on listings is SECTION-based (0.50 credits per call; the 5 section calls = 2.50 credits, plus a per-object pull for the surface analysis). Listings section filters accept ONLY municipality / district / canton / postal_code / bbox + property_type (an ARRAY) + deal_type ("sale" | "rent", REQUIRED) + date_from / date_to (ISO with T+Z) + active (boolean) — no rooms/price filters. active: true = the CURRENT asking market; if a section returns too few listings for a small commune, retry without active but with date_from 12 months back. Follow these steps exactly:

1. HEADLINE LEVELS (0.50 cr)
   Call entity_stats with:
   { entity_type: "listings", filters: { municipality: "<municipality>", deal_type: "sale", active: true },
     section: "city_analysis" }
   Extract: median/avg price per m², the p5–p95 fan, and the per-property-type medians (apartments vs houses) for the KPI row.

2. PRICE BY ROOMS — BOTH UNITS (2 calls, 1.00 cr)
   Call entity_stats twice with { municipality: "<municipality>", deal_type: "sale", active: true }:
   once with section: "price_per_room" (total CHF) and once with section: "price_m2_per_room" (CHF/m²).

3. PRICE TREND (0.50 cr)
   Call entity_stats with:
   { entity_type: "listings", filters: { municipality: "<municipality>", deal_type: "sale", property_type: ["apartment"] },
     section: "price_m2_evolution" }
   Quarterly median asking price/m²; identify the peak quarter and the 12-month change.

4. TIME TO SELL (0.50 cr)
   Call entity_stats with:
   { entity_type: "listings", filters: { municipality: "<municipality>", deal_type: "sale", date_from: "<12 months ago, ISO with T+Z>" },
     section: "distribution" }
   Read time_on_market (share sold within 30/60/90/120 days, the average) and construction_year (average build year, max = off-plan deliveries) for the KPI row.

5. SURFACE ANALYSIS (recommended — per-object cost, preview first)
   search_entities listings { area: { municipality: "<municipality>" }, transaction_type: "sale", active: true } with preview: true FIRST (typically ~100–150 ads ≈ 5–8 credits; skip if the user is cost-sensitive); then compute the median asking price per surface band AND per rooms × surface cell (apartments).

6. THE DOCUMENT (blueprint — follow the kit)
   - Verdict + 4 KPIs: median CHF/m² · 12-month change · average days to sell · average build year of the offer (note off-plan deliveries when max year is ahead)
   - "Price by rooms": band chart (p30–p70 + median tick) with a CHF ↔ CHF/m² unit toggle; name where the m² premium sits and warn that thin bands price on comps
   - "Price by living area": bar ladder of median price per surface band (if step 5 ran); the analyst paragraph reads where the offer stacks and where it is shallow
   - "Where rooms and surface meet": heatmap of median asking price per cell (if step 5 ran), fading cells under 3 ads; name the liquid pockets and the scarce ones
   - "Price trend": quarterly line since ~2020, peak annotated, last quarter emphasised, apartments vs houses medians in the caption
   - "How fast homes sell": DOM bands with % labels; analyst paragraph linking the slow tail to over-anchored pricing — the listing-pitch argument
   Notes-line: asking prices, not realised; Vaud-style non-disclosure where relevant; thin bands. Next step: /popety:investment-yield for a specific property, or /popety:find-development-sites to source parcels — offer the more relevant one.

PRESENTATION — how to write the final answer (applies to every step above):
- The answer is a client-ready document that will be shared as-is. Not a chat log, not an analysis diary.
- Open with a title line (subject + commune + date) and a one-line verdict a reader would pay for. Never open with method, tool narration, or what you are about to do.
- The reader is a real-estate professional (broker, investor, developer). Every section closes with ONE analyst paragraph stating the professional implication — pricing, absorption, sourcing, underwriting — never a definition or a tutorial sentence. Show sample sizes and visibly de-emphasise figures resting on fewer than 3 observations.
- 3–6 titled sections, ordered by what matters most to the reader; one idea per section. Prefer short prose with embedded figures; use a table only when comparing 3+ items across 2+ dimensions, max ~6 rows, one comparison per table. Select the figures that change the reader's decision — do not dump every number you retrieved.
- Swiss formats: CHF 1'250'000 (apostrophe thousands), m², CHF/m²; official Swiss real-estate terminology in the reader's language. Write the whole document in the language the user wrote in.
- Caveats: at most ONE short "Notes" line at the end of the document (e.g. coverage gaps, asking ≠ realised prices). Never inline a disclaimer after a figure. Never mention tools, credits, section ids, API mechanics, or observations "for the platform team" inside the document — if you have a genuine data/product observation, put it after the document under a separate "---" divider, in one or two lines.
- Close the document with at most ONE "Next step" line offering the single most relevant follow-up — an offer, not a menu.
- In clients that support artifacts/canvas, render the document as a styled artifact (clean typography, generous spacing, no emojis) and keep the chat reply to a two-line summary; otherwise use clean markdown with real headings.

---

Requires the Popety connector (bundled with this plugin — authenticate on first use). Follow the workflow's cost gates: free/preview steps before any paid call.

## Building the document

When the client supports artifacts, render the final document as a plain HTML artifact (NEVER React/JSX — a JSX rewrite loses the stylesheet) built from `references/popety-kit.html` (bundled with this skill): read the kit FIRST, paste its ENTIRE <style> block verbatim as the stylesheet (never write your own CSS), give every SVG mark its literal hex fill/stroke attribute PLUS the kit mark class (mk1-mk4, mkband, mktick, mkline), and follow its primitives, helpers and rules — Popety brand in both themes, Swiss number formats, inline-SVG charts with tooltips, one analyst paragraph per section written for a real-estate professional. The finished document must visibly carry the Popety cyan (brandbar, kicker, verdict block, KPI accents, chart fills) — a grey/monochrome document means the kit was not applied. Without artifact support, apply the same structure in clean markdown.
