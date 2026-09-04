<!-- GENERATED FROM api/popety-mcp/src/prompts.ts — do not edit; run pnpm --filter @popety-io/mcp generate:integrations -->

# Popety.io Claude plugin

Swiss real-estate intelligence for Claude — 9 guided workflows (parcel reports,
valuations, market analysis, development sourcing) driving the official Popety.io
connector. Installing the plugin bundles the connector and the workflow commands.

This repository is a generated mirror of `integrations/claude-plugin/` in the
Popety.io monorepo — do not edit here; changes are overwritten on the next publish.

## Install

### Claude Desktop

1. Settings → plugins: **Add marketplace** → **Add from a repository**.
2. Enter `https://github.com/Popety/popety-io-claude-plugin`.
3. Install the **Popety.io** plugin from the marketplace.

### Claude Code

```
/plugin marketplace add Popety/popety-io-claude-plugin
/plugin install popety@popety-marketplace
```

## Authentication

The Popety connector authenticates on first use: your client prompts you to connect
via OAuth or a Popety API key (manage keys at https://developers.popety.io).

## Commands

| Command | What it does |
| --- | --- |
| `/popety:feasibility-snapshot` | Assess buildability, zoning and constraints of a Swiss parcel from an address. |
| `/popety:market-overview` | Asking-price levels and transaction trends for a Swiss commune. |
| `/popety:parcel-report` | Full dossier for a parcel: zoning, restrictions, buildings, history and context. |
| `/popety:value-estimate` | AVM valuation with user-confirmed attributes, cross-checked against the market. |
| `/popety:find-development-sites` | Hunt under-exploited parcels with development potential in a commune or canton. |
| `/popety:neighborhood-profile` | Demographics, taxes, transport, noise and hazards around an address. |
| `/popety:development-activity` | Construction permits, transactions and new-build pulse of a commune. |
| `/popety:investment-yield` | Gross-yield check: AVM purchase and rent estimates vs current asking rents. |
| `/popety:listing-hunter` | Find active listings matching a client brief and build a shortlist. |

Each command follows cost gates: free/preview steps run before any paid call, and
valuations always confirm property attributes with you first.
