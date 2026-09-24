# rtr-performance-insights

Working copy of changes to the `rtr-subscription-insights` skill — the routine that
produces the twice-weekly RTR subscription performance post in `#rtr_internal`.

## `rtr-subscription-insights-updated/`

The skill files with Mark's Sep 24, 2026 feedback applied. **These are not live.**
Skills are managed in claude.ai settings; the copy a session sees is synced from the
account and gets overwritten on the next sync, so edits made during a session do not
persist. To make these changes take effect, paste the contents into the skill in
claude.ai settings.

Changed files: `SKILL.md`, `references/rules.md`. `references/baseline.md` and
`references/northbeam-queries.md` are unchanged and included only so the directory is
a complete copy.

## What changed

1. **No internal rule codes in Slack output.** `G3`, `I12`, `Tier C` and the rest are
   scaffolding for the skill doc, not language for a channel post. Every finding is
   stated in plain words or cut. `SKILL.md` §1.8 carries a translation table.
2. **The scorecard always shows every Δ% column for every channel on it**, whatever
   its confidence level. A blank cell reads as missing data. Low confidence changes how
   a channel is described in the commentary, not whether its numbers appear.
3. **Anything quoting spend and sign-ups also quotes CAC**, so nobody divides two
   numbers by hand to reach the metric the account is managed on.
4. **Channels with $0 spend in both the current and comparison window are dropped from
   the scorecard**, and named in CAVEATS. Two exceptions, because both are findings
   rather than absence: spend that fell to $0 this window, and $0 spend carrying real
   sign-ups.
5. **Everything displays as whole numbers.** This overrides the skill's previous
   "never round" rule and the Northbeam connector's own guidance, so it is fenced:
   all arithmetic runs at full precision and rounds once at write time, a sign-up count
   that would round to 0 is written "under 1" instead, and CAVEATS states the
   convention. Northbeam's underlying values stay exact and fractional — multi-touch
   attribution splits credit across touchpoints.
6. **Account context is gathered before recommendations are written** (`SKILL.md` §5.1).
   The routine reads recent `#rtr_internal` and `#external-rtr-deo` messages and Fathom
   recaps for direction already given, so it does not recommend scaling something the
   team agreed to pause. Context never deletes a finding — it reframes it, and the
   conflict gets stated with its source.
