---
name: gmgn-smart-money-profile
description: 'Orchestrate the canonical GMGN smart money profile workflow for a wallet: compare 7d and 30d stats -> infer trading style from activity -> estimate take-profit and stop-loss behavior -> approximate copy-trade ROI -> optionally rank multiple wallets in a leaderboard. Delegates CLI execution to gmgn-portfolio and gmgn-track skills. Use when asked what a wallet''s trading style is, when it takes profit or cuts losses, whether copying it would have worked, or which smart money wallets are best to follow. Requires: gmgn-cli, GMGN_API_KEY, and the referenced skills installed.'
argument-hint: "--wallet <wallet_address> [--chain <sol|bsc|base|eth>]"
---

# GMGN Smart Money Profile

Orchestration skill. Delegates all CLI commands and field interpretation to the **gmgn-portfolio** and **gmgn-track** skills — this file only defines the behavior-analysis workflow, copy-trade estimation logic, and output format. Do not inline CLI syntax here; refer to the referenced skills for command details.

Use when: smart money profile, what is this wallet's trading style, when does it take profit, when does it cut losses, if I copied this wallet what would my return be, or which smart money wallets are most worth following.

> For basic "is this wallet worth following" analysis, use **gmgn-wallet-analysis** skill instead. This workflow goes deeper into behavior patterns and copy-trade estimation.

## Parameters

| Param | Default | Description |
|-------|---------|-------------|
| `--chain` | `sol` | `sol` / `bsc` / `base` / `eth` |
| `--wallet` | required | Wallet address to profile |

## Workflow

### Step 1 — Trading Stats (Both Periods)

Use the **gmgn-portfolio** skill to fetch stats for both `7d` and `30d` to detect performance trends.

Key metrics:

| Field | Meaning | Threshold |
|-------|---------|-----------|
| `winrate` | % of profitable trades (0–1) | > 0.6 strong, > 0.5 acceptable |
| `pnl` | realized_profit / total_cost multiplier | > 1.0 = net positive |
| `realized_profit` | USD profit locked in | context-dependent |
| `buy_count` / `sell_count` | trading frequency | high = active trader |
| `token_num` | number of distinct tokens traded | high = diversified |

**Trend signal:** If 7d `winrate` is significantly higher than 30d, performance is improving. If lower, recent form is declining.

### Step 2 — Activity Analysis (Style Inference)

Use the **gmgn-portfolio** skill's activity view, limit 100.

For each token that appears in both a buy and a sell event, compute holding duration:
- `sell.timestamp - buy.timestamp` in hours

**Style classification:**

| Holding Duration | Style Label |
|-----------------|-------------|
| < 1 hour | Scalper |
| 1h – 24h | Day trader |
| 1d – 7d | Swing trader |
| > 7d | Position / long-term holder |

Also check:
- **Position sizing consistency** — are buy amounts roughly similar (disciplined) or highly variable?
- **Token concentration** — does the wallet repeatedly trade the same tokens (specialist) or always new ones (trend chaser)?
- **Sell behavior** — do sells follow a pattern (e.g., always sells after 2–3x, or cuts at -30%)?

### Step 3 — Take-Profit and Stop-Loss Pattern

From activity data, cross-reference buy price vs sell price for completed round trips:

- For each token: find a `buy` event followed by a `sell` event
- Compute approximate return: `(sell_total_usd - buy_total_usd) / buy_total_usd`
- Group outcomes: wins vs losses

Look for:
- **Typical gain at exit** — does the wallet consistently take profit at ~2x, ~5x, or higher?
- **Typical loss at cut** — does the wallet cut quickly at -20% or hold through large drawdowns?
- **Asymmetry** — wins larger than losses = positive expected value. Reverse = risk.

### Step 4 — Copy-Trade ROI Estimation (Approximate)

> **Note:** This is an approximation based on historical activity data, not a precise backtest.

For the wallet's last 20–30 completed trades (round-trip buys + sells):

1. List all buy events: token, amount_usd, timestamp
2. List all sell events for the same tokens
3. Compute per-trade return: `(sell_usd - buy_usd) / buy_usd`
4. Average the returns

**If you want to estimate "if I had followed today":**
For still-open positions (buy with no matching sell), use holdings view to get current `usd_value` vs `cost`, computing unrealized return.

Present as:
```
Copy-trade estimate (last 30d completed trades):
  Avg return per trade: +X%
  Win rate:             X / Y trades profitable
  Best trade:           +X% on TOKEN
  Worst trade:          -X% on TOKEN
  Approximate 30d return if equal-weight copy: ~X%
⚠️ This is an approximation. Actual results depend on entry timing, slippage, and fees.
```

### Step 5 — Smart Money Leaderboard (Multi-Wallet Comparison)

When the user wants to compare multiple smart money wallets:

Use the **gmgn-portfolio** skill to batch-stats up to 10 wallets at once for `30d`.

Rank wallets by composite score. Suggested weights:
- `winrate` × 40%
- `pnl` × 40%
- `token_num` (diversity) × 10%
- Recency (7d winrate vs 30d winrate improvement) × 10%

To discover active smart money wallets to compare, first use the **gmgn-track** skill's smart money feed to extract unique wallet addresses, then batch-query their stats.

## Output Template

The output MUST include the input parameters used and full wallet addresses for every wallet mentioned.

```
Smart Money Profile

📥 INPUT PARAMETERS
  Chain:           {chain}
  Wallet address:  {full_wallet_address}
  Data:            7d + 30d

─── Performance ────────────────────────────
Win Rate (7d / 30d):  {X}% / {X}%     [trend: ↑ improving / ↓ declining / → stable]
PnL Ratio (30d):      {X}x
Realized Profit (30d): ${X}

─── Trading Style ──────────────────────────
Style:          Scalper / Day trader / Swing trader / Long-term holder
Avg Hold Time:  ~{X} hours / days
Position Size:  Consistent (disciplined) / Variable (opportunistic)
Token Focus:    Specialist (repeats tokens) / Trend chaser (always new)

─── Exit Behavior ──────────────────────────
Typical take-profit: ~+{X}% gain
Typical stop-loss:   ~-{X}% loss
Win/loss ratio:      {avg_win}x / {avg_loss}x

─── Copy-Trade Estimate ────────────────────
Approx. 30d return if copied: ~{X}%
Based on {N} completed trades
⚠️ Approximation only

─── Verdict ────────────────────────────────
🟢 High-conviction follow — strong stats, consistent style, favorable exit pattern
🟡 Selective follow — good stats but inconsistent or high-risk behavior
🔴 Avoid copying — low win rate, poor exit discipline, or declining form

─── FULL ADDRESSES ─────────────────────────
  Wallet:           {full_wallet_address}
  {If other wallets mentioned (leaderboard), list them here}
```

## Follow-Up Actions

- General wallet quality assessment: use **gmgn-wallet-analysis** skill
- Deep dive on tokens this wallet holds: use **gmgn-token-research** skill

## Dependencies

This skill requires the following companion skills to be installed and eligible:
- **gmgn-portfolio** — stats, activity, holdings, and wallet field knowledge
- **gmgn-track** — smart money discovery for leaderboard comparison