# Orchestration Skills Reference — 编排技能速查

本文件列出 `skills-x/` 下所有编排技能的完整调用信息、适用场景和决策路由。在不确定该用哪个技能时，先查此表。

## 全部编排技能一览

| # | 技能名称 | 必填参数 | 可选参数 | 作用 | 什么时候用 |
|---|---------|---------|---------|------|-----------|
| 1 | `gmgn-daily-brief` | — | `--chain <sol\|bsc\|base\|eth>` | 每日市场简报：大盘脉搏 → 聪明钱动向 → 新币盯盘 → 风险扫描，一次性输出全局概况 | 想看今天市场怎么样、聪明钱在买什么、有哪些新币值得注意、有没有风险信号。也适合定时 cron 自动运行 |
| 2 | `gmgn-market-opportunities` | — | `--chain <sol\|bsc\|base\|eth>`, `--interval <1m\|5m\|1h\|6h\|24h>`, `--pool-size <number>`, `--top <number>` | 从热门榜中多因子筛选排名，输出 top 候选及理由，不针对单个代币做深度尽调 | 想看热门榜里哪些代币综合评分最好、有没有值得关注的标的、不想一个一个翻榜单 |
| 3 | `gmgn-early-project-screening` | — | `--chain <sol\|bsc\|base\|eth>`, `--type <new_creation\|near_completion\|completed>...`, `--filter-preset <safe\|strict\|none>`, `--max-candidates <number>` | 从新发 Launchpad 代币中做初筛：抓 trenches → 安全过滤 → 聪明钱入场检查 → 给出是否值得关注的判定 | 想看 Pump.fun / FourMeme 上新发的币、想知道哪些新项目有聪明钱早期进入、想抢先发现潜力标的 |
| 4 | `gmgn-token-due-diligence` | `--address <token_address>` | `--chain <sol\|bsc\|base\|eth>` | 快速买入前安全检查：基本信息 → 安全 → 流动池 → 聪明钱信号 → 买/谨慎/不买判定 | 准备买入一个代币前做快速安检、只需要一个快判而不需要完整研究报告 |
| 5 | `gmgn-token-research` | `--address <token_address>` | `--chain <sol\|bsc\|base\|eth>`, `--trending-interval <1m\|5m\|1h\|6h\|24h>` | 完整代币研究：基本信息 → 安全 → 流动池 → 市场热度 → 聪明钱信号 → 买入/观望/跳过判定 | 需要对一个代币做完整尽调、想知道这个币值不值得买、需要结构化的 buy/watch/skip 结论 |
| 6 | `gmgn-project-deep-report` | `--address <token_address>` | `--chain <sol\|bsc\|base\|eth>`, `--kline-resolution <1m\|5m\|15m\|1h\|4h\|1d>`, `--trending-interval <1m\|5m\|1h\|6h\|24h>` | 深度项目报告：基本面 → 安全 → 流动性 → 聪明钱持仓信心 → 价格走势 → 评分制判定。比 token-research 更深、更适合重仓决策 | 要全面分析一个项目、考虑较重仓位、需要多维度评分和完整文字报告 |
| 7 | `gmgn-risk-warning` | `--address <token_address>` | `--chain <sol\|bsc\|base\|eth>`, `--kline-resolution <1m\|5m\|15m\|1h\|4h\|1d>` | 持仓风险预警：安全快照 → 流动性检查 → 巨鲸持仓分析 → 聪明钱流向 → 量价异常 → 风险等级判定 | 持有一个代币后想知道是否危险、有没有巨鲸在出货、流动性是否健康、开发者是否在跑路 |
| 8 | `gmgn-wallet-analysis` | `--wallet <wallet_address>` | `--chain <sol\|bsc\|base\|eth>`, `--period <7d\|30d>`, `--holdings-limit <number>`, `--top-holdings <number>` | 钱包分析：当前持仓 → 30d 统计 → 近期活跃度 → 跟单质量评估 → 是否值得跟随判定 | 想分析一个钱包是否值得跟单、它的持仓和战绩如何、投资风格是什么 |
| 9 | `gmgn-smart-money-profile` | `--wallet <wallet_address>` | `--chain <sol\|bsc\|base\|eth>`, `--activity-limit <number>`, `--leaderboard-period <7d\|30d>` | 聪明钱行为画像：7d/30d 双窗口对比 → 交易风格推断 → 止盈止损习惯 → 跟单 ROI 估算 → 可选多钱包排行榜 | 想知道一个钱包是短线还是长线、什么时候止盈止损、跟着它买收益如何、多个聪明钱钱包中哪个最值得跟 |

## 决策路由：你的需求对应哪个技能

### 按需求类型

