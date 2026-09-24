# Northbeam query recipes and gotchas

Dashboard: `83bbcd7b-4684-456d-9506-ac1aab4ed707` (Rent The Runway — the only one).

---

## Gotchas

1. `list_metrics` takes no real args, but `list_breakdowns`, `list_custom_metrics`,
   `list_custom_goals`, `list_attribution_models` and `list_attribution_windows` all require
   `dashboard_id` **and** `user_prompt`.
2. **`time_granularity` does not create per-period rows by itself.** Setting `weekly` without `date`
   in `dimension_ids` returns one aggregate row per dimension for the whole range. Add `date` if you
   need a time series.
3. **Cash mode forces an infinite attribution window.** Do not pass `attribution_window` alongside
   `accounting_mode: cash`.
4. `customMetric:806` returns **`null`**, not 0, when sign-ups are 0. Handle explicitly.
5. Any dimension used in `dimension_filters` must also appear in `dimension_ids`.
6. Use `compare_date_range` — it returns both periods in one call and halves query volume.
7. Sign-up counts are **fractional by design** (MTA distributes credit across touchpoints). Never
   round. Report exact values.
8. **Do NOT filter on `breakdown:Business Lines` to scope subscriptions.** It covers only ~29% of
   paid spend and the untagged bucket holds the overwhelming majority of sign-ups. Use the
   subscription *metrics* across all paid spend instead — they are tag-independent.
9. The `status` dimension returns clean `active` / `inactive`, which is how the paused-2-weeks rule
   works.
10. There are **zero custom goals** on this dashboard. `list_custom_goals` returns empty.

---

## Working query shapes

### Channel scorecard, week over week

```
dimension_ids:    ["breakdown:Platform (Northbeam)"]
dimension_filters: [{dimension_id: "breakdown:Platform (Northbeam)",
                     values: ["Facebook Ads","Google Ads","Pinterest","TikTok","YouTube Ads"]}]
level:            "campaign"
time_granularity: "weekly"
accounting_mode:  "cash"
attribution_model: "northbeam_custom__va"
date_range + compare_date_range
```

### Total / Paid split

```
dimension_ids: ["breakdown:High Level Buckets"]   // Paid / Owned / Earned / (not set)
```

Sum **all** buckets for total CAC. Do not use `customMetric:11014`.

### Campaign level with paused detection

```
dimension_ids: ["campaignName","breakdown:Platform (Northbeam)","status"]
sorting:       [{metric_id: "spend", order: "desc"}]
limit:         100
```

### Meta ad level, format-bucketed, C+DV

```
dimension_ids:     ["adName","breakdown:Meta - Format","breakdown:Platform (Northbeam)"]
dimension_filters: [{dimension_id: "breakdown:Platform (Northbeam)", values: ["Facebook Ads"]}]
level:             "ad"
attribution_model: "northbeam_custom__enh"
date_range:        28 days
```

C+DV output is for within-channel, within-format relative ranking only. It never enters the
scorecard or any cross-channel statement.

### Landing page rollup

Parse from the ad-name convention — `...ValueLP`, `...ConvenienceLP`, `...HowItWorksLP`,
`...MostHeartedGridLP`. `breakdown:FB: Ad Landing Page` is also available. Keep this; the LP rollup
produced the single most actionable Phase 0 finding.

---

## Reference: C+DV vs C+MV volume multiples

Trailing 14d to Aug 2, 2026, spend and visits held constant, only the attribution model changed:

| Channel | C+MV sign-ups | C+DV sign-ups | Multiple |
|---|---|---|---|
| Meta | 470.37 | 1,279.51 | 2.72x |
| Google | 905.25 | 716.40 | 0.79x |
| Pinterest | 60.89 | 859.86 | 14.12x |
| TikTok | 15.43 | 191.35 | 12.40x |

This is why C+DV is contained to within-channel ad ranking. Google is the one channel where C+DV is
worse, and Google runs ad-level on a monthly cadence where a 28-day C+MV window gives enough volume
anyway.
