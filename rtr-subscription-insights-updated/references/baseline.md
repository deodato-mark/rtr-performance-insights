# Phase 0 baseline and open items

## Baseline — Jul 27 to Aug 2, 2026

Exact values from Northbeam, Cash + Clicks + Modeled Views. Use these to sanity-check a run: if a
channel comes back at a wildly different order of magnitude, suspect the query before believing the
finding.

| Channel | Spend | Sign-ups | CAC |
|---|---|---|---|
| Google | $103,250.75 | 450.27 | $229.31 |
| Meta | $64,957.51 | 183.24 | $354.49 |
| Pinterest | $21,170.28 | 28.21 | $750.52 |
| TikTok | $11,248.00 | 7.82 | $1,438.43 |
| YouTube | $0.00 | 6.00 | n/a |
| **Paid (all)** | $216,875.58 | 1,375.51 | **$157.67** |
| **Total** | $216,875.58 | 3,946.00 | **$54.96** |

YouTube at $0.00 spend with 6.00 sign-ups is a live G2 case — absolute figures only, note in
VALIDATE.

Tier distribution at these thresholds: of 21 qualifying campaigns, 10 land in Tier A and 7 in
Tier C. The Tier C set is exactly the group that would otherwise dominate a worst-CAC list with
meaningless six-figure rates.

Phase 0 test post: https://deodatoco.slack.com/archives/C0ALLSTV6CD/p1785868094118119

---

## Open items for Mark / RTR

Carry these forward. They are not blocking the engine, but several change what it can do.

1. **Request membership-funnel custom goals in NBM.** Ask Matt Logan for 1–2 URL-rule custom goals
   (e.g. Subscription Checkout Start, Membership Plan Select). This is the real fix for ad-level
   measurement and would let the engine drop C+DV entirely. Context: the dashboard has no custom
   goals, and Meta has only two custom conversions (`Search Lift Organic`, `Search Lift Paid`) —
   neither is a funnel step. Add-to-cart is the wrong proxy for RTR subs because sign-up is a
   membership flow, so ATC volume mostly reflects Reserve/KIF browsing.

2. **Google PMAX attribution.** `Pros_PMAX_X_BOF_Purchase_NCBidOnlyBid` read $4,829.96 / 174.80
   sign-ups = CAC $27.63 over 14 days, ~7x better than the next Google campaign. Validate before any
   scaling logic trusts it. Gate G5 currently quarantines it.

3. **Pinterest and TikTok efficiency.** Ten weeks to Aug 2: Pinterest $457,153.52 → 297.74 sign-ups
   ($1,535.40 CAC); TikTok $197,471.50 → 71.90 ($2,746.48). Both are view-heavy so C+DV reads far
   better, and neither has been checked against MMM or incrementality — but at that spend it is not
   small-sample noise.

4. **The incrementality question Phase 0 raised.** Paid spend fell 51.4% WoW while total sign-ups
   rose 2.3%. One week, with a confirmed Meta outage in it, so not proof of anything — but if it
   repeats it is the most important thing in the account.

5. **Meta YoY deterioration.** CAC $354.49 vs $142.80 the same week last year; sign-ups −66.1% on
   only 15.8% less spend. Structural, not noise, given RTR's YoY consistency.

---

## Rollout status

| Phase | What | Status |
|---|---|---|
| 0 | Dry run against live data, tune thresholds, test post to `C0ALLSTV6CD` | Done |
| 1 | Monday + Thursday 6:00 AM ET to `C0ALLSTV6CD` | Skipped — Mark promoted straight to production Aug 4, 2026 |
| 2 | `rtr_internal` `C08T29DB7AA`, `<@Uxxxx>` tagging on | **Current — LIVE** |
| 3 | Optional client-facing version | Later |

Because Phase 1 was skipped, thresholds have only been validated against the single Phase 0 window.
Watch the first few production runs for false positives and tune §1.6 of SKILL.md rather than
assuming the numbers are settled.

**Scheduling note.** The routine must be created with `mcp__claude-code-remote__create_trigger`, not
`CronCreate` — the local cron tools run an in-process scheduler that dies with the session, so the
routine would silently never fire. Cron is evaluated in **UTC**: 6:00 AM ET during EDT is
`0 10 * * 1,4`. After **Nov 1, 2026** ET is UTC−5 and it needs to become `0 11 * * 1,4`.
