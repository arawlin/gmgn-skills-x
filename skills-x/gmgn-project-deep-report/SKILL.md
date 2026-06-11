---
name: gmgn-project-deep-report
description: 'Orchestrate the canonical GMGN project deep report for a token: fundamentals -> security -> liquidity -> smart money conviction -> price action -> scored verdict. Delegates CLI execution to gmgn-token and gmgn-market skills. Use when asked for a deep report, full project analysis, complete investment research, or whether a token is worth a large position. Requires: gmgn-cli, GMGN_API_KEY, and the referenced skills installed.'
argument-hint: "--address <token_address> [--chain <sol|bsc|base|eth>]"
---

# GMGN Project Deep Report

Orchestration skill. Delegates all CLI commands and field interpretation to the **gmgn-token** and **gmgn-market** skills — this file only defines the multi-step report workflow, scoring logic, and output format. Do not inline CLI syntax here; refer to the referenced skills for command details.

Use when: deep report, full project analysis, complete investment research, or whether a token is worth a meaningful position instead of just a quick speculative trade.

> For a quick pre-buy check, use **gmgn-token-research** skill instead. This workflow is more comprehensive and produces a full written report.

## Parameters

| Param | Default | Description |
|-------|---------|-------------|
| `--chain` | `sol` | `sol` / `bsc` / `base` / `eth` |
| `--address` | required | Token address to analyze |

## Workflow

### Step 1 — Fundamentals

Use the **gmgn-token** skill's info view.

Extract and assess:

| Field | What to Note |
|-------|-------------|
| `price` | Current price |
| Market cap | `price × circulating_supply` — compute this manually |
| `liquidity` | USD in pool — < $50k is thin for a "serious" position |
| `holder_count` | Total wallets holding. Growing = organic adoption |
| `wallet_tags_stat.smart_wallets` | Smart money holders count |
| `wallet_tags_stat.renowned_wallets` | KOL holders count |
| `link.*` | Social presence: Twitter, Telegram, website |
| `cto_flag` | Community takeover? |
| `creator_token_status` | Dev still holding or has sold? |

**Fundamental score (0–3):**
- +1 if market cap reasonable for the chain/category
- +1 if strong social presence (2+ active channels)
- +1 if `smart_wallets` ≥ 3 AND `holder_count` growing

### Step 2 — Security Assessment

Use the **gmgn-token** skill's security view.

**Hard stops (any one = do not proceed):**
- `is_honeypot = "yes"` (BSC/Base)
- `rug_ratio > 0.5`
- `renounced_mint = false` AND `renounced_freeze_account = false` (SOL) — both unrenounced
- `sell_tax > 0.15`

**Security score (0–4):**
- +1 if contract open source / renounced
- +1 if `rug_ratio < 0.1`
- +1 if `top_10_holder_rate < 0.3`
- +1 if no snipers (`sniper_count < 5`) and no wash trading

### Step 3 — Liquidity and Pool Health

Use the **gmgn-token** skill's pool view.

Assess:
- **Liquidity depth:** > $100k = healthy; $10k–$100k = thin; < $10k = high exit slippage
- **Pool age:** older pool = more stable price history
- **DEX:** recognized exchange (Raydium, Uniswap v3, PancakeSwap) = better
- **Bonding curve status** (`is_on_curve`): if still on curve, token has not graduated — higher volatility window

**Liquidity score (0–2):**
- +1 if liquidity > $50k
- +1 if DEX is major and pool age > 24h

### Step 4 — Smart Money Conviction Analysis

This is the key differentiator from basic token research.

Use the **gmgn-token** skill's holders and traders views:
- Smart money holders: filter to `smart_degen`, ordered by `buy_volume_cur` descending, top 20
- KOL holders: filter to `renowned`, ordered by `profit` descending, top 10
- Top holders overall: ordered by `amount_percentage` descending, top 20 — check concentration

Assess smart money conviction:

| Signal | Bullish | Bearish |
|--------|---------|---------|
| Smart money count | ≥ 3 distinct wallets | 0 or 1 |
| Net direction | `buy_volume_cur` > `sell_volume_cur` | Selling exceeds buying |
| Unrealized profit | Large (still holding, not selling) | Small or negative |
| Realized profit | Moderate (some took profit, healthy) | Very large (majority already exited) |
| KOL involvement | ≥ 1 KOL with active position | None |
| Wallet diversity | Multiple different wallets | One whale dominating |

