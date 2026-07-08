Fix the Stars impact calculation in api.py — 
it is showing completely unrealistic values 
like 34.62 actual and 7.5 projected which 
would immediately discredit the demo.

Here is how Stars impact should be calculated:

CORRECT FORMULA:

Stars impact per gap closed = 
(1 / total_eligible_members_for_measure_on_plan) 
x star_weight x 0.5

Where:
- total_eligible_members = denominator size for 
  that measure on that plan. Since we do not have 
  real denominator data, estimate it as:
  total members on that plan x eligibility_rate
  
  Use these eligibility rates per measure:
  BCS: 30 percent of members (women 50-74)
  COL: 45 percent of members (adults 45-75)
  EED: 15 percent of members (diabetics 18-75)
  CDC: 35 percent of members (hypertension)
  MAD: 15 percent of members (diabetics on meds)
  AFV: 80 percent of members (most members)
  SPC: 20 percent of members (cardiovascular)

- star_weight = from dim_measure
- 0.5 = each measure can move Stars by up to 
  0.5 in the best case scenario

So for example:
Plan P002 has roughly 500 members total
EED eligibility = 15 percent = 75 eligible members
Closing 5 EED gaps = 5/75 x 2 x 0.5 = 0.067 stars

IMPLEMENTATION:

In POST /evaluate/{run_id} calculate Stars impact as:

For each gap closed in this campaign:
stars_per_gap = (1 / estimated_denominator) 
                x star_weight x 0.5

stars_impact_actual = sum of stars_per_gap 
for all actually closed gaps

stars_impact_projected = sum of stars_per_gap 
for all gaps that would close if campaign 
hits expected closure rate

Both values should be between 0.01 and 0.50 
for any realistic single campaign.
Cap at 0.50 as absolute maximum.

Also fix the portfolio summary in the 
Evaluation tab header:
STARS IMPACT ACTUAL should show total 
projected Stars improvement across all 
evaluated campaigns — this should be 
a number between 0.1 and 1.0 for a 
realistic portfolio, never above 1.5

Update all existing evaluation records 
with corrected Stars impact values by 
re-running POST /evaluate/run-scheduled

Show me the before and after values for 
each campaign to confirm the fix worked.
