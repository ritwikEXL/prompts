Fix the following critical issues in the 
Opportunities tab before we proceed to 
the agentic loop:

FIX 1 — Add realistic closed gaps to synthetic data

The database currently has almost no closed gaps
which makes compliance rate show as 0 percent 
everywhere. This looks wrong.

Run this SQL to close a realistic proportion 
of gaps to simulate a real plan's current 
performance:

For each measure use these target compliance rates
that reflect a struggling but not failing plan:
BCS: close gaps until compliance reaches 55 percent
COL: close gaps until compliance reaches 45 percent  
EED: close gaps until compliance reaches 52 percent
CDC: close gaps until compliance reaches 42 percent
MAD: close gaps until compliance reaches 48 percent
AFV: close gaps until compliance reaches 50 percent
SPC: close gaps until compliance reaches 58 percent

For each measure x plan combination calculate
how many gaps need to be Closed to reach the
target rate. Randomly select that many gap rows
and update gap_status to Closed.

This will make compliance rates look realistic
— below the national average but not at zero.

FIX 2 — Fix the Stars improvement cap

The 0.5000 cap is too aggressive — it is hitting
on almost every opportunity making them all look
equal. 

Replace the flat cap with a proportional formula:

stars_improvement = min(
    (expected_closures / total_eligible) 
    x star_weight x 0.5,
    star_weight x 0.15
)

The new cap is star_weight x 0.15 per campaign
So EED (weight 2) caps at 0.30 Stars
BCS (weight 1) caps at 0.15 Stars
This creates meaningful differentiation between
high-weight and low-weight measure opportunities.

FIX 3 — Fix the ROI ratio display

The 999x cap exists because outreach costs are
tiny compared to CMS bonus values. This is
mathematically correct but misleading in display.

Change the ROI display logic:
- Never show a ratio above 500x
- Instead of showing the ratio for very high ROI
  opportunities, show the label "Exceptional ROI"
  with a teal badge
- Show the actual dollar figures prominently:
  Cost: $513 → Bonus: $2.6M
  That tells the story better than 999x

Also reconsider whether the CMS bonus calculation
is realistic. The formula is:
stars_improvement x plan_revenue x 0.05

For a small campaign improving Stars by 0.116
on a $450M plan: 0.116 x 450M x 0.05 = $2.6M

This is actually mathematically correct — a 
0.1 Stars improvement on a large plan IS worth
millions in bonus payments. The issue is the
outreach cost is too low relative to the impact.

So instead of capping the ratio, add a 
confidence indicator:
- ROI above 200x: show "High confidence ROI"
  only if plan has 500+ eligible members
- ROI above 200x with fewer than 50 eligible
  members: show "Small cohort — verify with 
  full plan data" warning badge

FIX 4 — Fix budget optimizer totals

The $33.7M total is the sum of CMS bonus across
all 14 campaigns on all 5 plans. This is wrong
because:
1. CMS pays bonus per plan contract separately
2. A plan cannot improve by 0.5 Stars on every
   measure simultaneously in one quarter
3. The optimizer should show portfolio-level
   impact conservatively

Fix: cap total portfolio Stars improvement at
0.25 per plan per quarter in the optimizer.
Total CMS bonus = sum across plans of 
min(total_stars_improvement, 0.25) x plan_revenue x 0.05

This will give a more realistic total of 
$3M to $8M across the portfolio — still compelling
but believable.

FIX 5 — Add plan filter persistence to 
benchmark chart and financial cards

When a user selects a specific plan in the 
Plan filter dropdown, ALL sections should 
filter to that plan:
- Benchmark chart shows only that plan's 
  compliance rates
- Financial cards show only that plan's 
  opportunities
- Stars forecast shows that plan specifically
- Budget optimizer shows budget for that plan only

Currently the filters only affect the cards
but not the benchmark chart or Stars forecast.

After all fixes restart uvicorn and show me
sample numbers for BCS x Aurora MA-PD Choice:
- Plan compliance rate (should be around 55%)
- Gap to benchmark (should be around 19pp)
- Stars improvement capped (should be around 0.12)
- CMS bonus (should be around $2.7M)
- ROI display (should show Exceptional ROI badge
  not 999x)
