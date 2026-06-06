---
name: gmgn-wallet-analysis
description: 'Orchestrate a GMGN wallet analysis workflow for a specific wallet: current holdings -> 30d stats -> recent activity -> optional follow-wallet trade feed -> verdict. Delegates CLI execution to gmgn-portfolio and gmgn-track skills. Use when asked whether a wallet is worth following, what its investment style is, or how strong its track record looks. Requires: gmgn-cli, GMGN_API_KEY, and the referenced skills installed. Some steps require GMGN_PRIVATE_KEY.'
argument-hint: "--wallet <wallet_address> [--chain <sol|bsc|base|eth>] [--period <7d|30d>] [--holdings-limit <number>] [--top-holdings <number>]"
metadata: {"openclaw": {"requires": {"bins": ["gmgn-cli"], "env": ["GMGN_API_KEY"]}, "primaryEnv": "GMGN_API_KEY"}}
---

# GMGN Wallet Analysis

Orchestration skill. Delegates all CLI commands and field interpretation to the **gmgn-portfolio** and **gmgn-track** skills — this file only defines the multi-step workflow, analysis rules, and output format. Do not inline CLI syntax here; refer to the referenced skills for command details.

Use when: wallet analysis, is this wallet worth following, wallet track record review, wallet investment style, or whether a wallet shows consistent profitable behavior.

## Parameters

| Param | Default | Description |
|-------|---------|-------------|
| `--chain` | `sol` | `sol` / `bsc` / `base` / `eth` |
| `--wallet` | required | Wallet address to analyze |
| `--period` | `30d` | Stats lookback window |
| `--holdings-limit` | `50` | Holdings/activity depth to review |
| `--top-holdings` | `3` | Number of top current positions to summarize |

## Workflow

### Step 1 — Current Holdings

Use the **gmgn-portfolio** skill to fetch the wallet's current holdings ordered by `usd_value`.

Assess:
- Position concentration: whether one token dominates or risk is spread out
- Current conviction: large profitable open positions suggest continued conviction
- `unrealized_profit` distribution across holdings
- `profit_change` patterns per position

### Step 2 — Trading Stats

Use the **gmgn-portfolio** skill to fetch wallet stats for `--period`.

Interpret:
- `winrate > 0.6` as strong, but only when supported by healthy PnL
- `realized_profit` as locked-in USD profit, not paper gains
- `pnl` as trade efficiency: `2.0` means the wallet doubled deployed capital on completed trades
- `buy_count` and `sell_count` as clues for frequency and style

### Step 3 — Recent Activity

Use the **gmgn-portfolio** skill to inspect recent activity.

Look for:
- Trading frequency: multiple trades per day suggests active trading
- Holding duration: compare buy vs sell timing where possible
- Token diversity: broad rotation versus focused conviction
- Position sizing discipline: consistent sizing is stronger than erratic bet sizing

### Step 4 — Optional Follow-Wallet Feed

If the wallet is followed on GMGN and signed auth is available, use the **gmgn-track** skill's follow-wallet view.

If `GMGN_PRIVATE_KEY` is unavailable, skip this step and note that the conclusion is based on holdings, stats, and activity only.

Check:
- `is_open_or_close = 1` to distinguish full opens/closes from partial adds/reductions
- `price_change` to gauge how well recent entries aged
- Whether the live feed confirms the style inferred from stats and activity

### Step 5 — Top Holdings Quality Check

For the top `--top-holdings` positions by `usd_value`, use the **gmgn-portfolio** skill's holdings output and summarize whether the wallet concentrates in strong names, speculative names, or a mix.

If the user wants a deeper conclusion on any holding, hand off to the existing token-research workflow rather than expanding this skill.

### Step 6 — Verdict

Assign one verdict:
- 🟢 **Worth following** — strong win rate, solid realized PnL, coherent style, and current holdings do not look reckless
- 🟡 **Watch first** — promising metrics but limited sample size, mixed consistency, or missing signed-auth follow feed
- 🔴 **Not recommended** — weak win rate, poor realized results, erratic behavior, or obvious high-risk patterns

Call out uncertainty explicitly when data is partial.

## Output Format

```
═══════════════════════════════════════════
  WALLET ANALYSIS — {short_wallet} — {chain}
═══════════════════════════════════════════

Period: {period}

─── PERFORMANCE ──────────────────────────
Win Rate:       {winrate}%
Realized P&L:   ${realized_profit}
PnL Ratio:      {pnl}x
Trades:         {buy_count} buys / {sell_count} sells

─── STYLE ────────────────────────────────
Trading Style:  Day trader / Swing trader / Holder
Token Focus:    Meme / DeFi / Mixed / Specific sector
Sizing Pattern: Consistent / Mixed / Erratic

─── CURRENT POSITIONS ────────────────────
Top holdings:   {token1}, {token2}, {token3}
Open P&L:       ${total_unrealized}
Concentration:  High / Medium / Low

─── FOLLOW FEED ──────────────────────────
Available:      Yes / No
Read-through:   Recent opens/closes confirm style / Not available / Mixed

─── VERDICT ──────────────────────────────
🟢 Worth following / 🟡 Watch first / 🔴 Not recommended
Reason: {1-2 sentence justification}
═══════════════════════════════════════════
```

## Dependencies

This skill requires the following companion skills to be installed and eligible:
- **gmgn-portfolio** — holdings, stats, activity, and field knowledge
- **gmgn-track** — optional follow-wallet feed and signed-auth nuances

Install via `openclaw skills install` or symlink them from the gmgn-skills repo.