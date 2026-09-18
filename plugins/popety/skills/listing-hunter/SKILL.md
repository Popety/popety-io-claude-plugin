---
name: listing-hunter
description: "Find active listings matching a client brief and build a shortlist. Use when the user asks about find listings, property search, homes or apartments for sale or rent, client shortlist."
disable-model-invocation: false
---

<!-- GENERATED FROM api/popety-mcp/src/prompts.ts — do not edit; run pnpm --filter @popety-io/mcp generate:integrations -->

# Listing hunter

Find active listings matching a client brief and build a shortlist.

**Arguments to collect from the user:**

- `location` — Commune (municipality) name, e.g. "Nyon". (required)
- `deal_type` — "sale" or "rent". Defaults to "sale". (optional)
- `budget_max_chf` — Maximum price in CHF (optional). (optional)
- `rooms_min` — Minimum number of rooms (optional). (optional)
- `property_type` — One of: apartment, house, commercial, land, parking (optional). (optional)

## Workflow

Shortlist current on-market sale listings in <municipality> for a client. Follow these steps exactly:

1. BUILD THE FILTERS
   Call search_entities with:
   { entity_type: "listings", filters: { area: { municipality: "<municipality>" },
     transaction_type: "sale", active: true } }
   ALWAYS keep active: true — the client wants current inventory, not the ~8M historical listings.

2. PREVIEW FIRST (free)
   Run the call above with preview: true. This search bills per returned object — if object_count is large, tighten the filters (budget, rooms_min, living_area_min_m2) BEFORE the paid call.

3. RUN THE FULL SEARCH
   Repeat the call without preview, passing the query_id from step 2, sort: "listing_timestamp_desc", and limit: 10.

4. THE DOCUMENT
   Structure the shortlist document as:
   - Title + a one-line recap of the brief (location, deal type, budget, rooms).
   - Shortlist: each listing with price, rooms, living area m², price per m², address, listing age (from listing_timestamp), and the url when present — plus bedrooms and parking (spaces/types) where present. Client briefs are often written in bedrooms: match against bedrooms when present but NEVER exclude listings missing it (coverage ~1 in 3; absence = unknown, not zero).
   - Standouts: the best fits and why, flagging listings priced above the commune's asking median — optionally cross-check with entity_stats { entity_type: "listings", filters: { municipality: "<municipality>", deal_type: "sale" }, section: "price_per_room" } (0.50 credits) and compare each listing to the median of its room band (section figures span the full listing history, not only active ads).
   - Next-step line: offer the neighborhood_profile prompt on a favourite or the value_estimate prompt to sanity-check an asking price — whichever fits the brief.
   Notes-line material: results are advertised listings — availability changes fast; verify with the lister before presenting to the client.

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
