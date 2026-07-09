Fix three critical calculation errors in the 
financial intelligence layer that are producing 
unrealistic numbers:

FIX 1 — Compliance rate calculation is wrong

Current compliance rate should be calculated as:
compliant_members / total_eligible_members

Where:
- compliant_members = members whose gap_status 
  is Closed for this measure on this plan
- total_eligible_members = total members on this 
  plan multiplied by the eligibility rate for 
  this measure

Use these eligibility rates:
BCS: 30 percent, COL: 45 percent, EED: 15 percent,
CDC: 35 percent, MAD: 15 percent, AFV: 80 percent,
SPC: 20 percent

And these total member counts per plan:
P001: 1200 members, P002: 950 members,
P003: 800 members, P004: 1400 members,
P005: 600 members

So for example:
EED compliance on P002:
- Total eligible = 950 x 15 percent = 143 members
- Compliant = members with Closed EED gaps on P002
- Compliance rate = compliant / 143

The benchmark gap should never exceed 40 percentage 
points. If it does the calculation is wrong.

FIX 2 — Return on investment calculation is wrong

The CMS bonus payment impact must be calculated 
per plan separately, not across the whole portfolio.

For a single measure x plan opportunity:
stars_improvement = (expected_closures / total_eligible) 
                    x star_weight x 0.5

cap stars_improvement at 0.15 per single campaign
(no single campaign can move Stars more than 0.15)

cms_bonus_impact = stars_improvement x plan_revenue x 0.05

For example EED on P002:
- Expected closures: 18 members
- Total eligible: 143 members  
- Stars improvement: (18/143) x 2 x 0.5 = 0.126
- Cap at 0.15 — so 0.126 (under cap, use as is)
- CMS bonus: 0.126 x 380,000,000 x 0.05 = $2.39 million

ROI ratio = cms_bonus_impact / total_outreach_cost
This should typically be between 50 and 500.
If ROI exceeds 1000 something is wrong — 
add a warning in console and cap display at 999x.

FIX 3 — Portfolio Stars improvement is too high

The budget optimizer portfolio summary shows 
plus 0.92 Stars improvement. This is wrong 
because it is summing Stars improvements across 
all plans as if they are one plan.

Fix: show Stars improvement separately per plan.
In the budget optimizer show:
P001: expected +0.08 stars
P002: expected +0.12 stars
P003: expected +0.15 stars (highest gap)
P004: expected +0.06 stars
P005: expected +0.18 stars (lowest current rating)

Portfolio summary shows the average improvement 
across plans, not the sum. This should be 
between 0.05 and 0.25 for a realistic quarter.

FIX 4 — Stars forecast scoping

The Stars forecast panel should show results 
for one selected plan at a time, not the 
whole portfolio mixed together.

Add a plan selector dropdown above the 
Stars forecast panel. Default to the plan 
with the lowest current Stars rating (P005).

Show the three scenarios for that specific plan:
- Current Stars: 2.5 (for P005)
- Do nothing end of year: 2.52
- Digital first: 2.58  
- Full campaign: 2.68

These numbers should be current Stars plus 
the incremental improvement from each scenario.
Never show year-end Stars above current Stars 
plus 0.3 for a single year of outreach.

After fixing restart uvicorn and show me 
sample numbers for EED x P002 to verify:
- Compliance rate (should be 60 to 80 percent)
- Gap to benchmark (should be 2 to 15 percentage points)
- Stars improvement (should be 0.05 to 0.15)
- CMS bonus impact (should be $1M to $5M)
- ROI ratio (should be 50 to 500)
