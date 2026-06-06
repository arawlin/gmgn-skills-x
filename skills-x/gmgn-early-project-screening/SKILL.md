---
name: gmgn-early-project-screening
description: 'Orchestrate a GMGN early project screening workflow for newly launched launchpad tokens: trenches fetch -> first-pass filter -> security checks -> smart money entry check -> verdicts. Delegates CLI execution to gmgn-market and gmgn-token skills. Use when asked which new tokens are worth watching, whether smart money entered early, or to screen Pump.fun and other launchpad tokens before full due diligence. Requires: gmgn-cli, GMGN_API_KEY, and the two referenced skills installed.'
argument-hint: "[--chain <sol|bsc|base|eth>] [--type <new_creation|near_completion|completed>...] [--filter-preset <safe|strict|none>] [--max-candidates <number>]"
---

# GMGN Early Project Screening

Orchestration skill. Delegates all CLI commands and field interpretation to the **gmgn-market** and **gmgn-token** skills — this file only defines the multi-step screening workflow, decision rules, and output format. Do not inline CLI syntax here; refer to the referenced skills for command details.

Use when: early project screening, new token screening, launchpad screening, which new tokens are worth watching, whether smart money entered early, or any new Pump.fun / FourMeme / launchpad tokens look worth accumulating.

## Parameters

| Param | Default | Description |
|-------|---------|-------------|
| `--chain` | `sol` | `sol` / `bsc` / `base` / `eth` |
| `--type` | `new_creation,near_completion` | Screening stage(s): `new_creation` / `near_completion` / `completed` |
| `--filter-preset` | `safe` | Baseline server-side prefilter: `safe` / `strict` / none |
| `--max-candidates` | `5` | Maximum shortlisted tokens to review deeply |

## Workflow

### Step 1 — Fetch New Launches

Use the **gmgn-market** skill to fetch trenches results for the requested `--type` values.

If the user does not specify a stage, start with `new_creation` and `near_completion`. Prefer server-side filtering when available: use `safe` as the baseline and `strict` when the user explicitly wants fewer but cleaner candidates.

From results, capture each token's `address`, `symbol`, `smart_degen_count`, `renowned_count`, `volume`, `swaps`, `rug_ratio`, and any insider, bundler, or wash-trading indicators exposed by the response.

### Step 2 — First-Pass Filter

Discard immediately if:
- `rug_ratio > 0.3`
- `is_wash_trading = true`
- `bundler_rate > 0.3`
- `rat_trader_amount_rate > 0.3`
- `smart_degen_count = 0` and `renowned_count = 0` and `volume < 10000`

Keep for deeper screening when any of:
- `smart_degen_count >= 1`
- `renowned_count >= 1`
- `bluechip_owner_percentage > 0`
- `volume` is strong relative to token age

Shortlist up to `--max-candidates` tokens.

### Step 3 — Security Check

Use the **gmgn-token** skill to run security checks on every shortlisted token.

Hard-stop and discard immediately if any of:
- `is_honeypot = "yes"` on `bsc` or `base`
- `renounced_mint = false` on `sol`
- `renounced_freeze_account = false` on `sol`
- `rug_ratio > 0.3`
- `sell_tax > 0.10`
- `top_10_holder_rate > 0.6`

### Step 4 — Early Smart Money Check

Use the **gmgn-token** skill's holders and traders views for each remaining token.

Strong signal:
- Multiple distinct `smart_degen` wallets entered early
- Entry timing looks recent relative to token age or last activity
- Top traders show positive `profit` or favorable PnL

Weak signal:
- Only one smart money wallet is present
- Smart money is present but already underwater
- Signal quality depends mostly on concentration instead of multiple wallets

### Step 5 — Info Spot Check

Use the **gmgn-token** skill's info view for each remaining token.

Check:
- At least one social signal exists in Twitter/X, Telegram, or website
- `holder_count` is non-trivial and appears to be growing
- `wallet_tags_stat.smart_wallets` confirms smart money presence
- `cto_flag = 1` can be neutral-to-positive when the context clearly indicates community takeover
- `creator_token_status = creator_close` reduces immediate dev dump risk, but for very new launches it can also imply low team commitment

### Step 6 — Verdict

Rate each token:
- 🟢 **Small position** — clean security, multiple early smart money entries, and social presence
- 🟡 **Watch** — some positive signals but missing confirmation; monitor for 30-60 minutes
- 🔴 **Skip** — any hard stop triggered, or no meaningful smart money interest

If no tokens pass, say so explicitly rather than forcing a pick.

## Output Format

Before rendering the screening report:
- Always display the full on-chain token address for every candidate, shortlist entry, and final recommendation.
- Symbols or token names may be shown only as secondary context next to the full address.
- Never shorten any address with `...` or any other ellipsis form.
- Always include an `INPUT PARAMETERS` section that echoes the effective value of every declared input parameter.
- If the user omitted a parameter, still show the final default value that the skill used.

```
═══════════════════════════════════════════
  EARLY PROJECT SCREENING — {chain} — {date}
═══════════════════════════════════════════

─── INPUT PARAMETERS ─────────────────────
  Chain (--chain): {chain}
  Type (--type): {type_values}
  Filter Preset (--filter-preset): {filter_preset}
  Max Candidates (--max-candidates): {max_candidates}

Screened: {N} tokens from trenches -> {M} shortlisted -> {K} passed security

# | Address | Symbol | Smart Degens | Rug Risk | Security | Verdict
1 | {token_1_address} | {token_1_symbol} | {N} wallets | {X} | ✅/⚠️/🚫 | Watch / Small position / Skip
2 | {token_2_address} | {token_2_symbol} | {N} wallets | {X} | ✅/⚠️/🚫 | Watch / Small position / Skip

─── TOP PICK ──────────────────────────────
{top_pick_address} ({top_pick_symbol}): Smart money in early, security clean, social present
Action: Small exploratory position / Watch 30-60m / Skip

─── NOTES ─────────────────────────────────
- Why shortlisted: {full_address} ({symbol}) because ...
- Why rejected: {full_address} ({symbol}) because ...
- Missing data / ambiguities: {full_address} ({symbol}) / none
═══════════════════════════════════════════
```

## Dependencies

This skill requires the following companion skills to be installed and eligible:
- **gmgn-market** — trenches discovery plus launch-stage field knowledge
- **gmgn-token** — security, holders, traders, and info plus field knowledge