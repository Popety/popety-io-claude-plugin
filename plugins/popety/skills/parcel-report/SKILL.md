---
name: parcel-report
description: "Full dossier for a parcel: zoning, restrictions, buildings, history and context. Use when the user asks about parcel dossier, land-registry extract, cadastre details, plot information, parcel summary."
disable-model-invocation: false
---

<!-- GENERATED FROM api/popety-mcp/src/prompts.ts — do not edit; run pnpm --filter @popety-io/mcp generate:integrations -->

# Parcel report

Full dossier for a parcel: zoning, restrictions, buildings, history and context.

**Arguments to collect from the user:**

- `address` — Full Swiss address. Supply at least one of address or land_id. (optional)
- `land_id` — Popety land identifier (popetyio_land_id). Supply at least one of address or land_id. (optional)

## Workflow

Generate a concise land-registry report for the Swiss parcel at "<address>". Follow these steps exactly:

1. RESOLVE PARCEL
   Call find_land with { by: "address", value: "<address>" }.
   Extract the returned land_id (popetyio_land_id).

2. LOAD ALL LAND DATA
   Call get_land with { land_id: "<land_id from step 1>", include: "all" }.
   This returns: cadastral area, zoning, owners, restrictions, recent permits, and recent transactions on this parcel.
   The response's _omitted_sections lists the large market annexes (purchase_market, rental_market) that are NOT included by default — mention their availability in the report and fetch one explicitly (include: ["purchase_market"]) only if the user asks for market depth.

3. REPORT
   Present a structured summary:
   - Identification: EGRID, parcel number (code_number), municipality
   - Area: cadastral area (m²)
   - Zoning: zone name, IUS, permitted uses
   - Owners: number of owners, type (private/public/institutional)
   - Restrictions: list RDPPF restrictions and servitudes
   - Recent permits: last 3 construction permits (type, date, status)
   - Recent transactions: last 3 transactions (type, date, price if disclosed)
   Keep the report concise — one section per bullet above.

4. OFFER NEXT STEPS
   Close with two concrete hand-offs: value this property (/popety:value-estimate) and market conditions for the commune (/popety:market-overview) — phrased as an offer, not run automatically.

---

Requires the Popety connector (bundled with this plugin — authenticate on first use). Follow the workflow's cost gates: free/preview steps before any paid call.
