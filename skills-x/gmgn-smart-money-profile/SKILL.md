---
name: gmgn-smart-money-profile
description: 'Orchestrate a GMGN smart money profile workflow for a wallet: compare 7d and 30d stats -> infer trading style from activity -> estimate take-profit and stop-loss behavior -> approximate copy-trade ROI -> optionally rank multiple wallets in a leaderboard. Delegates CLI execution to gmgn-portfolio and gmgn-track skills. Use when asked what a wallet''s trading style is, when it takes profit or cuts losses, whether copying it would have worked, or which smart money wallets are best to follow. Requires: gmgn-cli, GMGN_API_KEY, and the referenced skills installed.'
argument-hint: "--wallet <wallet_address> [--chain <sol|bsc|base|eth>] [--activity-limit <number>] [--leaderboard-period <7d|30d>]"
metadata: {"openclaw": {"requires": {"bins": ["gmgn-cli"], "env": ["GMGN_API_KEY"]}, "primaryEnv": "GMGN_API_KEY"}}
---

# GMGN Smart Money Profile

Orchestration skill. Delegates all CLI commands and field interpretation to the **gmgn-portfolio** and **gmgn-track** skills — this file only defines the behavior-analysis workflow, copy-trade estimation logic, and output format. Do not inline CLI syntax here; refer to the referenced skills for command details.

Use when: smart money profile, what is this wallet's trading style, when does it take profit, when does it cut losses, if I copied this wallet what would my return be, or which smart money wallets are most worth following.

## Parameters

| Param | Default | Description |
|-------|---------|-------------|
| `--chain` | `sol` | `sol` / `bsc` / `base` / `eth` |
| `--wallet` | required | Wallet address for single-wallet profile mode |
| `--activity-limit` | `100` | Activity depth used for style inference |
| `--leaderboard-period` | `30d` | Default comparison window for multi-wallet ranking |

## Workflow

### Step 1 — Dual-Window Stats

Use the **gmgn-portfolio** skill to fetch stats for both `7d` and `30d`.

Assess:
- Whether `winrate` and `pnl` are improving or deteriorating
- Whether recent form is stronger or weaker than the broader 30d baseline
- Whether trade count and token count imply active rotation or selective trading

### Step 2 — Activity-Based Style Inference

Use the **gmgn-portfolio** skill's activity view.

Infer from completed buy and sell cycles:
- Approximate holding duration
- Scalper, day trader, swing trader, or longer-term holder behavior
- Position sizing consistency
- Whether the wallet is a specialist or a trend chaser

### Step 3 — Exit Pattern Analysis

Use the same activity data to approximate round-trip outcomes.

Look for:
- Typical take-profit level
- Typical stop-loss or drawdown tolerance
- Whether win sizes are larger than loss sizes
- Whether exits look disciplined or erratic

### Step 4 — Copy-Trade ROI Estimate

Approximate the wallet's last 20-30 completed trades using historical buy and sell activity.

Summarize:
- Average return per trade
- Number of profitable versus losing trades
- Best and worst recent trades
- Approximate equal-weight copy result over the recent sample

If the wallet still has open positions, optionally use the **gmgn-portfolio** skill's holdings view to note unrealized context, but keep the ROI estimate clearly labeled as approximate rather than a precise backtest.

### Step 5 — Optional Leaderboard Comparison

When the user wants multi-wallet comparison:
- Use the **gmgn-track** skill's smartmoney feed to discover active wallets when needed
- Use the **gmgn-portfolio** skill's batch stats to compare multiple wallets

Rank with a composite view of:
- Win rate
- PnL ratio
- Diversity or token count
- Short-term versus medium-term performance trend

### Step 6 — Verdict

Assign one conclusion:
- 🟢 **High-conviction follow** — strong stats, coherent style, favorable exit discipline
- 🟡 **Selective follow** — some edge is visible, but behavior or trend quality is mixed
- 🔴 **Avoid copying** — weak stats, poor exit pattern, or clearly declining form

Call out uncertainty whenever activity data is too sparse to infer style reliably.

## Output Format

```
═══════════════════════════════════════════
  SMART MONEY PROFILE — {short_wallet}
  {chain} | Data: 7d + 30d
═══════════════════════════════════════════

PERFORMANCE
- Win Rate (7d / 30d): {x}% / {y}%
- PnL Ratio (30d): {x}x
- Realized Profit (30d): ${x}
- Trend: Improving / Stable / Declining

TRADING STYLE
- Style: Scalper / Day trader / Swing trader / Long-term holder
- Avg Hold Time: {x}
- Position Size: Consistent / Variable
- Token Focus: Specialist / Trend chaser / Mixed

EXIT BEHAVIOR
- Typical Take-Profit: ~+{x}%
- Typical Stop-Loss: ~-{x}%
- Win/Loss Ratio: {avg_win} / {avg_loss}

COPY-TRADE ESTIMATE
- Approx. Return if Copied: ~{x}%
- Based on {n} completed trades
- Confidence: Low / Medium / High

VERDICT
- 🟢 High-conviction follow / 🟡 Selective follow / 🔴 Avoid copying
- Reason: {1-2 sentence summary}
═══════════════════════════════════════════
```

## Dependencies

This skill requires the following companion skills to be installed and eligible:
- **gmgn-portfolio** — stats, activity, holdings, and wallet field knowledge
- **gmgn-track** — smart money discovery for leaderboard comparison

Install via `openclaw skills install` or symlink them from the gmgn-skills repo.