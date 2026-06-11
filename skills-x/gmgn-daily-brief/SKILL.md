---
name: gmgn-daily-brief
description: 'Orchestrate the canonical GMGN daily market brief: market pulse → smart money moves → new token watch → risk scan. Delegates CLI execution to gmgn-market, gmgn-track, and gmgn-token skills. Requires: gmgn-cli, GMGN_API_KEY, and the three referenced skills installed.'
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

### Step 1 — Market Pulse (Trending Tokens)

Use the **gmgn-market** skill to fetch trending tokens at `1h` and `6h` intervals, sorted by volume, limit 20.

From the results, assess:
- **Market phase:** Are top tokens mostly meme/speculation (risk-on) or utility/DeFi (risk-off)?
- **Breadth:** Are many tokens trending or just 1–2? Broad trends = healthier market.
- **Smart money confirmation:** Do trending tokens have non-zero `smart_degen_count`? Trending without smart money = retail-driven pump.
- **Volume quality:** Compare `volume` vs `swaps`. High volume with low swap count = whale activity. High swaps with low volume = retail noise.

Key signal summary from this step:
```
Market Phase:    Risk-on (meme dominated) / Risk-off (utility) / Mixed
Breadth:         Broad ({N} tokens trending) / Narrow (1–2 tokens dominate)
Smart Money:     Confirmed in trending / Absent (retail-driven)
```

### Step 2 — Smart Money Activity (What Are They Buying/Selling?)

Use the **gmgn-track** skill to check smart money and KOL trades.

From the results:
- Group trades by direction: **net buying** vs **net selling** per token
- Identify tokens where **multiple** smart money wallets traded the same direction (cluster signal)
- Note `price_change` on each trade — positive = their past entries aged well (good track record lately)
- Flag any token appearing in both smart money AND trending data — double confirmation

Output for this step:
```
Smart Money Moves (last ~2h):
  Buying:  TOKEN_A ({N} wallets), TOKEN_B ({N} wallets)
  Selling: TOKEN_C ({N} wallets)
  Notable: TOKEN_A appears in both trending AND smart money buys → strong signal
```

### Step 3 — New Token Watch (Early Opportunities)

Use the **gmgn-market** skill to fetch trenches: `near_completion` and `completed`.

Quick filter: from results, surface tokens with:
- `smart_degen_count` ≥ 1
- `rug_ratio` < 0.2
- Non-zero `volume` and `swaps`

List up to 3 tokens that pass this quick filter as "early watch" candidates.

### Step 4 — Risk Scan (Anything to Avoid Today?)

For any tokens the user currently holds (if known), or for the top tokens from steps 1–2, use the **gmgn-token** skill to run security checks.

Flag immediately if any held/watched token shows:
- `rug_ratio` increase (compare to prior knowledge)
- `top_10_holder_rate` > 0.5
- `creator_token_status` = `creator_hold` (dev still in)
- `is_wash_trading` = `true`

If no specific tokens to check, skip this step and note it in the brief.

## Daily Brief Output

The output MUST include the input parameters used and full token addresses for every token mentioned.

```
═══════════════════════════════════════════
  DAILY MARKET BRIEF — {chain} — {date}
═══════════════════════════════════════════

📥 INPUT PARAMETERS
  Chain:        {chain}

📊 MARKET PULSE
  Phase:        Risk-on / Risk-off / Mixed
  Breadth:      {N} tokens trending (broad/narrow)
  Top movers:   TOKEN_A (+X%), TOKEN_B (+X%), TOKEN_C (+X%)
  Smart money:  Present in trending ✅ / Absent (retail-driven) ⚠️

🧠 SMART MONEY MOVES
  Buying:
    • TOKEN_A — {N} wallets accumulating, avg price_change +{X}%
    • TOKEN_B — {N} wallets, fresh entry
  Selling:
    • TOKEN_C — {N} wallets reducing positions
  Cluster signal: TOKEN_A (trending + smart money overlap) 🔥

🌱 EARLY WATCH
  • TOKEN_X — near graduation, {N} smart degens in, rug_ratio {X}
  • TOKEN_Y — just graduated, strong volume, clean security
  (Run /gmgn-token → workflow-early-project-screening for deeper check)

⚠️ RISK SIGNALS
  • No active warnings detected
  OR
  • TOKEN_Z: whale concentration rising (top_10 = {X}%), monitor closely

─── SUGGESTED ACTIONS ─────────────────────
  Opportunity:  TOKEN_A worth researching → run full token research
  Caution:      TOKEN_C seeing smart money exits → tighten stop
  New entry:    TOKEN_X early screening recommended

─── FULL ADDRESSES ────────────────────────
  TOKEN_A:  {full_token_address}
  TOKEN_B:  {full_token_address}
  TOKEN_C:  {full_token_address}
  TOKEN_X:  {full_token_address}
  TOKEN_Y:  {full_token_address}
  TOKEN_Z:  {full_token_address}
═══════════════════════════════════════════
```

## Follow-Up Actions

- Deep dive on an opportunity → use **gmgn-token-research** skill
- Screen early tokens further → use **gmgn-early-project-screening** skill
- Check a specific wallet that showed up → use **gmgn-smart-money-profile** skill
- Risk check on a held position → use **gmgn-risk-warning** skill
- Execute a trade → use **gmgn-swap** skill

## Dependencies

This skill requires the following companion skills to be installed and eligible:
- **gmgn-market** — trending, trenches commands + field knowledge
- **gmgn-track** — smartmoney, kol commands + field knowledge
- **gmgn-token** — security command + field knowledge
