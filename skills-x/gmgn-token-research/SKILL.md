---
name: gmgn-token-research
description: 'Orchestrate a full GMGN token research workflow for a token: basic info -> security -> liquidity pool -> market heat -> smart money signals -> structured buy, watch, or skip verdict. Delegates CLI execution to gmgn-token and gmgn-market skills. Use when asked whether a token is worth researching or buying, or when the user wants a full due diligence flow instead of a quick checklist. Requires: gmgn-cli, GMGN_API_KEY, and the referenced skills installed.'
argument-hint: "--address <token_address> [--chain <sol|bsc|base|eth>] [--trending-interval <1m|5m|1h|6h|24h>]"
metadata: {"openclaw": {"requires": {"bins": ["gmgn-cli"], "env": ["GMGN_API_KEY"]}, "primaryEnv": "GMGN_API_KEY"}}
---

# GMGN Token Research

Orchestration skill. Delegates all CLI commands and field interpretation to the **gmgn-token** and **gmgn-market** skills — this file only defines the full research workflow, decision thresholds, and output format. Do not inline CLI syntax here; refer to the referenced skills for command details.

Use when: full token research, token due diligence, should I buy this token, or when the user wants a structured buy/watch/skip answer instead of a quick pre-buy screen.

## Parameters

| Param | Default | Description |
|-------|---------|-------------|
| `--chain` | `sol` | `sol` / `bsc` / `base` / `eth` |
| `--address` | required | Token address to analyze |
| `--trending-interval` | `1h` | Current market-heat window |

## Workflow

### Step 1 — Basic Info

Use the **gmgn-token** skill's info view.

Check:
- `price` and approximate market cap
- `liquidity`
- `holder_count`
- `wallet_tags_stat.smart_wallets` and `wallet_tags_stat.renowned_wallets`
- Social presence from website, Twitter/X, and Telegram fields

Red flags:
- All social fields are empty
- Liquidity is below thin-market territory
- No smart money and no renowned wallets at all

### Step 2 — Security Check

Use the **gmgn-token** skill's security view.

Assess the workflow thresholds directly:
- `is_honeypot`
- `open_source`
- `owner_renounced`
- On Solana, `renounced_mint` and `renounced_freeze_account`
- `buy_tax` and `sell_tax`
- `top_10_holder_rate`
- `rug_ratio`
- `creator_token_status`
- `sniper_count`

Hard stops:
- `is_honeypot = "yes"` on supported chains
- Clearly unsafe authority posture
- Very high rug, tax, or concentration risk

If a hard stop is triggered, keep that as the dominant conclusion.

### Step 3 — Liquidity Pool

Use the **gmgn-token** skill's pool view.

Assess:
- Liquidity depth and likely slippage
- DEX quality
- Pool age

Low liquidity should count as a real execution risk even if the token looks attractive elsewhere.

### Step 4 — Market Heat

Use the **gmgn-market** skill's trending view.

Check:
- Whether the token is currently trending
- Trending rank if present
- Volume, `smart_degen_count`, and short-window price change

Interpretation:
- Found in trending with active smart money support is a positive confirmation
- Not trending is neutral, not an automatic rejection

### Step 5 — Smart Money Signals

Use the **gmgn-token** skill's holders and traders views.

Assess:
- Whether smart money wallets are accumulating or distributing
- Whether unrealized posture suggests conviction is still open
- Whether KOL wallets are active holders or already exiting
- Whether large holders are too concentrated

Bullish signs:
- Smart money buy pressure remains stronger than sell pressure
- Unrealized gains are still meaningful
- KOL participation exists without obvious exit flow

Bearish signs:
- Smart money is distributing
- Large realized exits dominate the recent posture
- Supply concentration is high and unstable

### Step 6 — Verdict

Present one structured decision:
- 🟢 **Buy** — mostly healthy signals and no hard stops
- 🟡 **Watch** — mixed picture, no hard stop, but more confirmation needed
- 🔴 **Skip** — hard stop or too many meaningful warnings

Decision logic:
- Any major red flag or hard stop should move the result to **Skip**
- Multiple warning signals without a hard stop should usually land on **Watch**
- A **Buy** verdict needs healthy security, workable liquidity, and supportive smart money posture

## Output Format

```
═══════════════════════════════════════════
  TOKEN RESEARCH SUMMARY — {symbol}
  {chain} | {short_address}
═══════════════════════════════════════════

SECURITY
- Honeypot: No / Yes
- Contract Posture: Safe / Mixed / Unsafe
- Rug Risk: Low / Medium / High
- Top-10 Concentration: Low / Medium / High

LIQUIDITY
- Pool Liquidity: ${liquidity}
- Venue: {exchange}
- Slippage Risk: Low / Medium / High

MARKET HEAT
- Trending: Yes (rank #{rank}) / Not trending
- Smart Degen Count: {x}
- Short-Window Momentum: Strong / Mixed / Weak

SMART MONEY
- Smart Wallets: {x}
- KOL Wallets: {x}
- Smart Money Activity: Accumulating / Mixed / Distributing / Absent

VERDICT
- 🟢 Buy / 🟡 Watch / 🔴 Skip
- Reason: {1-2 sentence summary}
═══════════════════════════════════════════
```

## Dependencies

This skill requires the following companion skills to be installed and eligible:
- **gmgn-token** — info, security, pool, holders, traders, and field knowledge
- **gmgn-market** — trending context and field knowledge

Optional downstream follow-up skills:
- **gmgn-project-deep-report** — deeper multi-factor report for larger-position decisions
- **gmgn-swap** — execution after explicit user confirmation

Install via `openclaw skills install` or symlink them from the gmgn-skills repo.