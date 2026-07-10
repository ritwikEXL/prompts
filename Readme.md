I'm working on CareIntel, a Medicare Stars Next Best Action platform.
Project folder: C:\Users\vmuser\Documents\NBA_Claude_Cli
Stack: dashboard/index.html (frontend), api.py (FastAPI backend, port 8000),
careintel.db (SQLite).

CONTEXT: We just fixed compliance rate display (now shows 36-58% instead of
0%), Stars improvement capping, and ROI badge display. But a deeper review
surfaced that the underlying synthetic data scale is unrealistic, which is
now the priority fix. Please address these in order — don't skip ahead,
each step depends on the previous one:

STEP 1 — Scale up the synthetic member population
- Current: 100 members across 5 plans x 7 measures produces cohorts of
  1-30 members per plan x measure combination, which is far too small to
  look like a real book of business.
- Fix: regenerate the synthetic dataset in careintel.db with a much larger
  member population (target: several thousand members, e.g. 5,000-10,000)
  distributed realistically across the 5 plans, so each plan x measure
  combination lands in the hundreds or low thousands of eligible members,
  not single/low double digits.
- Keep the existing schema (100 members, 7 measures, 5 plans, gaps table)
  intact — just scale the row counts and update any generation script
  accordingly. Preserve realistic variance in compliance rates per plan
  (don't make them uniform).

STEP 2 — Recalibrate CMS bonus calculations off real PMPM economics
- Current: CMS bonus figures (e.g. $9.5M, $20.8M per single measure-plan
  card) don't reconcile with a ~100-member population and read as
  arbitrary large numbers.
- Fix: rework the bonus calculation to be driven by actual Medicare
  Advantage quality bonus payment methodology — i.e., bonus should scale
  off per-member-per-month (PMPM) revenue x Stars rating bump x member
  count, not a flat number assigned per measure/plan combo. Add a
  configurable PMPM assumption (e.g. as a parameter in the database or
  config file) so it's not hardcoded, and make sure summed bonus across
  all 16 measure-plan cards produces a total that's plausible for a
  several-thousand-member plan, not hundreds of millions from 100 members.

STEP 3 — Fix the budget optimizer so it actually optimizes
- Current: total cost across all "fully funded" campaigns is ~$1,483,
  while the budget slider ranges $50K-$5M — the constraint never binds,
  so dragging the slider does nothing.
- Fix: once Step 1 scales up cohort sizes (and therefore outreach costs
  scale up proportionally), re-test the optimizer at realistic budget
  levels. It should show some campaigns NOT fully funded at lower budget
  levels, with the greedy ROI-ranked allocation actually leaving gaps
  unfunded until the budget increases. Add a visible "X campaigns funded
  of Y, $Z unallocated" or similar indicator showing the trade-off.

STEP 4 — Add real ROI tiering
- Current: every single opportunity card shows "Exceptional ROI" with no
  variation, which makes the label meaningless for prioritization.
- Fix: define ROI tiers (e.g. Marginal, Good, Strong, Exceptional) based
  on actual ROI thresholds computed from the recalibrated bonus and cost
  figures. Make sure the distribution across the 16 measure-plan cards
  shows real variance — not everything landing in the top tier.

STEP 5 — Sanity check Stars improvement variance
- Current: budget optimizer summary shows every plan at exactly +0.250
  Stars improvement, suggesting all plans are hitting the same hardcoded
  cap rather than being data-driven.
- Fix: after Steps 1-2 are done, re-verify the star_weight x 0.15 cap
  logic produces realistic variance across plans based on their actual
  gap profiles, rather than every plan saturating the same value.

After each step, don't move to the next until you show me updated numbers
so I can confirm they look realistic before we continue.

Do NOT touch: Run Session tab, Evaluation tab, WhatsApp/Twilio, SendGrid,
or agent_loop.py — those are out of scope for this fix and should stay
exactly as-is.
