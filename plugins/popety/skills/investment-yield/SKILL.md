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

4. MARKET-RENT CONTEXT (0.50 credits)
   Call entity_stats with { entity_type: "listings", filters: { municipality: <municipality>, deal_type: "rent" }, section: "city_analysis" }.
   (Sections take NO active/date/rooms filters — deal_type is required.) Read the city's monthly rent per m² stats and percentile fan and compare the model rent per m² to them. Figures span the full listing history, 2% trimmed.

5. THE DOCUMENT
   Cover:
   - Gross yield: <yield_pct_lower>% – <yield_pct_upper>% (from the estimate)
   - Model rent: CHF <rent>/month vs current asking rents per m² in the commune
   - Verdict: is the implied yield above or below the local asking-rent picture?
   Notes-line material: asking rents ≠ realised rents; the yield is GROSS — it ignores charges, vacancy, maintenance, and taxes; this is an AI estimate, not investment advice or a formal appraisal.

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
