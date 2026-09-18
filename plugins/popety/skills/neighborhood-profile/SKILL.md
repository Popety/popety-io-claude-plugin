---
name: neighborhood-profile
description: "Demographics, taxes, transport, noise and hazards around an address. Use when the user asks about neighborhood quality, livability, taxes, noise, transport, natural hazards around an address."
disable-model-invocation: false
---

<!-- GENERATED FROM api/popety-mcp/src/prompts.ts — do not edit; run pnpm --filter @popety-io/mcp generate:integrations -->

# Neighborhood profile

Demographics, taxes, transport, noise and hazards around an address.

**Arguments to collect from the user:**

- `address` — Full Swiss address including street, number, and municipality. (required)

## Workflow

Build a livability and investment-context profile for the neighborhood around "<address>". Follow these steps exactly:

1. LOCATE THE PARCEL
   Call find_land with { by: "address", value: "<address>" }.
   Extract the returned land_id (popetyio_land_id).

2. LOAD ALL CONTEXT TOPICS (one call, 1 credit)
   Call get_context_at with:
   { target: { land_id: <land_id> }, topics: ["demographics", "employment", "taxes",
     "transport", "noise", "hazards", "electricity", "accessibility"] }

3. WHAT IS ACTUALLY NEARBY (one call, 1 credit)
   Call pois_nearby with the parcel coordinates from step 1's response:
   { lat: <lat>, lon: <lon> }
   Summary mode returns nine scored amenity categories (schools, shops, transit, health, …) with the nearest examples.

4. THE DOCUMENT (blueprint — follow the kit)
   - Verdict + 4 KPIs: transit score · noise dB · communal tax multiplier · the strongest amenity score
   - Amenity categories as a scored bar list (pois_nearby), each with its nearest concrete example; hazards as badges (good/warn/danger)
   Themes to cover (group related ones into sections):
   - People: population, median income, average household size (demographics)
   - Economy: employment figures (employment)
   - Taxes: communal tax multiplier (taxes)
   - Getting around: transit score and walk score (transport, accessibility)
   - Quiet or noisy: road/rail noise in dB (noise)
   - Risks: flood/landslide/avalanche hazard levels (hazards)
   - Running costs: electricity tariff (electricity)
   - Daily life nearby: strongest and weakest amenity categories with the nearest concrete examples (pois_nearby)
   The verdict is the livability / investment-context call, grounded ONLY in the returned data. Notes-line material: any topic that came back empty — flag it rather than guessing.

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