| 你想做什么 | 用哪个技能 | 备注 |
|-----------|-----------|------|
| 快速了解今天市场全貌 | `gmgn-daily-brief` | 一次看大盘+聪明钱+新币+风险，适合每日开盘 |
| 从热门榜里发现机会 | `gmgn-market-opportunities` | 不做深度尽调，只筛选排名，后续需转到 token-research |
| 看新发的 Launchpad 币 | `gmgn-early-project-screening` | 针对 Pump.fun / FourMeme 等新币，看聪明钱有没有早入 |
| 买币前快速安检 | `gmgn-token-due-diligence` | 4 步快速判买/不买，比 token-research 轻 |
| 完整研究一个代币 | `gmgn-token-research` | 6 步完整流程，输出 buy/watch/skip |
| 深度分析决定是否重仓 | `gmgn-project-deep-report` | 比 token-research 更深，多维度评分+文字报告 |
| 检查持有的币是否危险 | `gmgn-risk-warning` | 巨鲸出货/流动性枯竭/开发者跑路检查 |
| 分析钱包值不值得跟 | `gmgn-wallet-analysis` | 看战绩、持仓、风格，判定是否跟随 |
| 分析钱包交易风格和跟单质量 | `gmgn-smart-money-profile` | 比 wallet-analysis 更深，含止盈止损习惯和 ROI 估算 |

### 按典型触发词

| 用户说的关键词 | 应进入的技能 |
|---------------|-------------|
| 每日简报、今天市场、市场概况、聪明钱今天买了什么 | `gmgn-daily-brief` |
| 热门币、榜单、有什么币值得看、trending、发现机会 | `gmgn-market-opportunities` |
| 新币、刚上的币、pump.fun、新项目、早入、埋伏 | `gmgn-early-project-screening` |
| 这个币安全吗、能买吗、快速看一下、买入前检查 | `gmgn-token-due-diligence` |
| 研究一下这个币、这个币怎么样、值得买吗、尽调 | `gmgn-token-research` |
| 深度分析、全面报告、重仓、这个项目能拿多久 | `gmgn-project-deep-report` |
| 风险、危险、巨鲸跑了没、流动性还在吗、要不要跑 | `gmgn-risk-warning` |
| 分析钱包、这个地址怎么样、值不值得跟、战绩如何 | `gmgn-wallet-analysis` |
| 聪明钱画像、交易风格、短线还是长线、跟单收益、排行榜 | `gmgn-smart-money-profile` |

## 推荐流水线

编排技能按用途分为三层，推荐按以下顺序串联：

```
┌─────────────────────────────────────────┐
│  发现层：先找到目标                      │
│  daily-brief / market-opportunities     │
│  / early-project-screening              │
└──────────────┬──────────────────────────┘
               │ 找到感兴趣的代币或钱包
               ▼
┌─────────────────────────────────────────┐
│  研判层：对目标做判断                    │
│  token-due-diligence（快速）            │
│  → token-research（完整）               │
│  → project-deep-report（重仓级）        │
│  wallet-analysis → smart-money-profile  │
│  risk-warning（持有中持续监控）          │
└──────────────┬──────────────────────────┘
               │ 用户确认要交易
               ▼
┌─────────────────────────────────────────┐
│  执行层：仅用户明确确认后才执行           │
│  gmgn-swap（不在 skills-x 中）          │
└─────────────────────────────────────────┘
```

**典型串联路径示例：**

1. **发现 → 快速判断 → 交易**：`daily-brief` → 发现某代币 → `token-due-diligence` → 通过 → `gmgn-swap`
2. **发现 → 深度研究 → 交易**：`market-opportunities` → 选出 top → `token-research` → Buy 判定 → `gmgn-swap`
3. **发现 → 深度研究 → 重仓决策**：`early-project-screening` → 发现潜力新币 → `project-deep-report` → 高分 → 考虑建仓
4. **持仓监控**：持币中 → 定期跑 `risk-warning` → 无风险则继续持有 / 有风险则考虑减仓
5. **钱包跟踪**：`wallet-analysis` → 值得跟随 → `smart-money-profile` → 了解风格和跟单 ROI → 决定是否跟单

## 技能间的依赖关系

编排技能不直接内含 CLI 命令，而是委托给基础技能 (`skills/` 目录)：

| 编排技能 | 依赖的基础技能 |
|---------|--------------|
| `gmgn-daily-brief` | `gmgn-market` + `gmgn-track` + `gmgn-token` |
| `gmgn-market-opportunities` | `gmgn-market` |
| `gmgn-early-project-screening` | `gmgn-market` + `gmgn-token` |
| `gmgn-token-due-diligence` | `gmgn-token` |
| `gmgn-token-research` | `gmgn-token` + `gmgn-market` |
| `gmgn-project-deep-report` | `gmgn-token` + `gmgn-market` |
| `gmgn-risk-warning` | `gmgn-token` + `gmgn-track` + `gmgn-market` |
| `gmgn-wallet-analysis` | `gmgn-portfolio` + `gmgn-track` |
| `gmgn-smart-money-profile` | `gmgn-portfolio` + `gmgn-track` |

因此安装时必须同时安装被依赖的基础技能，编排技能才能正常工作。

## 安装

基础技能和编排技能需分别安装。编排技能当前不在 `npm` 发布包中，需从仓库路径直接安装或软链接：

```bash
# 基础技能（从 npm 包安装）
npx skills add GMGNAI/gmgn-skills

# 编排技能（从仓库源码安装）
openclaw skills install skills-x/gmgn-daily-brief --as gmgn-daily-brief
openclaw skills install skills-x/gmgn-market-opportunities --as gmgn-market-opportunities
# ... 以此类推

# 或批量软链接
for d in skills-x/*(/); do
  ln -sfn "$PWD/$d" "$HOME/.openclaw/skills/${d:t}"
done
```
