---
name: gmgn-token-due-diligence
description: 'Orchestrate the canonical GMGN token due diligence checklist before buying: basic info -> security review -> liquidity pool check -> smart money signals -> concise buy or avoid verdict. Delegates CLI execution to the gmgn-token skill. Use when the user wants a quick pre-buy or pre-swap safety check rather than a full research report. Requires: gmgn-cli, GMGN_API_KEY, and gmgn-token installed.'
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

### Step 1 — Get Basic Info

Use the **gmgn-token** skill's info view.

Check: `price`, `liquidity`, `holder_count`, `wallet_tags_stat.smart_wallets`, `wallet_tags_stat.renowned_wallets`, `link.website` / `link.twitter_username` / `link.telegram`.

**Red flags**: all `link.*` social fields empty, very low liquidity (<$10k), zero `wallet_tags_stat.smart_wallets` and `renowned_wallets`.

### Step 2 — Check Security

Use the **gmgn-token** skill's security view.

Check these fields and their safe thresholds:

| Field | Safe | Warning | Danger |
|-------|------|---------|--------|
| `is_honeypot` | `"no"` | — | `"yes"` → Do not buy |
| `open_source` | `"yes"` | `"unknown"` | `"no"` |
| `owner_renounced` | `"yes"` | `"unknown"` | `"no"` |
| `renounced_mint` (SOL) | `true` | — | `false` → mint risk |
| `renounced_freeze_account` (SOL) | `true` | — | `false` → freeze risk |
| `buy_tax` / `sell_tax` | `0` | `0.01–0.05` | `>0.10` → high tax |
| `top_10_holder_rate` | `<0.20` | `0.20–0.40` | `>0.50` → whale risk |
| `rug_ratio` | `<0.10` | `0.10–0.30` | `>0.30` → high rug risk |
| `creator_token_status` | `creator_close` | — | `creator_hold` → dev not sold |
| `sniper_count` | `<5` | `5–20` | `>20` → heavily sniped |

### Step 3 — Check Liquidity Pool

Use the **gmgn-token** skill's pool view.

Check: liquidity amount, which DEX (`exchange`), pool age (`creation_timestamp`). Low liquidity means high slippage risk when buying or selling.

### Step 4 — Check Smart Money Signals

Use the **gmgn-token** skill's holders and traders views:
- Smart money holders: filter to `smart_degen`, ordered by `buy_volume_cur` descending, top 20
- KOL traders: filter to `renowned`, ordered by `profit` descending, top 20

**Bullish signals**: smart_degen wallets buying heavily, `unrealized_profit` is large (still holding), renowned wallets accumulating, low `sell_volume_cur`.

**Bearish signals**: `sell_volume_cur > buy_volume_cur` for smart money, large realized profits already taken (they may be done), top holders with very high `amount_percentage` starting to sell.

## Verdict

- 🟢 **Looks buyable** — no major danger signal and some meaningful conviction exists
- 🟡 **Needs caution** — mixed picture, thin liquidity, or weak conviction
- 🔴 **Do not buy** — one or more strong danger signals appear

## Due Diligence Output

The output MUST include the input parameters used and the full token address.

```
Token Due Diligence: {SYMBOL}

📥 INPUT PARAMETERS
  Chain:           {chain}
  Token address:   {full_token_address}

─── Basic Info ─────────────────────────────
  Price:          ${price}
  Market Cap:     ~${market_cap}
  Liquidity:      ${liquidity}
  Holders:        {holder_count}
  Smart wallets:  {smart_wallets}  |  KOL wallets: {renowned_wallets}

─── Security ──────────────────────────────
  {Key security checks with ✅/⚠️/🚫}

─── Liquidity Pool ────────────────────────
  Pool:           ${liquidity} on {exchange}

─── Smart Money ───────────────────────────
  {Bullish/bearish signals}

─── Verdict ───────────────────────────────
  🟢 Looks buyable / 🟡 Needs caution / 🔴 Do not buy
  Reason: {summary}

─── FULL ADDRESSES ─────────────────────────
  Token:            {full_token_address}
```

## Follow-Up Actions

- Full research report: use **gmgn-token-research** skill
- Deeper multi-factor analysis: use **gmgn-project-deep-report** skill
- Execute swap: use **gmgn-swap** skill

## Dependencies

This skill requires the following companion skill to be installed and eligible:
- **gmgn-token** — info, security, pool, holders, traders, and field knowledge