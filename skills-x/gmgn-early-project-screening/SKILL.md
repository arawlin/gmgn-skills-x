---
name: gmgn-early-project-screening
description: 'Orchestrate the canonical GMGN early project screening workflow for newly launched launchpad tokens: trenches fetch -> first-pass filter -> security checks -> smart money entry check -> verdicts. Delegates CLI execution to gmgn-market and gmgn-token skills. Use when asked which new tokens are worth watching, whether smart money entered early, or to screen Pump.fun and other launchpad tokens before full due diligence. Requires: gmgn-cli, GMGN_API_KEY, and the two referenced skills installed.'
argument-hint: "[--chain <sol|bsc|base|eth>] [--type <new_creation|near_completion|completed>...]"
---

# GMGN Early Project Screening

Orchestration skill. Delegates all CLI commands and field interpretation to the **gmgn-market** and **gmgn-token** skills — this file only defines the multi-step screening workflow, decision rules, and output format. Do not inline CLI syntax here; refer to the referenced skills for command details.

Use when: early project screening, new token screening, launchpad screening, which new tokens are worth watching, whether smart money entered early, or any new Pump.fun / FourMeme / launchpad tokens look worth accumulating.

## Parameters

| Param | Default | Description |
|-------|---------|-------------|
| `--chain` | `sol` | `sol` / `bsc` / `base` / `eth` |
| `--type` | `new_creation,near_completion` | Screening stage(s): `new_creation` / `near_completion` / `completed` |

## Workflow

### Step 1 — Fetch Newly Launched Tokens

Use the **gmgn-market** skill to fetch trenches results for the requested `--type` values.

- `new_creation` — highest upside potential, highest risk. Many will fail.
- `near_completion` — approaching graduation, momentum building. Tighter window.
- `completed` — already trading on DEX, more liquidity but earlier gains may be gone.

If the user does not specify a stage, default to `new_creation` and `near_completion`.

From the results, note each token's `address`, `symbol`, `smart_degen_count`, `renowned_count`, `volume`, `swaps`, and `rug_ratio`.

Use server-side filter flags to pre-screen at fetch time:
- `--filter-preset safe` as the baseline (applies rug, bundler, and insider filters server-side before results are returned)
- `--filter-preset strict` when the user wants fewer but cleaner candidates
- Custom manual range filters (`--max-rug-ratio`, `--max-bundler-rate`, `--max-insider-ratio`, `--min-smart-degen-count`, `--min-volume-24h`) when precise control is needed

> If `--filter-preset safe` or `--filter-preset strict` was used, the rug_ratio, bundler_rate, and insider_ratio checks in Step 2 are already applied server-side. Verify the remaining signals manually.

### Step 2 — First-Pass Filter (In-Response Scan)

Before running any per-token CLI commands, apply a quick in-response filter on the trenches results.

**Discard immediately if:**
- `rug_ratio` > 0.3
- `is_wash_trading` = `true`
- `bundler_rate` > 0.3
- `rat_trader_amount_rate` > 0.3
- Zero `smart_degen_count` AND zero `renowned_count` AND volume < $10k

**Keep for deeper screening if any of:**
- `smart_degen_count` ≥ 1 (smart money has entered)
- `renowned_count` ≥ 1 (KOL has entered)
- `bluechip_owner_percentage` > 0 (quality wallet base)
- `volume` is strong relative to token age

Select up to **5 tokens** that pass this filter.

### Step 3 — Security Check (Per Token)

Use the **gmgn-token** skill to run security checks on every shortlisted token.

Hard stops — discard token immediately if:

| Field | Hard Stop |
|-------|-----------|
| `is_honeypot` | `"yes"` (BSC/Base) |
| `renounced_mint` (SOL) | `false` |
| `renounced_freeze_account` (SOL) | `false` |
| `rug_ratio` | `> 0.3` |
| `sell_tax` | `> 0.10` |
| `top_10_holder_rate` | `> 0.6` |

Proceed with tokens that pass all hard stops.

### Step 4 — Smart Money Early Entry Check

Use the **gmgn-token** skill's holders and traders views for each remaining token.

For holders: filter to `smart_degen`, ordered by `buy_volume_cur` descending, top 10.
For traders: ordered by `profit` descending, top 10.

**Strong signal:**
- Smart money wallets entered **early** (check `buy_30m` / `buy_1h` counts on the holder, or cross-reference `last_active_timestamp` being recent)
- Multiple distinct smart money wallets (not one large wallet — that's concentration risk)
- Top traders show profit (token has already rewarded early holders, positive momentum)

**Weak signal:**
- Only one smart money wallet in
- Smart money wallet entered but `profit` is negative (they're underwater)

### Step 5 — Token Info Spot Check

Use the **gmgn-token** skill's info view for each remaining token.

Check:
- Social presence: `link.twitter_username`, `link.telegram`, `link.website` — at least one should exist
- `holder_count` — growing is a positive sign
- `wallet_tags_stat.smart_wallets` — confirms smart money count
- `cto_flag` — if `1`, community has taken over, dev is gone (neutral to positive, evaluate context)
- `creator_token_status` — `creator_close` means dev has sold (mixed: reduces dump risk, but also less team commitment for very new tokens)

### Step 6 — Verdict

Rate each token:
- 🟢 **Small position** — clean security + smart money early entry + social presence
- 🟡 **Watch** — some positive signals but missing key indicators; monitor for 30–60 min
- 🔴 **Skip** — any hard stop triggered, or no smart money interest at all

## Screening Output

Present results as a table, then a per-token verdict. The output MUST include the input parameters used and full token addresses for every token mentioned.

```
Early Project Screening — {chain} / {type}

📥 INPUT PARAMETERS
  Chain:        {chain}
  Type:         {type}
  Filter preset: {filter_preset}

Screened: {N} tokens from trenches → {M} passed filter

# | Symbol | Address (short) | Smart Degens | Rug Risk | Security | Verdict
1 | ...     | ...             | {N} wallets  | {X}      | ✅/⚠️/🚫  | Watch / Small position / Skip
...

─── Top Pick ───────────────────────────────
{SYMBOL}: Smart money in early, security clean, social present
→ Suggested action: Small exploratory position / Watch for 1h / Skip

─── FULL ADDRESSES ─────────────────────────
  TOKEN_1 ({SYMBOL}):  {full_token_address}
  TOKEN_2 ({SYMBOL}):  {full_token_address}
  ...
```

**Verdict scale:**
- 🟢 **Small position** — clean security + smart money early entry + social presence
- 🟡 **Watch** — some positive signals but missing key indicators; monitor for 30–60 min
- 🔴 **Skip** — any hard stop triggered, or no smart money interest at all

## Follow-Up Actions

For any token rated 🟢:
- Run full due diligence: use **gmgn-token-research** skill
- Check risk warnings before sizing up: use **gmgn-risk-warning** skill
- Execute swap if satisfied: use **gmgn-swap** skill

## Dependencies

This skill requires the following companion skills to be installed and eligible:
- **gmgn-market** — trenches discovery plus launch-stage field knowledge
- **gmgn-token** — security, holders, traders, and info plus field knowledge