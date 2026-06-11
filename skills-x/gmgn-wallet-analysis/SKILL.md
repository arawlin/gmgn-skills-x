---
name: gmgn-wallet-analysis
description: 'Orchestrate the canonical GMGN wallet analysis workflow for a specific wallet: current holdings -> 30d stats -> recent activity -> optional follow-wallet trade feed -> verdict. Delegates CLI execution to gmgn-portfolio and gmgn-track skills. Use when asked whether a wallet is worth following, what its investment style is, or how strong its track record looks. Requires: gmgn-cli, GMGN_API_KEY, and the referenced skills installed. Some steps require GMGN_PRIVATE_KEY.'
argument-hint: "--wallet <wallet_address> [--chain <sol|bsc|base|eth>]"
---

# GMGN Wallet Analysis

Orchestration skill. Delegates all CLI commands and field interpretation to the **gmgn-portfolio** and **gmgn-track** skills — this file only defines the multi-step workflow, analysis rules, and output format. Do not inline CLI syntax here; refer to the referenced skills for command details.

Use when: wallet analysis, is this wallet worth following, wallet track record review, wallet investment style, or whether a wallet shows consistent profitable behavior.

## Parameters

| Param | Default | Description |
|-------|---------|-------------|
| `--chain` | `sol` | `sol` / `bsc` / `base` / `eth` |
| `--wallet` | required | Wallet address to analyze |

## Workflow

### Step 1 — Current Holdings

Use the **gmgn-portfolio** skill to fetch the wallet's current holdings ordered by `usd_value` descending, limit 50.

Check: what tokens they hold, position sizes, `usd_value`, `unrealized_profit` distribution, `profit_change` per position. A wallet holding many positions with strong unrealized gains is still in accumulation mode.

### Step 2 — Trading Stats

Use the **gmgn-portfolio** skill to fetch wallet stats for `30d`.

Key metrics:
- `winrate` — ratio of profitable trades (0–1); > 0.6 is strong
- `realized_profit` — total USD profit locked in over 30 days
- `pnl` — profit/loss ratio = `realized_profit / total_cost`; `2.0` = doubled money
- `buy_count` / `sell_count` — trading frequency and style

### Step 3 — Recent Activity

Use the **gmgn-portfolio** skill to inspect recent activity, limit 50.

Look for:
- Trading frequency (multiple trades per day = active trader)
- Average holding duration: compare `last_active_timestamp` of buy vs sell events for the same token
- Token diversity: does the wallet trade many different tokens or focus on a few?
- Position sizing patterns: are buys consistent size or highly variable?

### Step 4 — If Wallet Is Followed on GMGN

If the user has followed this wallet on the GMGN platform:

> **Requires `GMGN_PRIVATE_KEY`** in `.env` — `track follow-wallet` uses signature auth. If the key is not configured, skip this step and note it in the conclusion.

Use the **gmgn-track** skill's follow-wallet view for this wallet.

Check `is_open_or_close` (1 = full position open/close, 0 = partial) and `price_change` (how well past trades aged).

### Step 5 — Deep Dive: Evaluate Their Top Holdings

For the top 3–5 holdings by `usd_value`, run full token research to verify the quality of what this wallet holds.

→ Use **gmgn-token-research** skill for the full token analysis on each top holding.

## Conclusion Framework

After completing all steps, output a wallet profile. The output MUST include the input parameters used and full wallet/token addresses for every address mentioned.

```
Wallet Analysis

📥 INPUT PARAMETERS
  Chain:           {chain}
  Wallet address:  {full_wallet_address}
  Period:          30d
─── Performance ────────────────────────────
Win Rate:       {winrate × 100}%
Realized P&L:   ${realized_profit}
PnL Ratio:      {pnl}x
Trades:         {buy_count} buys / {sell_count} sells
─── Style ──────────────────────────────────
Trading Style:  Day trader / Swing trader / Holder
                (Day trader: many trades/day; Swing: holds days–weeks; Holder: few sells)
Token Focus:    Meme / DeFi / Mixed / Specific sector
─── Current Positions ──────────────────────
Top holdings by value: {token1}, {token2}, {token3}
Open unrealized P&L: ${total_unrealized}
─── Smart Money Score ──────────────────────
Are their picks confirmed by other smart money? (check smart_degen_count on top holdings)
─── Verdict ────────────────────────────────
🟢 Worth following — strong win rate + consistent P&L + smart money overlap
🟡 Watch first — promising stats but limited data or inconsistent style
🔴 Not recommended — low win rate, losses, or high-risk behavior patterns

─── FULL ADDRESSES ─────────────────────────
  Wallet:           {full_wallet_address}
  {If top-holding token addresses mentioned, list them here}
```

## Follow-Up Actions

- Deeper behavior analysis (trading style, take-profit/stop-loss, copy-trade ROI): use **gmgn-smart-money-profile** skill

## Dependencies

This skill requires the following companion skills to be installed and eligible:
- **gmgn-portfolio** — holdings, stats, activity, and field knowledge
- **gmgn-track** — optional follow-wallet feed and signed-auth nuances