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

3. THE DOCUMENT
   Cover:
   - Identification: EGRID, parcel number (code_number), municipality
   - Area: cadastral area (m²)
   - Zoning: zone name, IUS, permitted uses
   - Owners: number of owners, type (private/public/institutional)
   - Restrictions: list RDPPF restrictions and servitudes
   - Recent permits: last 3 construction permits (type, date, status)
   - Recent transactions: last 3 transactions (type, date, price if disclosed)

4. NEXT STEP
   Next-step candidates: value this property (/popety:value-estimate) or market conditions for the commune (/popety:market-overview) — offer the more relevant one, never run it automatically.

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
