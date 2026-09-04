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

4. CURRENT ASKING PRICES (cross-check — works everywhere)
   Call entity_stats with { entity_type: "listings", filters: { municipality: <municipality>, deal_type: "sale", active: true }, aggs: { asking: { stats: { field: "price_per_square_meter" } } } }.
   Compare the estimate's price per m² to the current asking-price stats.

5. COMPARABLE TRANSACTIONS (only where recorded)
   Transaction records cover French-speaking cantons and Ticino; Zürich and most DE-CH areas have none — skip this step there and rely on the listings cross-check. Where covered, call search_entities with { entity_type: "transactions", filters: { municipality: <municipality>, date_from: "<2 years ago>", step_name: "sale" } } and select up to 5 comparables.

6. PRESENT THE ESTIMATE
   - Estimated value: CHF <value> (range: CHF <low> – CHF <high>)
   - Estimated rent: CHF <rent>/month
   - Price per m²: CHF <ppm2> vs current asking median in the commune
   - Comparables: transactions where available, otherwise active-listing stats
   - Methodology caveat: the estimate is AI-generated from the confirmed property attributes and market data; it does not substitute for a formal appraisal. Listing figures are ASKING prices (typically above realised prices). In non-disclosure cantons (VD/VS/FR/TI), transaction prices are largely unpublished and confidence bands are wider.

---

Requires the Popety connector (bundled with this plugin — authenticate on first use). Follow the workflow's cost gates: free/preview steps before any paid call.
