---
name: gmgn-project-deep-report
description: 'Orchestrate a comprehensive GMGN project deep report for a token: fundamentals -> security -> liquidity -> smart money conviction -> price action -> scored verdict. Delegates CLI execution to gmgn-token and gmgn-market skills. Use when asked for a deep report, full project analysis, complete investment research, or whether a token is worth a large position. Requires: gmgn-cli, GMGN_API_KEY, and the referenced skills installed.'
argument-hint: "--address <token_address> [--chain <sol|bsc|base|eth>] [--kline-resolution <1m|5m|15m|1h|4h|1d>] [--trending-interval <1m|5m|1h|6h|24h>]"
---

# GMGN Project Deep Report

Orchestration skill. Delegates all CLI commands and field interpretation to the **gmgn-token** and **gmgn-market** skills — this file only defines the multi-step report workflow, scoring logic, and output format. Do not inline CLI syntax here; refer to the referenced skills for command details.

Use when: deep report, full project analysis, complete investment research, or whether a token is worth a meaningful position instead of just a quick speculative trade.

## Parameters

| Param | Default | Description |
|-------|---------|-------------|
| `--chain` | `sol` | `sol` / `bsc` / `base` / `eth` |
| `--address` | required | Token address to analyze |
| `--kline-resolution` | `4h` | Price-action context window |
| `--trending-interval` | `1h` | Momentum check window |

## Workflow

### Step 1 — Fundamentals

Use the **gmgn-token** skill's info view.

Assess:
- Current price and approximate market cap
- Liquidity depth relative to position size
- `holder_count` and holder growth quality
- `wallet_tags_stat.smart_wallets` and `wallet_tags_stat.renowned_wallets`
- Social presence across Twitter/X, Telegram, and website
- `cto_flag` and `creator_token_status`

Fundamental score (0-3):
- +1 if market cap looks reasonable for the chain and category
- +1 if social presence is strong
- +1 if smart wallet presence and holder quality both look credible

### Step 2 — Security Assessment

Use the **gmgn-token** skill's security view.

Hard stops:
- `is_honeypot = "yes"` on supported chains
- `rug_ratio > 0.5`
- On Solana, both `renounced_mint = false` and `renounced_freeze_account = false`
- `sell_tax > 0.15`

Security score (0-4):
- +1 if authority and contract posture look safe
- +1 if `rug_ratio < 0.1`
- +1 if `top_10_holder_rate < 0.3`
- +1 if sniper and wash-trading indicators look clean

### Step 3 — Liquidity and Pool Health

Use the **gmgn-token** skill's pool view.

Assess:
- Liquidity depth and likely exit slippage
- Pool age and market stability
- Whether the DEX is a major venue
- `is_on_curve` to distinguish post-graduation liquidity from bonding-curve risk

Liquidity score (0-2):
- +1 if liquidity comfortably exceeds thin-market territory
- +1 if pool age and venue quality look strong

### Step 4 — Smart Money Conviction

Use the **gmgn-token** skill's holders and traders views.

Assess:
- Distinct smart money wallet count
- Net buy versus net sell posture
- Whether unrealized profit suggests they are still holding conviction
- Whether KOL wallets are present with active positions
- Whether holder diversity is broad or dominated by a single whale

Smart money score (0-4):
- +1 if smart wallet count is meaningfully strong
- +1 if aggregate direction still looks like accumulation
- +1 if smart money unrealized posture is positive
- +1 if at least one KOL is actively involved

### Step 5 — Price Action Context

Use the **gmgn-market** skill's kline and trending views.

Assess:
- Whether the token is near a reasonable entry zone or already parabolic
- Whether bullish candles show healthier volume than bearish candles
- Whether the token is currently trending with smart money confirmation

Price action score (0-2):
- +1 if the setup does not look like late chasing
- +1 if momentum and volume structure look healthy

### Step 6 — Risk Summary and Verdict

Aggregate the five areas into one report.

Risk categories to summarize:
- Security
- Liquidity
- Smart Money
- Holder Quality
- Price Action

Verdict framework:
- 🟢 **Strong buy candidate** — score >= 11 and no hard stops
- 🟡 **Watchlist** — score 7-10 and no hard stops
- 🔴 **Skip** — any hard stop or score < 7

If a hard stop is triggered, stop optimizing the narrative and say so directly.

## Output Format

Before rendering the report:
- Always display the full on-chain token address whenever the token is referenced.
- Symbols or token names may be shown only as secondary context after the full address.
- Never shorten any address with `...` or any other ellipsis form.

```
═══════════════════════════════════════════
  PROJECT DEEP REPORT — {address}
  {chain} | Symbol: {symbol} | {date}
═══════════════════════════════════════════

FUNDAMENTALS
- Price: ${price}
- Market Cap: ~${market_cap}
- Liquidity: ${liquidity}
- Holders: {holder_count}
- Social: Twitter/X {yes/no}, Telegram {yes/no}, Website {yes/no}
- Dev Status: {creator_close / creator_hold / other}
- Score: {x}/3

SECURITY
- Honeypot: {yes/no}
- Rug Risk: {rug_ratio}
- Concentration: {top_10_holder_rate}
- Wash Trading: {yes/no}
- Score: {x}/4

LIQUIDITY
- Pool Venue: {dex}
- Pool Age: {pool_age}
- Curve Status: Graduated / Still on curve
- Score: {x}/2

SMART MONEY CONVICTION
- Smart Wallets: {n}
- Net Direction: Accumulating / Mixed / Distributing
- Unrealized Posture: Positive / Mixed / Weak
- KOL Presence: {n}
- Score: {x}/4

PRICE ACTION
- Trend: Healthy accumulation / Parabolic / Weak
- Trending Now: Yes / No
- Score: {x}/2

RISK FLAGS
- {list major warning signals or state none}

TOTAL SCORE
- {x}/15

VERDICT
- 🟢 Strong buy candidate / 🟡 Watchlist / 🔴 Skip
- Reason: {1-2 sentence summary}
═══════════════════════════════════════════
```

## Dependencies

This skill requires the following companion skills to be installed and eligible:
- **gmgn-token** — info, security, pool, holders, traders, and field knowledge
- **gmgn-market** — kline and trending context plus field knowledge

Optional downstream follow-up skills:
- **gmgn-swap** — execution after explicit user confirmation