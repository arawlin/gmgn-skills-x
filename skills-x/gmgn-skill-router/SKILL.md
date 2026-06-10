---
name: gmgn-skill-router
description: 'Route a user query to the correct GMGN orchestration skill. When the user describes a goal but is unsure which skill to use — e.g. "how do I analyze a token", "what skill do I use to check risks", "I want to find new coins" — match their intent against the routing table and recommend the right skill(s) with full invocation syntax. Also activates when the user asks "which skill" or "what skill" in the context of GMGN workflows. Delegates to other orchestration skills; does not execute CLI commands directly.'
argument-hint: "<user's stated goal or question>"
---

# GMGN Skill Router

Meta-orchestration skill. Does NOT execute CLI commands or call base skills directly. Instead, it routes the user's stated goal to the correct orchestration skill(s) and outputs the recommended invocation.

Use when:
- The user describes what they want to do but doesn't know which skill to call
- The user asks "which skill should I use for X" or "what skill does Y"
- The user's request is ambiguous and could match multiple orchestration skills
- The user wants to understand the available GMGN workflow options

## Workflow

### Step 1 — Parse the User's Goal

Read the user's query and extract:
- **Domain**: token analysis, wallet analysis, market discovery, risk monitoring, or daily overview
- **Depth**: quick check vs full research vs deep report
- **Input**: do they have a token address, a wallet address, or no address (discovery mode)
- **Action**: are they looking to buy, monitor, or just explore

### Step 2 — Match Against the Routing Table

Use the table below to identify which orchestration skill(s) match the user's goal.

| User Goal | Domain | Has Address? | Depth | Recommended Skill |
|-----------|--------|-------------|-------|-------------------|
| Get today's market overview | Market | No | Overview | `gmgn-daily-brief` |
| Find opportunities from trending | Market | No | Discovery | `gmgn-market-opportunities` |
| Screen newly launched tokens | Market | No | Discovery | `gmgn-early-project-screening` |
| Quick pre-buy safety check | Token | Yes | Quick | `gmgn-token-due-diligence` |
| Full token research with verdict | Token | Yes | Full | `gmgn-token-research` |
| Deep analysis for large position | Token | Yes | Deep | `gmgn-project-deep-report` |
| Check if a held token is at risk | Token | Yes | Monitoring | `gmgn-risk-warning` |
| Analyze a wallet's track record | Wallet | Yes | Full | `gmgn-wallet-analysis` |
| Profile wallet trading style & ROI | Wallet | Yes | Deep | `gmgn-smart-money-profile` |

### Step 3 — Handle Ambiguous or Multi-Skill Cases

If the user's goal maps to more than one skill, present options ranked by fit:

| Pattern | Recommendation |
|---------|---------------|
| "I want to find and research tokens" | 1️⃣ `gmgn-market-opportunities` (discover) → 2️⃣ `gmgn-token-research` (research top pick) |
| "Should I buy this token?" | If urgent: `gmgn-token-due-diligence`. If there's time: `gmgn-token-research`. If large position: `gmgn-project-deep-report` |
| "I want to follow smart money" | 1️⃣ `gmgn-wallet-analysis` (evaluate a specific wallet) → 2️⃣ `gmgn-smart-money-profile` (understand style) |
| "I hold tokens, are they safe?" | `gmgn-risk-warning` for each held token |
| "What's happening in the market?" | `gmgn-daily-brief` for full overview; `gmgn-market-opportunities` if only interested in picks |

### Step 4 — Output the Recommendation

For each matched skill, output:
1. **Skill name** (the exact name to invoke)
2. **Why it fits** (one sentence mapping the user's goal to the skill's purpose)
3. **Full invocation syntax** with argument hints
4. **What to expect** (the verdict type or output format the skill produces)
5. **What to do next** (downstream follow-up skills, if applicable)

## Output Format

```
═══════════════════════════════════════════
  SKILL ROUTER — Goal: {user's stated goal}
═══════════════════════════════════════════

🔍 MATCHED SKILL(S)

1️⃣  {skill_name}
    Why:      {one-line reason this skill fits the user's goal}
    Invoke:   {skill_name} {argument-hint}
    Returns:  {verdict type, e.g. "Buy / Watch / Skip"}
    Next:     {recommended follow-up skill, or "None — conclusion is self-contained"}

2️⃣  {skill_name}  (if multiple matches)
    ...

─── RECOMMENDED PIPELINE ─────────────────
  {if multi-step, show the flow: Skill A → Skill B → Skill C}
  {if single step, state: "This goal is fully covered by one skill."}
═══════════════════════════════════════════
```

## Routing Reference (Complete)

The full routing table with all 9 orchestration skills, their parameters, and decision logic is maintained in the companion document at `skills-x/ORCHESTRATION-SKILLS.md`. When detailed parameter defaults or dependency chains are needed, consult that file.

### Quick Mapping by Trigger Word

| User says (keywords) | → Skill |
|----------------------|---------|
| daily brief, today's market, market overview, 每日简报, 今天市场 | `gmgn-daily-brief` |
| trending, hot tokens, top volume, what's pumping, 热门, 榜单, 发现机会 | `gmgn-market-opportunities` |
| new tokens, just launched, pump.fun, early entry, 新币, 刚上, 埋伏 | `gmgn-early-project-screening` |
| is this token safe, quick check, pre-buy, 安全吗, 快速看, 能买吗 | `gmgn-token-due-diligence` |
| research this token, should I buy, due diligence, 研究, 尽调, 值得买吗 | `gmgn-token-research` |
| deep report, full analysis, large position, 深度, 全面, 重仓 | `gmgn-project-deep-report` |
| risk, danger, whales dumping, liquidity drain, 风险, 危险, 巨鲸, 跑路 | `gmgn-risk-warning` |
| analyze wallet, track record, worth following, 分析钱包, 战绩, 值不值得跟 | `gmgn-wallet-analysis` |
| trading style, copy-trade ROI, smart money profile, 交易风格, 跟单, 画像, 短线长线 | `gmgn-smart-money-profile` |

### Quick Mapping by Input Type

| User provides | Best starting skill |
|--------------|-------------------|
| No address — wants to browse | `gmgn-daily-brief` or `gmgn-market-opportunities` |
| Token address — wants safety check | `gmgn-token-due-diligence` |
| Token address — wants research | `gmgn-token-research` |
| Token address — wants deep report | `gmgn-project-deep-report` |
| Token address — already holding | `gmgn-risk-warning` |
| Wallet address — wants evaluation | `gmgn-wallet-analysis` |
| Wallet address — wants style profile | `gmgn-smart-money-profile` |

## Dependencies

This skill does NOT execute CLI commands and has no base-skill dependencies for its own operation. It delegates to other orchestration skills, which in turn depend on the base skills (`gmgn-market`, `gmgn-token`, `gmgn-track`, `gmgn-portfolio`).

For the router's recommendations to be actionable, the recommended orchestration skills and their base-skill dependencies must be installed. See `skills-x/ORCHESTRATION-SKILLS.md` for the full dependency map.