**Smart money score (0–4):**
- +1 if `smart_wallets` ≥ 3
- +1 if net buy direction (`buy_volume_cur > sell_volume_cur` across smart wallets)
- +1 if average `unrealized_profit` is positive (they're still in profit, still holding)
- +1 if at least 1 KOL has an active position

### Step 5 — Price Action Context

Use the **gmgn-market** skill:
- Kline view at `4h` resolution for recent price action
- Trending view at `1h` interval to check current momentum

Look for:
- **Entry context:** Is price near a local bottom (potential value) or after a run-up (chasing)?
- **Volume confirmation:** Do bullish candles have higher volume than bearish candles?
- **Trending:** If it appears in trending with `smart_degen_count > 0`, momentum + conviction overlap

**Price action score (0–2):**
- +1 if price is not parabolic (< 5x from recent low) — not chasing
- +1 if volume is rising on up-candles (healthy accumulation pattern)

### Step 6 — Risk Factors Summary

Aggregate all warning signals from Steps 1–5:

| Category | Risk Level | Key Signals |
|----------|-----------|-------------|
| Security | ✅/⚠️/🚫 | honeypot, rug_ratio, concentration |
| Liquidity | ✅/⚠️/🚫 | pool size, pool age |
| Smart Money | ✅/⚠️/🚫 | count, direction, conviction |
| Holder Quality | ✅/⚠️/🚫 | bundler_rate, rat_trader_rate, wash_trading |
| Price Action | ✅/⚠️/🚫 | entry timing, momentum |

## Verdict

- 🟢 **Strong buy candidate** — score ≥ 11 and no hard stops
- 🟡 **Watchlist** — score 7–10 and no hard stops
- 🔴 **Skip** — any hard stop OR score < 7

## Deep Report Output

The output MUST include the input parameters used and full token addresses for every token mentioned.

```
╔══════════════════════════════════════════════════════╗
║        PROJECT DEEP REPORT — {SYMBOL}                ║
║        {chain} | {date}                              ║
╚══════════════════════════════════════════════════════╝

📥 INPUT PARAMETERS
  Chain:             {chain}
  Token address:     {full_token_address}
  Kline resolution:  {kline_resolution}
  Trending interval: {trending_interval}

📋 FUNDAMENTALS
  Price:          ${price}
  Market Cap:     ~${market_cap}
  Liquidity:      ${liquidity} on {exchange}
  Holders:        {holder_count}
  Social:         Twitter ✅/❌ | Telegram ✅/❌ | Website ✅/❌
  Dev Status:     {creator_close = sold ✅ / creator_hold = still in ⚠️}
  Fundamental Score: {X}/3

🔒 SECURITY
  Honeypot:       ✅ No / 🚫 YES
  Contract:       {open_source} | {renounced}
  Rug Risk:       {rug_ratio} → ✅/⚠️/🚫
  Concentration:  Top-10 hold {top_10_holder_rate%} → ✅/⚠️/🚫
  Wash Trading:   ✅ None / ⚠️ Detected
  Security Score: {X}/4

💧 LIQUIDITY
  Pool:           ${liquidity} | {exchange} | Age: {pool_age}
  Bonding Curve:  Graduated ✅ / Still on curve ⚠️
  Liquidity Score: {X}/2

🧠 SMART MONEY CONVICTION
  SM Holders:     {N} wallets
  Net Direction:  Accumulating ✅ / Distributing ⚠️ / Mixed
  SM Unrealized:  +{X}% avg (still holding) ✅ / Underwater ⚠️
  KOL Presence:   {N} KOL wallets active
  Smart Money Score: {X}/4

📈 PRICE ACTION
  Recent trend:   Healthy accumulation / Parabolic (avoid chasing) / Declining
  Trending now:   Yes (rank #{rank}) ✅ / Not trending
  Price Action Score: {X}/2

─── RISK FLAGS ──────────────────────────────────────
  {List any ⚠️ or 🚫 signals here, or "No major risk flags"}

─── TOTAL SCORE ─────────────────────────────────────
  {X} / 15

─── VERDICT ─────────────────────────────────────────
  🟢 STRONG BUY CANDIDATE (score ≥ 11, no hard stops)
     Smart money confirmed, clean security, healthy liquidity
     → Suggested: research position sizing, use gmgn-swap

  🟡 WATCHLIST (score 7–10, no hard stops)
     Some positive signals but missing key conviction indicators
     → Suggested: monitor for 24–48h, re-assess if SM increases

  🔴 SKIP (any hard stop OR score < 7)
     Risk factors outweigh opportunity
     → Reason: {specific flag}

─── FULL ADDRESSES ─────────────────────────
  Token:            {full_token_address}
  {If SM/KOL wallets mentioned, list them here}
╚══════════════════════════════════════════════════════╝
```

## Follow-Up Actions

- Quick pre-buy check instead: use **gmgn-token-research** skill
- Ongoing risk monitoring after entering a position: use **gmgn-risk-warning** skill
- Deep dive on specific smart money wallets holding this token: use **gmgn-smart-money-profile** skill
- Find more tokens to run this report on: use **gmgn-market-opportunities** skill

## Dependencies

This skill requires the following companion skills to be installed and eligible:
- **gmgn-token** — info, security, pool, holders, traders, and field knowledge
- **gmgn-market** — kline and trending context plus field knowledge