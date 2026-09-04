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

5. SUMMARY
   Summarise buildability (zone type, IUS, max floors, permitted uses), key constraints (RDPPF restrictions, hazard levels, noise), access quality, and any regulatory specifics from the regulation excerpts. Flag anything that materially limits development potential.

---

Requires the Popety connector (bundled with this plugin — authenticate on first use). Follow the workflow's cost gates: free/preview steps before any paid call.
