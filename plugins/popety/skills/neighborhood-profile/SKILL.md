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

3. STRUCTURED PROFILE
   Present one section per theme:
   - People: population, median income, average household size (demographics)
   - Economy: employment figures (employment)
   - Taxes: communal tax multiplier (taxes)
   - Getting around: transit score and walk score (transport, accessibility)
   - Quiet or noisy: road/rail noise in dB (noise)
   - Risks: flood/landslide/avalanche hazard levels (hazards)
   - Running costs: electricity tariff (electricity)
   Close with a short livability / investment-context verdict grounded ONLY in the returned data — flag any topic that came back empty rather than guessing.

---

Requires the Popety connector (bundled with this plugin — authenticate on first use). Follow the workflow's cost gates: free/preview steps before any paid call.
