---
name: gmgn-token-due-diligence
description: 'Orchestrate a fast GMGN token due diligence checklist before buying: basic info -> security review -> liquidity pool check -> smart money signals -> concise buy or avoid verdict. Delegates CLI execution to the gmgn-token skill. Use when the user wants a quick pre-buy or pre-swap safety check rather than a full research report. Requires: gmgn-cli, GMGN_API_KEY, and gmgn-token installed.'
argument-hint: "--address <token_address> [--chain <sol|bsc|base|eth>]"
---

# GMGN Token Due Diligence

Orchestration skill. Delegates all CLI commands and field interpretation to the **gmgn-token** skill — this file only defines the fast four-step checklist, warning thresholds, and output format. Do not inline CLI syntax here; refer to the referenced skill for command details.

Use when: quick pre-buy check, pre-swap due diligence, should I buy this token, or when the user wants a compact safety review instead of a full research report.

## Parameters

| Param | Default | Description |
|-------|---------|-------------|
| `--chain` | `sol` | `sol` / `bsc` / `base` / `eth` |
| `--address` | required | Token address to review |

## Workflow

### Step 1 — Basic Info

Use the **gmgn-token** skill's info view.

Check:
- Price and current liquidity
- Holder count and whether the holder base looks non-trivial
- `wallet_tags_stat.smart_wallets` and `wallet_tags_stat.renowned_wallets`
- Whether any meaningful social presence exists

Immediate concerns:
- No social presence at all
- Very low liquidity
- No smart money or KOL presence at all

### Step 2 — Security Review

Use the **gmgn-token** skill's security view.

Pay special attention to:
- `is_honeypot`
- `open_source` and `owner_renounced`
- On Solana, `renounced_mint` and `renounced_freeze_account`
- `buy_tax` and `sell_tax`
- `top_10_holder_rate`
- `rug_ratio`
- `creator_token_status`
- `sniper_count`

Hard-stop examples:
- Honeypot or equivalent sell trap
- Very high `rug_ratio`
- Unrenounced mint or freeze authority on Solana
- Very high tax or concentration risk

### Step 3 — Liquidity Pool Check

Use the **gmgn-token** skill's pool view.

Assess:
- Liquidity amount and likely slippage
- Exchange quality
- Pool age

Very low liquidity should sharply reduce confidence, even if other signals look acceptable.

### Step 4 — Smart Money Signals

Use the **gmgn-token** skill's holders and traders views for smart money and renowned wallets.

Bullish signs:
- Smart money wallets still buying or holding strongly
- Unrealized posture suggests conviction remains
- KOL wallets are accumulating rather than distributing

Bearish signs:
- Smart money sell volume exceeds buy volume
- Large realized profits are already taken and conviction may be gone
- Top holders with very large supply shares are starting to distribute

### Step 5 — Verdict

Summarize the token into one quick outcome:
- 🟢 **Looks buyable** — no major danger signal and some meaningful conviction exists
- 🟡 **Needs caution** — mixed picture, thin liquidity, or weak conviction
- 🔴 **Do not buy** — one or more strong danger signals appear

If a hard stop is present, say so explicitly and do not dilute the wording.

## Output Format

Before rendering the due-diligence summary:
- Always display the full on-chain token address whenever the token is referenced.
- Symbols or token names may be shown only as secondary context after the full address.
- Never shorten any address with `...` or any other ellipsis form.

```
═══════════════════════════════════════════
  TOKEN DUE DILIGENCE — {address}
  {chain} | Symbol: {symbol}
═══════════════════════════════════════════

BASIC INFO
- Liquidity: ${x}
- Holders: {x}
- Smart Wallets: {x}
- Social Presence: Strong / Weak / None

SECURITY
- Honeypot: Yes / No
- Rug Ratio: {x}
- Authority Status: Safe / Mixed / Unsafe
- Concentration: Low / Medium / High

POOL
- Exchange: {exchange}
- Pool Age: {x}
- Slippage Risk: Low / Medium / High

SMART MONEY
- Smart Money Direction: Buying / Mixed / Selling
- KOL Posture: Accumulating / Mixed / Exiting

VERDICT
- 🟢 Looks buyable / 🟡 Needs caution / 🔴 Do not buy
- Reason: {1-2 sentence summary}
═══════════════════════════════════════════
```

## Dependencies

This skill requires the following companion skill to be installed and eligible:
- **gmgn-token** — info, security, pool, holders, traders, and field knowledge

Optional downstream follow-up skills:
- **gmgn-swap** — execution after explicit user confirmation
- **gmgn-project-deep-report** — deeper multi-factor analysis if the user wants a larger position decision