---
name: gmgn-token-research
description: 'Orchestrate the canonical GMGN token research workflow for a token: basic info -> security -> liquidity pool -> market heat -> smart money signals -> structured buy, watch, or skip verdict. Delegates CLI execution to gmgn-token and gmgn-market skills. Use when asked whether a token is worth researching or buying, or when the user wants the full workflow rather than a quick checklist. Requires: gmgn-cli, GMGN_API_KEY, and the referenced skills installed.'
argument-hint: "--address <token_address> [--chain <sol|bsc|base|eth>]"
---

# GMGN Token Research

Orchestration skill. Delegates all CLI commands and field interpretation to the **gmgn-token** and **gmgn-market** skills — this file only defines the full research workflow, decision thresholds, and output format. Do not inline CLI syntax here; refer to the referenced skills for command details.

Use when: full token research, token due diligence, should I buy this token, or when the user wants a structured buy/watch/skip answer instead of a quick pre-buy screen.

## Parameters

| Param | Default | Description |
|-------|---------|-------------|
| `--chain` | `sol` | `sol` / `bsc` / `base` / `eth` |
| `--address` | required | Token address to analyze |

## Workflow

### Step 1 — Basic Info

Use the **gmgn-token** skill's info view.

Check:
- `price`
- `market_cap` (= `price × circulating_supply`)
- `liquidity`
- `holder_count`
- `wallet_tags_stat.smart_wallets` and `wallet_tags_stat.renowned_wallets`
- `link.website`, `link.twitter_username`, and `link.telegram`

Red flags:
- All social fields are empty
- Liquidity < $10k
- No smart money and no renowned wallets at all

### Step 2 — Security Check

Use the **gmgn-token** skill's security view.

Check each field against these exact workflow thresholds:

| Field | Safe ✅ | Warning ⚠️ | Danger 🚫 |
|-------|---------|-----------|---------|
| `is_honeypot` | `"no"` | — | `"yes"` → Stop immediately. Do not buy. BSC/Base only; empty string on SOL means not applicable. |
| `open_source` | `"yes"` | `"unknown"` | `"no"` |
| `owner_renounced` | `"yes"` | `"unknown"` | `"no"` |
| `renounced_mint` (SOL) | `true` | — | `false` → mint risk |
| `renounced_freeze_account` (SOL) | `true` | — | `false` → freeze risk |
| `buy_tax` / `sell_tax` | `0` | `0.01–0.05` | `>0.10` |
| `top_10_holder_rate` | `<0.20` | `0.20–0.50` | `>0.50` |
| `rug_ratio` | `<0.10` | `0.10–0.30` | `>0.30` |
| `creator_token_status` | `creator_close` | — | `creator_hold` |
| `sniper_count` | `<5` | `5–20` | `>20` |

If `is_honeypot = "yes"`, stop immediately and display exactly: `🚫 HONEYPOT DETECTED — Do not buy this token.` Do not proceed to later steps.

### Step 3 — Liquidity Pool

Use the **gmgn-token** skill's pool view.

Check:
- Liquidity amount
- Which DEX (`exchange`)
- Pool age (`creation_timestamp`)

Low liquidity should count as a real execution risk even if the token looks attractive elsewhere.

### Step 4 — Market Heat

Use the **gmgn-market** skill's trending view.

Check:
- Whether the token is currently trending
- Query market heat with the workflow's current window fixed at `1h`
- If present, note `rank`, `smart_degen_count`, `volume`, and `price_change_percent1h`

Interpretation:
- If found, that confirms active market interest
- If not found, treat it as neutral rather than an automatic rejection

### Step 5 — Smart Money Signals

Use the **gmgn-token** skill's holders and traders views.

Assess:
- Smart money holders ordered by `buy_volume_cur` descending, top 20, filtered to `smart_degen`
- KOL traders ordered by `profit` descending, top 20, filtered to `renowned`
- Whether smart money wallets are accumulating or distributing
- Whether unrealized posture suggests conviction is still open
- Whether KOL wallets are active holders or already exiting
- Whether top holders with high `amount_percentage` are starting to sell

Bullish signs:
- Smart degen wallets are buying heavily
- `unrealized_profit` is large and they still appear to be holding
- `sell_volume_cur` stays low

Bearish signs:
- `sell_volume_cur > buy_volume_cur` for smart money
- Large realized profits have already been taken and wallets may be exiting
- Top holders with very high `amount_percentage` are starting to sell

## Decision Framework

After completing all steps, present a structured conclusion.

Scoring logic:
- If any 🚫 appears, return **Skip**
- If 3 or more ⚠️ appear with no 🚫, return **Watch**
- If mostly ✅ signals are present and smart money is accumulating, return **Buy**

## Output Format

Render the conclusion in this exact structure:

```
Token Research Summary: {symbol} ({chain})
Address: {short_address}
─── Security ──────────────────────────────
Honeypot:         ✅ no / 🚫 YES — STOP
Contract verified:✅ yes / 🚫 no / ⚠️ unknown
Owner renounced:  ✅ yes / 🚫 no / ⚠️ unknown
Rug risk:         {rug_ratio} → ✅ low / ⚠️ medium / 🚫 high
Top-10 holders:   {top_10_holder_rate%} → ✅ <20% / ⚠️ 20–50% / 🚫 >50%
─── Liquidity ─────────────────────────────
Pool liquidity:   ${liquidity} on {exchange}
─── Market Heat ───────────────────────────
Trending:         yes (rank #{rank}) / not trending
─── Smart Money ───────────────────────────
SM holders: {smart_wallets}  |  KOL holders: {renowned_wallets}
SM activity: accumulating / distributing / absent
─── Verdict ───────────────────────────────
🟢 Buy — strong signals across all dimensions
🟡 Watch — mixed signals, monitor for confirmation
🔴 Skip — red flags present (specify which)
```

## Follow-Up Actions

- Run deeper multi-factor report: use **gmgn-project-deep-report** skill
- Execute swap: use **gmgn-swap** skill

## Dependencies

This skill requires the following companion skills to be installed and eligible:
- **gmgn-token** — info, security, pool, holders, traders, and field knowledge
- **gmgn-market** — trending context and field knowledge