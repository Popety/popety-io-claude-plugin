---
name: find-development-sites
description: "Hunt under-exploited parcels with development potential in a commune or canton. Use when the user asks about development potential, under-exploited parcels, site sourcing, densification opportunities."
disable-model-invocation: false
---

<!-- GENERATED FROM api/popety-mcp/src/prompts.ts — do not edit; run pnpm --filter @popety-io/mcp generate:integrations -->

# Find development sites

Hunt under-exploited parcels with development potential in a commune or canton.

**Arguments to collect from the user:**

- `location` — Commune (municipality) name, e.g. "Pully", or a 2-letter canton code, e.g. "VD". (required)
- `min_area_m2` — Minimum parcel area in m² (optional). (optional)

## Workflow

Find under-exploited parcels with development potential in <commune or canton>. Follow these steps exactly:

1. PREVIEW THE SEARCH (free)
   Call search_entities with:
   { entity_type: "lands", filters: { municipality: "<commune or canton>", under_exploited_score_min: 60, no_building_zone: "hide" },
     sort: "under_exploited_score_desc", preview: true, limit: 10 }
   (no_building_zone: "hide" keeps only constructible parcels — never shortlist land in a no-building zone. Preview with the SAME limit you will commit with: object_count/cost are quoted at that limit, matching_total is the full count.)
   Note the object_count and cost — this search bills per returned object.

2. RUN THE FULL SEARCH
   Repeat the same call without preview, passing the query_id from step 1 and limit: 10.
   Extract per parcel: land_id, code_number, municipality, area, under_exploited_score, development_score.

3. CHECK ZONING HEADROOM FOR THE TOP 3
   For the 3 highest-scoring parcels, call get_land with { land_id: <land_id>, include: ["zoning"] }.
   Note: zone name (main_lupa_name), authorized IUS (ius_authorized) vs what is currently built — the gap is the densification headroom.

4. THE DOCUMENT
   Rank the shortlist; per parcel: code_number + land_id, commune, area m², under-exploited score, development score, zone + IUS, and one line on WHY it is under-exploited (e.g. low built density vs authorized IUS).
   Notes-line material: under-exploited and development scores are Popety proprietary indicators, not building permits. Next step: run the feasibility_snapshot prompt on the best candidate to check zoning rules, restrictions, and hazards before acting.

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
