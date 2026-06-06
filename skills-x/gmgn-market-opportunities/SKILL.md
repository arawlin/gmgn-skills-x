---
name: gmgn-market-opportunities
description: 'Orchestrate a GMGN market opportunity discovery workflow from trending data: fetch a broad safe-filtered pool -> rank tokens with multi-factor analysis -> present top picks with rationale -> suggest deep dive or swap follow-ups. Delegates CLI execution to the gmgn-market skill. Use when asked what tokens are worth watching from trending data, which hot tokens have the best composite profile, or to discover opportunities from Solana, BSC, Base, or Ethereum trending feeds. Requires: gmgn-cli, GMGN_API_KEY, and gmgn-market installed.'
argument-hint: "[--chain <sol|bsc|base|eth>] [--interval <1m|5m|1h|6h|24h>] [--pool-size <number>] [--top <number>]"
metadata: {"openclaw": {"requires": {"bins": ["gmgn-cli"], "env": ["GMGN_API_KEY"]}, "primaryEnv": "GMGN_API_KEY"}}
---

# GMGN Market Opportunities

Orchestration skill. Delegates all CLI commands and field interpretation to the **gmgn-market** skill — this file only defines the multi-step ranking workflow, analysis logic, and output format. Do not inline CLI syntax here; refer to the referenced skill for command details.

Use when: market opportunities, what tokens are worth watching, top trending picks, hot tokens with smart money interest, or broad opportunity discovery from trending data.

## Parameters

| Param | Default | Description |
|-------|---------|-------------|
| `--chain` | `sol` | `sol` / `bsc` / `base` / `eth` |
| `--interval` | `1h` | Trending lookback window |
| `--pool-size` | `50` | Number of trending records to screen |
| `--top` | `5` | Number of final picks to present |

## Workflow

### Step 1 — Fetch Trending Pool

Use the **gmgn-market** skill to fetch a broad trending pool with safe baseline filters.

Prefer a large enough candidate set to compare multiple signals rather than cherry-picking a few top-volume tokens too early.

### Step 2 — Multi-Factor Ranking

Evaluate each candidate using judgment, not rigid cutoffs.

High-priority signals:
- `smart_degen_count` and `renowned_count` for conviction
- `volume` and `swaps` for real trading activity

Medium-priority signals:
- `bluechip_owner_percentage` for holder quality
- `change1h` and `change5m` for momentum quality
- `liquidity` for slippage and pool safety

Lower-priority context:
- `creation_timestamp` to avoid extremely fresh tokens unless the broader profile is very strong

Prefer tokens that score well across multiple dimensions over tokens that look exceptional in only one metric.

### Step 3 — Select Top Picks

Rank the pool and keep the top `--top` candidates.

Bias toward:
- Smart money confirmation plus real volume
- Positive but non-parabolic momentum
- Adequate liquidity
- Reasonable token maturity for the chosen interval

De-prioritize or exclude:
- Suspiciously thin liquidity
- Activity that looks dominated by noise rather than conviction
- Extremely new tokens with weak supporting signals

### Step 4 — Present Rationale

For each selected token, provide a one-line reason that explains why it outranked the rest of the pool.

The rationale should combine at least two signals, for example smart money plus volume, or momentum plus liquidity quality.

### Step 5 — Follow-Up Paths

For any surfaced token:
- Offer a deeper handoff to the token-research workflow when the user wants due diligence
- Offer a swap handoff only if the user explicitly wants to trade based on the trending screen alone

## Output Format

```
═══════════════════════════════════════════
  MARKET OPPORTUNITIES — {chain} / {interval}
═══════════════════════════════════════════

Screened: {pool_size} trending tokens
Selected: {top} composite picks

# | Symbol | Address | Smart Degens | Volume | {interval} Chg | Rationale
1 | ...    | ...     | ...          | ...    | ...            | Smart money accumulating + strong real volume
2 | ...    | ...     | ...          | ...    | ...            | ...

─── NEXT ACTIONS ─────────────────────────
- Deep dive: run token research on the strongest pick
- Trade: only if the user accepts trending-only conviction
═══════════════════════════════════════════
```

## Dependencies

This skill requires the following companion skill to be installed and eligible:
- **gmgn-market** — trending command behavior plus field knowledge

Optional downstream follow-up skills:
- **gmgn-token** — deeper token research after discovery
- **gmgn-swap** — trade execution after explicit user confirmation

Install via `openclaw skills install` or symlink them from the gmgn-skills repo.