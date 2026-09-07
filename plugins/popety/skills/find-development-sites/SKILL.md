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
     sort: "under_exploited_score_desc", preview: true }
   (no_building_zone: "hide" keeps only constructible parcels — never shortlist land in a no-building zone.)
   Note the object_count and cost — this search bills per returned object.

2. RUN THE FULL SEARCH
   Repeat the same call without preview, passing the query_id from step 1 and limit: 10.
   Extract per parcel: land_id, code_number, municipality, area, under_exploited_score, development_score.

3. CHECK ZONING HEADROOM FOR THE TOP 3
   For the 3 highest-scoring parcels, call get_land with { land_id: <land_id>, include: ["zoning"] }.
   Note: zone name (main_lupa_name), authorized IUS (ius_authorized) vs what is currently built — the gap is the densification headroom.

4. RANKED SHORTLIST
   Present a ranked table: parcel (code_number + land_id), commune, area m², under-exploited score, development score, zone + IUS, and one line on WHY it is under-exploited (e.g. low built density vs authorized IUS).

   CAVEAT: under-exploited and development scores are Popety proprietary indicators, not building permits. Before acting on a site, run the feasibility_snapshot prompt to check zoning rules, restrictions, and hazards.

---

Requires the Popety connector (bundled with this plugin — authenticate on first use). Follow the workflow's cost gates: free/preview steps before any paid call.
