---
name: gmgn-daily-brief
description: 'Orchestrate a structured GMGN daily market brief: market pulse → smart money moves → new token watch → risk scan. Delegates CLI execution to gmgn-market, gmgn-track, and gmgn-token skills. Requires: gmgn-cli, GMGN_API_KEY, and the three referenced skills installed.'
metadata: {"openclaw": {"requires": {"bins": ["gmgn-cli"], "env": ["GMGN_API_KEY"]}, "primaryEnv": "GMGN_API_KEY"}}
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

```
═══════════════════════════════════════════
  DAILY MARKET BRIEF — {chain} — {date}
═══════════════════════════════════════════

📊 MARKET PULSE
  Phase:        Risk-on / Risk-off / Mixed
  Breadth:      {N} tokens trending (broad/narrow)
  Top movers:   TOKEN_A (+X%), TOKEN_B (+X%), TOKEN_C (+X%)
  Smart money:  Present in trending ✅ / Absent ⚠️

🧠 SMART MONEY MOVES
  Buying:
    • TOKEN_A — {N} wallets accumulating, avg price_change +{X}%
    • TOKEN_B — {N} wallets, fresh entry
  Selling:
    • TOKEN_C — {N} wallets reducing
  Cluster signal: TOKEN_A (trending + smart money overlap) 🔥

🌱 EARLY WATCH
  • TOKEN_X — near graduation, {N} smart degens, rug_ratio {X}
  • TOKEN_Y — just graduated, strong volume, clean security

⚠️ RISK SIGNALS
  • No active warnings / TOKEN_Z: whale concentration rising (top_10 = {X}%)

─── SUGGESTED ACTIONS ─────────────────────
  Opportunity:  TOKEN_A → run full token research
  Caution:      TOKEN_C → smart money exiting
  New entry:    TOKEN_X → early screening recommended
═══════════════════════════════════════════
```

## Dependencies

This skill requires the following companion skills to be installed and eligible:
- **gmgn-market** — trending, trenches commands + field knowledge
- **gmgn-track** — smartmoney, kol commands + field knowledge
- **gmgn-token** — security command + field knowledge

Install via `openclaw skills install` or symlink them from the gmgn-skills repo.
