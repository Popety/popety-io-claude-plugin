---
name: feasibility-snapshot
description: "Assess buildability, zoning and constraints of a Swiss parcel from an address. Use when the user asks about buildability, what can be built on a parcel, zoning check, building constraints, feasibility of a plot."
disable-model-invocation: false
---

<!-- GENERATED FROM api/popety-mcp/src/prompts.ts — do not edit; run pnpm --filter @popety-io/mcp generate:integrations -->

# Feasibility snapshot

Assess buildability, zoning and constraints of a Swiss parcel from an address.

**Arguments to collect from the user:**

- `address` — Full Swiss address including street, number, and municipality. (required)

## Workflow

Assess the buildability and planning constraints of the Swiss parcel at "<address>". Follow these steps exactly:

1. LOCATE THE PARCEL
   Call find_land with { by: "address", value: "<address>" }.
   Extract the returned land_id (popetyio_land_id).

2. LOAD LAND DETAILS
   Call get_land with { land_id: <land_id>, include: ["zoning", "restrictions"] }.
   Note: zone name (main_lupa_name), floor-area ratio (ius_authorized), max floors, RDPPF restrictions (forest, flood, archaeological, danger zones), and any servitudes.

3. CONTEXTUAL HAZARDS AND ACCESS
   Call get_context_at with { target: { land_id: <land_id> }, topics: ["hazards", "noise", "transport"] }.
   Note: flood/avalanche/landslide risk levels, road/rail noise (dB), and transit score.

4. ZONING REGULATIONS
   Call search_regulations with { land_id: <land_id> } to retrieve applicable communal and cantonal planning rules for this zone.

5. THE DOCUMENT (blueprint — follow the kit)
   - Verdict: can it be developed, and how much — with the capacity MATH shown: authorized floor area (parcel area × IUS/IBUS) vs built today, and the headroom in m² (the land indices carry current-use ratios — use them)
   - 4 KPIs: zone + IUS · capacity headroom m² · under-exploited score (gauge) · hazard level
   - "Zoning & rules": zone card with max floors/height/dwellings and the cited regulation articles (document + article + page) — the citations ARE the value
   - "Constraints": RDPPF restrictions and hazard/noise levels as badges (good/warn/danger), each with its one-line implication
   - "Access & context": transit and noise in one strip
   Analyst paragraphs: what materially limits the potential and what to verify next (servitudes, PPE constitution, neighbour oppositions).

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
