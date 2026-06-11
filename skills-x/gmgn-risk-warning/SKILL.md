---
name: gmgn-risk-warning
description: 'Orchestrate the canonical GMGN risk warning workflow for a held or watched token: security snapshot -> liquidity check -> whale holder analysis -> smart money flow -> price and volume anomaly check -> structured risk verdict. Delegates CLI execution to gmgn-token, gmgn-track, and gmgn-market skills. Use when asked whether whales are dumping, whether liquidity is still healthy, whether the developer may be exiting, or whether a position is becoming dangerous to hold. Requires: gmgn-cli, GMGN_API_KEY, and the referenced skills installed.'
argument-hint: "--address <token_address> [--chain <sol|bsc|base|eth>]"
---

# GMGN Risk Warning

Orchestration skill. Delegates all CLI commands and field interpretation to the **gmgn-token**, **gmgn-track**, and **gmgn-market** skills — this file only defines the multi-step monitoring workflow, warning thresholds, and output format. Do not inline CLI syntax here; refer to the referenced skills for command details.

Use when: risk warning, are whales dumping, is liquidity still healthy, are there signs the developer is exiting, or whether a held position is showing active danger signals.

## Parameters

| Param | Default | Description |
|-------|---------|-------------|
| `--chain` | `sol` | `sol` / `bsc` / `base` / `eth` |
| `--address` | required | Token address to assess |

## Workflow

### Step 1 — Token Security Snapshot

Use the **gmgn-token** skill's security view.

Immediate red flags (any one triggers danger):

| Field | Danger Signal |
|-------|--------------|
| `is_honeypot` | `"yes"` → sells are blocked, exit impossible (BSC/Base only) |
| `rug_ratio` | `> 0.3` → high rug pull probability |
| `top_10_holder_rate` | `> 0.5` → extreme concentration, whale exit risk |
| `creator_token_status` | `creator_hold` → dev still holds, dump risk active |
| `renounced_mint` (SOL) | `false` → dev can inflate supply at any time |
| `renounced_freeze_account` (SOL) | `false` → dev can freeze wallets |
| `sell_tax` | `> 0.10` → exit penalty is severe |
| `bundler_rate` | `> 0.3` → heavily bot-bundled at launch, artificial price support |
| `rat_trader_amount_rate` | `> 0.3` → insider trading detected |
| `is_wash_trading` | `true` → volume is fake |

### Step 2 — Liquidity Check

Use the **gmgn-token** skill's pool view.

Check for liquidity drain:

- **Current liquidity (USD):** < $10k = extreme exit slippage risk
- **Liquidity vs earlier baseline:** if you have a prior reading, compare. A drop of > 30% in a short period is a warning signal.
- **Pool age (`creation_timestamp`):** very new pools (< 1h) combined with other risk signals = high risk.
- **DEX (`exchange`):** verify it's a known DEX (Raydium, Uniswap, PancakeSwap). Unknown or single-sided pools are suspicious.

### Step 3 — Whale Holder Analysis

Use the **gmgn-token** skill's holders view:
- Top holders by supply percentage, ordered by `amount_percentage` descending, top 20
- Smart money holders filtered to `smart_degen`, ordered by `amount_percentage` descending, top 20

Warning signals:

- **Concentration:** top 1–3 wallets hold > 20% combined → single exit can crash price
- **Smart money exodus:** zero or declining `smart_degen` holders = conviction leaving
- **Wallet tags:** wallets tagged `bundler` or `rat_trader` in top holders = insider concentration risk

### Step 4 — Recent Trade Flow (Smart Money Direction)

Use the **gmgn-track** skill's smart money feed, filtering results for the token address in question.

Check:

- Are smart money wallets **selling** this token recently? (`is_open_or_close` = 1 on sell side for kol/smartmoney)
- Is `price_change` on recent smart money buys negative? (their entry is underwater — they may exit)
- Cluster of sells from multiple tracked wallets = strong exit signal

### Step 5 — Price and Volume Anomaly (K-line)

Use the **gmgn-market** skill's kline view at `1h` resolution.

Look for:

- **Volume spike without price increase** — selling pressure absorbing buy volume
- **Price drop with volume spike** — active dump in progress
- **Volume collapse** — liquidity evaporating, exit windows closing
- **Consecutive red candles after ATH** — distribution phase

## Risk Summary Output

After running all steps, output a structured risk verdict. The output MUST include the input parameters used and full token/wallet addresses for every address mentioned.

```
Risk Assessment: {TOKEN_SYMBOL}

📥 INPUT PARAMETERS
  Chain:           {chain}
  Token address:   {full_token_address}
  Checked:         {timestamp}

─── Security ───────────────────────────────
Honeypot:            ✅ No / 🚫 YES — exit blocked
Rug ratio:           ✅ {X} / ⚠️ {X} / 🚫 {X} (> 0.3 danger)
Mint renounced:      ✅ Yes / 🚫 No
Dev holding:         ✅ Sold / 🚫 Still holding — dump risk

─── Liquidity ──────────────────────────────
Current liquidity:   ${X}  [✅ healthy / ⚠️ low / 🚫 critical]
Pool age:            {X} hours/days

─── Whale Concentration ────────────────────
Top 10 hold rate:    {X}%  [✅ < 20% / ⚠️ 20–50% / 🚫 > 50%]
Smart money holders: {N} wallets still in

─── Smart Money Flow ───────────────────────
Recent smart money: Buying ✅ / Mixed ⚠️ / Selling 🚫

─── Price Action ───────────────────────────
1h volume trend:     Normal / Spike (selling pressure) / Collapsing
Recent candles:      Accumulation / Distribution / Neutral

─── Overall Verdict ────────────────────────
🟢 No active risk signals — position appears stable
🟡 Watch closely — 1–2 warning signals present, monitor daily
🔴 HIGH RISK — multiple danger signals, consider exiting

─── FULL ADDRESSES ─────────────────────────
  Token:            {full_token_address}
  {If whale/smart money wallets mentioned, list them here}
```

## Follow-Up Actions

- Full pre-buy due diligence: use **gmgn-token-research** skill
- Comprehensive project analysis: use **gmgn-project-deep-report** skill

## Dependencies

This skill requires the following companion skills to be installed and eligible:
- **gmgn-token** — security, pool, holders, and field knowledge
- **gmgn-track** — smart money flow monitoring and direction signals
- **gmgn-market** — kline-based volume and price anomaly context