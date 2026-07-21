Fix the financial calculations permanently.
The ROI is showing 3000x+ because tier member 
counts are based on synthetic data (110 members) 
but CMS bonus is based on full plan revenue.
The fix is to scale tier counts to realistic 
eligible population sizes.

Here is the exact correct calculation:

STEP 1 — Get eligible population per measure x plan

From plan_population table get total_members.
Apply eligibility rates to get eligible pool:
BCS: 28%, COL: 42%, EED: 12%, CDC: 32%,
MAD: 12%, AFV: 75%, SPC: 18%

example: EED on P002 (38,000 members):
eligible_pool = 38,000 x 12% = 4,560 members

STEP 2 — Get current compliance from our data

From fact_member_gap count closed vs total
for this measure x plan:

closed = COUNT WHERE gap_status = 'Closed'
total_in_data = COUNT all rows
compliance_rate = closed / total_in_data

example: EED on P002 might show 
68 closed out of 280 total = 24% compliance

STEP 3 — Apply compliance rate to eligible pool

compliant_in_plan = eligible_pool x compliance_rate
open_gaps_realistic = eligible_pool - compliant_in_plan

example: 4,560 x (1 - 0.24) = 3,466 open gaps

STEP 4 — Split open gaps into tiers using 
propensity distribution from our data

From our synthetic data get propensity 
distribution for this measure x plan:
tier1_pct = % of open gaps with propensity > 0.70
tier2_pct = % of open gaps with propensity 0.45-0.70
tier3_pct = % of open gaps with propensity < 0.45

Apply to realistic open gap count:
tier1_count = open_gaps_realistic x tier1_pct
tier2_count = open_gaps_realistic x tier2_pct
tier3_count = open_gaps_realistic x tier3_pct

example: if our data shows 30% T1, 50% T2, 20% T3:
tier1 = 3,466 x 30% = 1,040 members
tier2 = 3,466 x 50% = 1,733 members
tier3 = 3,466 x 20% = 693 members

STEP 5 — Calculate expected closures

expected_closures = 
    (tier1_count x 0.60) +
    (tier2_count x 0.35) +
    (tier3_count x 0.18)

example:
T1: 1,040 x 0.60 = 624
T2: 1,733 x 0.35 = 606
T3: 693 x 0.18 = 125
Total expected closures: 1,355

STEP 6 — Calculate Stars improvement

stars_improvement = min(
    (expected_closures / eligible_pool) 
    x star_weight x 0.5,
    star_weight x 0.10
)

example:
(1,355 / 4,560) x 2 x 0.5 = 0.297
capped at star_weight x 0.10 = 0.20
stars_improvement = 0.20

STEP 7 — Calculate CMS bonus

plan_revenue from plan_population table

cms_bonus = stars_improvement x plan_revenue x 0.05

example:
0.20 x $380,000,000 x 0.05 = $3,800,000

STEP 8 — Calculate outreach cost using 
REALISTIC tier counts not synthetic counts

total_outreach_cost = 
    (tier1_count x 2) +
    (tier2_count x 17) +
    (tier3_count x 45)

example:
(1,040 x 2) + (1,733 x 17) + (693 x 45)
= $2,080 + $29,461 + $31,185
= $62,726

STEP 9 — Net return and display

net_return = cms_bonus - total_outreach_cost
= $3,800,000 - $62,726 = $3,737,274

Instead of ROI ratio show:
"Every $1 invested returns $60 in CMS bonus value"
(net_return / total_outreach_cost = 59.6x)

This is realistic and credible.

STEP 10 — Update all displays

Show realistic tier counts (1,040 / 1,733 / 693)
not synthetic counts (23 / 61 / 26)

Add label under tier breakdown:
"Projected member counts based on estimated 
eligible population of 4,560 members.
Connect Snowflake for plan-specific figures."

Update confidence indicator to use 
eligible_pool not synthetic count:
High: eligible_pool >= 5,000 AND 
      historical data exists
Medium: eligible_pool 1,000 to 4,999
Low: eligible_pool < 1,000

Show: "Medium confidence — estimated 4,560 
eligible members (industry benchmark rates applied)"

EXPECTED RESULTS after fix for EED on P002:
- Eligible pool: 4,560
- Open gaps realistic: ~3,500
- Tier 1: ~1,050 members
- Tier 2: ~1,750 members  
- Tier 3: ~700 members
- Expected closures: ~1,350
- Stars improvement: 0.20
- CMS bonus: $3.8M
- Outreach cost: ~$63K
- Net return: ~$3.7M
- Every $1 returns: ~$59

Run calculation for all 14 measure x plan 
combinations and show me a summary table:

Measure x Plan | Eligible | Open Gaps | 
Expected Closures | Stars | Bonus | Cost | 
Net Return | $1 returns $X

All values in the "$1 returns $X" column 
should be between 5x and 100x.
If any exceed 100x show me why.

Update the dashboard display to show 
realistic tier counts and the 
"Every $1 returns $X" format everywhere.

Remove all instances of ROI ratio numbers 
above 100x from the UI.
