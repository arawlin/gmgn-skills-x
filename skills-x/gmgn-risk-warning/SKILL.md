---
name: gmgn-risk-warning
description: 'Orchestrate a GMGN risk warning workflow for a held or watched token: security snapshot -> liquidity check -> whale holder analysis -> smart money flow -> price and volume anomaly check -> structured risk verdict. Delegates CLI execution to gmgn-token, gmgn-track, and gmgn-market skills. Use when asked whether whales are dumping, whether liquidity is still healthy, whether the developer may be exiting, or whether a position is becoming dangerous to hold. Requires: gmgn-cli, GMGN_API_KEY, and the referenced skills installed.'
argument-hint: "--address <token_address> [--chain <sol|bsc|base|eth>] [--kline-resolution <1m|5m|15m|1h|4h|1d>]"
---

# GMGN Risk Warning

Orchestration skill. Delegates all CLI commands and field interpretation to the **gmgn-token**, **gmgn-track**, and **gmgn-market** skills — this file only defines the multi-step monitoring workflow, warning thresholds, and output format. Do not inline CLI syntax here; refer to the referenced skills for command details.

Use when: risk warning, are whales dumping, is liquidity still healthy, are there signs the developer is exiting, or whether a held position is showing active danger signals.

## Parameters

| Param | Default | Description |
|-------|---------|-------------|
| `--chain` | `sol` | `sol` / `bsc` / `base` / `eth` |
| `--address` | required | Token address to assess |
| `--kline-resolution` | `1h` | Price and volume anomaly window |

## Workflow

### Step 1 — Security Snapshot

Use the **gmgn-token** skill's security view.

Immediate danger signals:
- `is_honeypot = "yes"`
- `rug_ratio > 0.3`
- `top_10_holder_rate > 0.5`
- `creator_token_status = creator_hold`
- On Solana, `renounced_mint = false` or `renounced_freeze_account = false`
- `sell_tax > 0.10`
- `bundler_rate > 0.3`
- `rat_trader_amount_rate > 0.3`
- `is_wash_trading = true`

If multiple red flags appear here, bias the final verdict toward immediate caution even before deeper checks.

### Step 2 — Liquidity Check

Use the **gmgn-token** skill's pool view.

Assess:
- Current USD liquidity and likely exit slippage
- Whether liquidity looks materially weaker than the user's prior baseline, if one exists
- Pool age and whether the liquidity venue looks credible
- Whether the token is still on a bonding curve versus already graduated to a DEX

### Step 3 — Whale Holder Analysis

Use the **gmgn-token** skill's holders view for both overall holders and smart money holders.

Warning signals:
- Top 1-3 wallets hold too much of supply
- Smart money holder count is absent or shrinking
- Top holders include `bundler` or `rat_trader` style concentration

### Step 4 — Smart Money Flow

Use the **gmgn-track** skill's smart money feed and focus only on the token under review.

Assess:
- Whether multiple tracked wallets are selling recently
- Whether clustered exits appear at the same time
- Whether recent smart money entries are already underwater, which can precede forced exits

### Step 5 — Price and Volume Anomaly

Use the **gmgn-market** skill's kline view.

Look for:
- Volume spikes without price progress
- Price drops on expanding volume
- Volume collapse that closes liquidity windows
- Consecutive red candles after a local top or all-time high

### Step 6 — Overall Verdict

Summarize across five areas:
- Security
- Liquidity
- Whale Concentration
- Smart Money Flow
- Price Action

Verdict framework:
- 🟢 **No active risk signals** — position appears stable
- 🟡 **Watch closely** — 1-2 meaningful warning signals present
- 🔴 **High risk** — multiple danger signals or any critical exit-risk condition

If a honeypot or equivalent hard-stop condition appears, say so directly and do not soften the conclusion.

## Output Format

Before rendering the warning report:
- Always display the full on-chain token address whenever the token is referenced.
- Symbols or token names may be shown only as secondary context after the full address.
- Never shorten any address with `...` or any other ellipsis form.

```
═══════════════════════════════════════════
  RISK WARNING — {address} ({symbol})
  {chain} | {timestamp}
═══════════════════════════════════════════

SECURITY
- Honeypot: Yes / No
- Rug Ratio: {value}
- Dev Status: {creator_hold / creator_close / other}
- Verdict: ✅ / ⚠️ / 🚫

LIQUIDITY
- Current Liquidity: ${value}
- Pool Age: {value}
- Venue Quality: Strong / Mixed / Weak
- Verdict: ✅ / ⚠️ / 🚫

WHALE CONCENTRATION
- Top 10 Hold Rate: {value}
- Smart Money Holders: {n}
- Verdict: ✅ / ⚠️ / 🚫

SMART MONEY FLOW
- Recent Direction: Buying / Mixed / Selling
- Clustered Exits: Yes / No
- Verdict: ✅ / ⚠️ / 🚫

PRICE ACTION
- Volume Trend: Normal / Selling Pressure / Collapsing
- Candle Structure: Accumulation / Neutral / Distribution
- Verdict: ✅ / ⚠️ / 🚫

OVERALL VERDICT
- 🟢 No active risk signals / 🟡 Watch closely / 🔴 High risk
- Reason: {1-2 sentence summary}
═══════════════════════════════════════════
```

## Dependencies

This skill requires the following companion skills to be installed and eligible:
- **gmgn-token** — security, pool, holders, and field knowledge
- **gmgn-track** — smart money flow monitoring and direction signals
- **gmgn-market** — kline-based volume and price anomaly context

Install via `openclaw skills install` or symlink them from the gmgn-skills repo.