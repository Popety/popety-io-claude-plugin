---
name: value-estimate
description: "AVM valuation with user-confirmed attributes, cross-checked against the market. Use when the user asks about property valuation, market value, how much a property is worth, price estimate."
disable-model-invocation: false
---

<!-- GENERATED FROM api/popety-mcp/src/prompts.ts — do not edit; run pnpm --filter @popety-io/mcp generate:integrations -->

# Value estimate

AVM valuation with user-confirmed attributes, cross-checked against the market.

**Arguments to collect from the user:**

- `address` — Full Swiss address of the property. Supply at least one of address or building_id. (optional)
- `building_id` — Popety building identifier. Supply at least one of address or building_id. (optional)

## Workflow

Estimate the market value of the Swiss property at "<address>". Follow these steps exactly:

1. RESOLVE THE PROPERTY (free)
   Call estimate_property with { address: "<address>" } (no confirm flag).
   This returns the register facts (address, building year, category) and the unit attributes still required: living_area, rooms_nb, bathroom_nb.

2. CONFIRM ATTRIBUTES WITH THE USER
   Ask the user for the required attributes (or confirm the returned suggestion for single-dwelling buildings). NEVER guess or invent them — the valuation is only as good as these inputs.

3. RUN THE VALUATION (paid, 3 credits)
   Call estimate_property again with the same target plus { confirm: true, property_attributes: { living_area, rooms_nb, bathroom_nb, ... } }.
   Extract: estimated purchase price, estimated rent, price per m², and confidence band.

4. CURRENT ASKING PRICES (cross-check — works everywhere, 0.50 credits)
   Call entity_stats with { entity_type: "listings", filters: { municipality: <municipality>, deal_type: "sale" }, section: "city_analysis" }.
   (Sections take NO active/date/rooms filters — deal_type is required, property_type would be an array.) Read the city's price/m² stats and percentile fan (p25/p50/p75) and compare the estimate's price per m² to them. Figures span the full listing history, 2% trimmed — treat the median as an asking-level anchor.

5. COMPARABLE TRANSACTIONS (only where recorded)
   Transaction records cover French-speaking cantons and Ticino; Zürich and most DE-CH areas have none — skip this step there and rely on the listings cross-check. Where covered, call search_entities with { entity_type: "transactions", filters: { municipality: <municipality>, date_from: "<2 years ago>", step_name: "sale" } } and select up to 5 comparables.

6. THE DOCUMENT (blueprint — follow the kit)
   - Verdict + estimate card: the value as a RANGE VISUAL (lower–mid–upper band with the mid marked, kit band+tick primitive) and the confidence band named
   - 4 KPIs: estimated value · CHF/m² vs the commune's asking median · estimated rent/month · implied gross yield
   - "Against the market": the property's CHF/m² plotted against the commune percentile fan (city_analysis) — say WHERE in the market it sits
   - "Comparables": transactions where available, otherwise active-listing stats
   Notes-line material: AI estimate from the confirmed attributes and market data, not a formal appraisal; listing figures are ASKING prices (typically above realised prices); in non-disclosure cantons (VD/VS/FR/TI) transaction prices are largely unpublished and confidence bands are wider.

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

When the client supports artifacts, render the final document as an artifact built from `references/popety-kit.html` (bundled with this skill): read the kit FIRST and follow its tokens, primitives, helpers and rules verbatim — Popety brand, both themes, Swiss number formats, inline-SVG charts with tooltips, one analyst paragraph per section written for a real-estate professional. Without artifact support, apply the same structure in clean markdown.
