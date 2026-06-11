---
name: gmgn-market-opportunities
description: 'Orchestrate the canonical GMGN market opportunity discovery workflow from trending data: fetch a broad safe-filtered pool -> rank tokens with multi-factor analysis -> present top picks with rationale -> suggest deep dive or swap follow-ups. Delegates CLI execution to the gmgn-market skill. Use when asked what tokens are worth watching from trending data, which hot tokens have the best composite profile, or to discover opportunities from Solana, BSC, Base, or Ethereum trending feeds. Requires: gmgn-cli, GMGN_API_KEY, and gmgn-market installed.'
argument-hint: "[--chain <sol|bsc|base|eth>]"
---

# GMGN Market Opportunities

Orchestration skill. Delegates all CLI commands and field interpretation to the **gmgn-market** skill — this file only defines the multi-step ranking workflow, analysis logic, and output format. Do not inline CLI syntax here; refer to the referenced skill for command details.

Use when: market opportunities, what tokens are worth watching, top trending picks, hot tokens with smart money interest, or broad opportunity discovery from trending data.

## Parameters

| Param | Default | Description |
|-------|---------|-------------|
| `--chain` | `sol` | `sol` / `bsc` / `base` / `eth` |

## Workflow

### Step 1 — Fetch Trending Data

Use the **gmgn-market** skill to fetch a broad trending pool: interval `1h`, ordered by volume, limit 50, with `not_honeypot` and `has_social` filters applied.

### Step 2 — Multi-Factor Analysis

Analyze each record using the following signals (apply judgment, not rigid rules):

| Signal | Field(s) | Weight | Notes |
|--------|----------|--------|-------|
| Smart money interest | `smart_degen_count`, `renowned_count` | High | Key conviction indicator |
| Bluechip ownership | `bluechip_owner_percentage` | Medium | Quality of holder base |
| Real trading activity | `volume`, `swaps` | Medium | Distinguishes genuine interest from wash trading |
| Price momentum | `change1h`, `change5m` | Medium | Prefer positive, non-parabolic moves |
| Pool safety | `liquidity` | Medium | Low liquidity = high slippage risk |
| Token maturity | `creation_timestamp` | Low | Avoid tokens less than ~1h old unless other signals are very strong |

Select the **top 5** tokens with the best composite profile. Prefer tokens that perform well across multiple signals rather than excelling in just one.

### Step 3 — Present Top 5 to User

Present results as a concise table, then give a one-line rationale for each pick. The output MUST include the input parameters used and full token addresses for every token mentioned.

```
Top 5 Trending Tokens — {chain} / 1h

📥 INPUT PARAMETERS
  Chain:        {chain}
  Interval:     1h
  Pool size:    {pool_size}

# | Symbol | Address (short) | Smart Degens | Volume | 1h Chg | Reasoning
1 | ...     | ...             | ...          | ...    | ...    | Smart money accumulating + high volume
2 | ...
...

─── FULL ADDRESSES ─────────────────────────
  TOKEN_1 ({SYMBOL}):  {full_token_address}
  TOKEN_2 ({SYMBOL}):  {full_token_address}
  ...
```

## Follow-Up Actions

For each token, offer:
- **Deep dive**: run full token research — use **gmgn-token-research** skill
- **Swap**: execute directly if the user is satisfied with the trending data alone — use **gmgn-swap** skill

## Dependencies

This skill requires the following companion skill to be installed and eligible:
- **gmgn-market** — trending command behavior plus field knowledge