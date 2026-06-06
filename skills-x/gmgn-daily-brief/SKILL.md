---
name: gmgn-daily-brief
description: 'Orchestrate a structured GMGN daily market brief: market pulse → smart money moves → new token watch → risk scan. Delegates CLI execution to gmgn-market, gmgn-track, and gmgn-token skills. Requires: gmgn-cli, GMGN_API_KEY, and the three referenced skills installed.'
argument-hint: "[--chain <sol|bsc|base|eth>]"
---

# GMGN Daily Market Brief

Orchestration skill. Delegates all CLI commands and field interpretation to the **gmgn-market**, **gmgn-track**, and **gmgn-token** skills — this file only defines the multi-step workflow and output format. Do not inline CLI syntax here; refer to the referenced skills for command details.

Use when: daily brief, market overview, smart money check, periodic market monitoring.

## Parameters

| Param | Default | Description |
|-------|---------|-------------|
| `--chain` | `sol` | `sol` / `bsc` / `base` / `eth` |

## Workflow

### Step 1 — Market Pulse

Use the **gmgn-market** skill to fetch trending tokens at `1h` and `6h` intervals, sorted by volume, limit 20.

Assess from results:
- **Market phase** — meme-dominated (risk-on) vs utility/DeFi (risk-off) vs mixed
- **Breadth** — many tokens trending or just 1–2
- **Smart money confirmation** — check `smart_degen_count` > 0
- **Volume quality** — compare `volume` vs `swaps`

### Step 2 — Smart Money Activity

Use the **gmgn-track** skill to check smart money and KOL trades in the last ~2h.

Group by direction (buying vs selling). Flag tokens with multiple wallets trading the same direction (cluster signal). Note any token appearing in both trending AND smart money data — double confirmation.

### Step 3 — New Token Watch

Use the **gmgn-market** skill to fetch trenches: `near_completion` and `completed`.

Quick filter: tokens with `smart_degen_count >= 1`, `rug_ratio < 0.2`, non-zero volume.

### Step 4 — Risk Scan

For top tokens from steps 1–2, use the **gmgn-token** skill to run security checks.

Flag: `rug_ratio > 0.3`, `top_10_holder_rate > 0.5`, `creator_token_status = creator_hold`, `is_wash_trading = true`.

## Output Format

Before rendering the brief:
- Always display the full on-chain token or wallet address for every referenced asset or wallet.
- Symbols, token names, and wallet labels are optional secondary context only and must never replace the full address.
- Never shorten any address with `...` or any other ellipsis form.
- Always include an `INPUT PARAMETERS` section that echoes the effective value of every declared input parameter.
- If the user omitted a parameter, still show the final default value that the skill used.

```
═══════════════════════════════════════════
  DAILY MARKET BRIEF — {chain} — {date}
═══════════════════════════════════════════

─── INPUT PARAMETERS ─────────────────────
  Chain (--chain): {chain}

📊 MARKET PULSE
  Phase:        Risk-on / Risk-off / Mixed
  Breadth:      {N} tokens trending (broad/narrow)
  Top movers:   {token_a_address} ({token_a_symbol}, +X%), {token_b_address} ({token_b_symbol}, +X%), {token_c_address} ({token_c_symbol}, +X%)
  Smart money:  Present in trending ✅ / Absent ⚠️

🧠 SMART MONEY MOVES
  Buying:
    • {token_a_address} ({token_a_symbol}) — {N} wallets accumulating, avg price_change +{X}%
    • {token_b_address} ({token_b_symbol}) — {N} wallets, fresh entry
  Selling:
    • {token_c_address} ({token_c_symbol}) — {N} wallets reducing
  Cluster signal: {token_a_address} ({token_a_symbol}, trending + smart money overlap) 🔥

🌱 EARLY WATCH
  • {token_x_address} ({token_x_symbol}) — near graduation, {N} smart degens, rug_ratio {X}
  • {token_y_address} ({token_y_symbol}) — just graduated, strong volume, clean security

⚠️ RISK SIGNALS
  • No active warnings / {token_z_address} ({token_z_symbol}): whale concentration rising (top_10 = {X}%)

─── SUGGESTED ACTIONS ─────────────────────
  Opportunity:  {token_a_address} ({token_a_symbol}) → run full token research
  Caution:      {token_c_address} ({token_c_symbol}) → smart money exiting
  New entry:    {token_x_address} ({token_x_symbol}) → early screening recommended
═══════════════════════════════════════════
```

## Dependencies

This skill requires the following companion skills to be installed and eligible:
- **gmgn-market** — trending, trenches commands + field knowledge
- **gmgn-track** — smartmoney, kol commands + field knowledge
- **gmgn-token** — security command + field knowledge
