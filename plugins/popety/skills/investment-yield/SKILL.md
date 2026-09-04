---
name: investment-yield
description: "Gross-yield check: AVM purchase and rent estimates vs current asking rents. Use when the user asks about rental yield, gross yield, buy-to-let return, investment return on a property."
disable-model-invocation: false
---

<!-- GENERATED FROM api/popety-mcp/src/prompts.ts — do not edit; run pnpm --filter @popety-io/mcp generate:integrations -->

# Investment yield

Gross-yield check: AVM purchase and rent estimates vs current asking rents.

**Arguments to collect from the user:**

- `address` — Full Swiss address of the property. Supply at least one of address or building_id. (optional)
- `building_id` — Popety building identifier. Supply at least one of address or building_id. (optional)

## Workflow

Assess the gross rental yield of the Swiss property at "<address>". Follow these steps exactly:

1. RESOLVE THE PROPERTY (free)
   Call estimate_property with { address: "<address>" } (no confirm flag).
   This returns the register facts and the unit attributes still required: living_area, rooms_nb, bathroom_nb.

2. CONFIRM ATTRIBUTES WITH THE USER
   Ask the user for living_area, rooms_nb, and bathroom_nb (or confirm the returned suggestion for single-dwelling buildings). NEVER guess or invent them.

3. RUN THE VALUATION (paid, 3 credits)
   Call estimate_property again with the same target plus { confirm: true, property_attributes: { living_area, rooms_nb, bathroom_nb } }.
   Extract: estimated purchase price, estimated monthly rent, and the gross yield band yield_pct_lower / yield_pct_upper.

4. MARKET-RENT CONTEXT
   Call entity_stats with { entity_type: "listings", filters: { municipality: <municipality>, deal_type: "rent", active: true }, aggs: { asking_rent: { stats: { field: "price_per_square_meter" } } } }.
   Compare the model rent per m² to current asking rents in the commune.

5. PRESENT THE YIELD
   - Gross yield: <yield_pct_lower>% – <yield_pct_upper>% (from the estimate)
   - Model rent: CHF <rent>/month vs current asking rents per m² in the commune
   - Verdict: is the implied yield above or below the local asking-rent picture?
   - Caveats: asking rents ≠ realised rents; the yield is GROSS — it ignores charges, vacancy, maintenance, and taxes; this is an AI estimate, not investment advice or a formal appraisal.

---

Requires the Popety connector (bundled with this plugin — authenticate on first use). Follow the workflow's cost gates: free/preview steps before any paid call.
