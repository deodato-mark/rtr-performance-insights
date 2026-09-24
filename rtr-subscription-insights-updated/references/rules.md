# Rule library

Thresholds are defined in the CONFIG block of SKILL.md (§1.6). The numbers repeated here are the
current values — if they disagree with CONFIG, CONFIG wins.

**The IDs below are internal.** They exist so this document and SKILL.md can refer to the same rule.
They must never appear in the Slack post — no `G3`, no `I12`, no `Tier C`. Every finding gets stated
in plain language or gets cut. See SKILL.md §1.8 for the translation table.

**Two output rules that apply to every rule in this file:**

- Any finding quoting spend and sign-ups also quotes CAC, whatever the confidence level.
- Everything displayed is rounded to whole numbers; everything computed uses full precision.

---

## Insights

| # | Rule | Basis | Trigger |
|---|---|---|---|
| I1 | Channel CAC vs its own 4-week rolling average | Self | ±20%, Tier A only |
| I2 | Paid CAC and Total CAC movement | Self | ±15% |
| I3 | Best / worst campaigns by CAC within a channel | Peer | Tier A only |
| I4 | Campaign beating its own rolling CAC | Self | −15% |
| I5 | Campaign degrading vs its own rolling CAC | Self | +10% in **each** of 2 consecutive windows |
| I6 | Ad off peer median on CTR / CVR / cost-per-visit | Peer — within campaign **and** format bucket | ±25% |
| I7 | Campaign carrying spend on too few active ads | Count | ≤3 ads above the spend floor |
| I8 | Creative fatigue — CTR falling while CPM rises | Self, 3-week slope | Both sustained |
| I9 | Spend concentration | Share of channel | One campaign >50% |
| I10 | New-subs share shift (`customMetric:7869`) vs 4-week average | Self | 10% **relative** (55% vs 50% triggers) |
| I11 | Sign-up ECR shift (`customMetric:803`) | Self | ±15% |
| I12 | CAC decomposition — CPM vs CTR vs CVR | Self | Runs whenever CAC moves ±20% |

**I12 is what makes recommendations specific.** CAC moving is the symptom; whether it moved on
auction cost, creative engagement, or on-site conversion is the diagnosis, and each routes to a
different owner. Every WHAT CHANGED bullet should carry its I12 driver.

I6 only ever compares ads inside the same channel and the same format bucket — see SKILL.md §4.

---

## Trends

| # | Rule | Window |
|---|---|---|
| T1 | Channel CAC inflation | 3-week trailing vs prior |
| T2 | Cross-channel CAC convergence / divergence | 4-week |
| T3 | Channel rank-order stability by CAC | 8-week |
| T4 | Funnel drift — CPM, CTR, ECR | 4-week |
| T5 | Seasonality-adjusted read vs the same period last year | YoY |
| T6 | New-subs share of total, and subscription share of transactions | 1-week |

**T5 is load-bearing.** RTR is highly seasonal and often consistent YoY. Phase 0: Meta CAC improved
43.3% WoW but was 148.2% worse YoY. Without the YoY lens the weekly read is actively misleading.
Never let a WoW improvement stand as the headline when the YoY read contradicts it.

---

## Recommendations

| # | Type | Requirements |
|---|---|---|
| R1 | Shift budget between campaigns **within** a channel | Both Tier A; donor above channel median CAC, recipient below; recipient shows headroom |
| R2 | Scale | Tier A; beating its rolling average; CAC stable or improving as spend rose |
| R3 | Pause / reduce | 1 window underperforming → **light** suggestion. 2 consecutive windows → **strong** suggestion. Excludes `[NB-CONTROL]` and campaigns live <14 days |
| R4 | Brief more creative | Triggered by I7 or I8 |
| R5 | Fix a specific KPI | From I12 — ECR → LP / offer / audience; CTR → creative; CPM → bid / placement |
| R6 | Monitor | Tier B, stated with the condition that would promote it to Tier A |
| R7 | Test candidate | Standing section — surface as many testing opportunities as the data supports |

Budget recommendations are **intra-channel reallocation only**, judged on CAC against rolling
averages. There is no external budget file; the rolling averages are the targets.

**Ranking:** dollars at stake (spend governed × size of the CAC gap), then confidence.

**Check R1–R7 against account context (SKILL.md §5.1) before issuing any of them.** If the team or
the client has already directed something contrary — pause a campaign this rule wants to scale, hold
a budget this rule wants to shift — the finding still gets stated, but as a conflict to resolve on
the call, not as an action. Name the direction and its source.

Every recommendation carries: the evidence, the confidence level in plain words, and the owner as a
`<@Uxxxxx>` tag.
Tag only the owners whose channel is implicated — not the full roster on every post. Mark
(`<@U01T0TNUHBK>`) is tagged on cross-channel and budget items only.

RTR is open to testing — both new channels and tests within existing channels. R7 should not be
empty on a Monday run unless the data genuinely supports nothing.

---

## Integrity gates

| Gate | Check | Action |
|---|---|---|
| G1 | Spend drops to $0 from >$0 in the prior window | Dedicated **VALIDATE** item — ask for human confirmation it was intentional. Never suppress |
| G2 | Spend = $0 but visits or sign-ups > 0 | Absolute figures only, no rate; note in VALIDATE |
| G3 | Total paid spend swings >40% vs the trailing 4-week average | Flag as a possible data issue **before** reading it as a trend |
| G4 | Campaign live <14 days | Insights yes, pause recommendations no |
| G5 | CAC more than 5x better than the channel's next-best Tier A campaign | Flag for validation, never auto-scale |

G5 exists because of Google PMAX: `Pros_PMAX_X_BOF_Purchase_NCBidOnlyBid` read $4,829.96 /
174.80 sign-ups = CAC $27.63 over 14 days, roughly 7x better than the next Google campaign.
Implausible enough that scaling logic must not trust it. G5 currently quarantines it.

G1 and G3 are the two most likely to fire on a run where something upstream broke. When G3 fires,
say so in the lede — do not narrate a 40% spend swing as strategy.
