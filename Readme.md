Fix the following issues in the redesigned 
Opportunities tab:

FIX 1 — Open gaps and members at risk counts are wrong

The plan summary banner is showing 2006 open gaps 
and 2047 members at risk for MA Value (P004). 
This is impossible — we only have 250 total gap 
rows in the entire database.

Fix the query for open gaps count to:
SELECT COUNT(*) FROM fact_member_gap 
WHERE plan_key = selected_plan 
AND gap_status IN ('Open', 'Borderline', 'Partial')
AND is_suppressed = 0

Fix the members at risk count to:
SELECT COUNT(DISTINCT member_key) FROM fact_member_gap
WHERE plan_key = selected_plan
AND gap_status IN ('Open', 'Borderline', 'Partial')
AND is_suppressed = 0

Show me the corrected counts for all 5 plans 
after fixing.

FIX 2 — Measures below benchmark count is wrong

A plan at 4.5 stars should not show 7 out of 7 
measures below national average. The below benchmark 
check must filter by plan_key not across all plans.

Fix the query to check each measure's compliance 
rate specifically for the selected plan against 
the national benchmark in measure_benchmarks table.

A measure is below benchmark only if:
(closed gaps for this plan on this measure / 
 eligible members for this plan on this measure) 
< national_avg_rate from measure_benchmarks

FIX 3 — Default plan should be lowest Stars rating

When the Opportunities tab loads, automatically 
select the plan with the lowest star_rating_current 
from dim_plan_contract as the default.

This is P005 Aurora MA-PD Signature at 2.5 stars.
This is the most urgent plan and most compelling 
demo story — not MA Value at 4.5 stars.

FIX 4 — Launch Campaign button must connect to 
Run Session tab

When the PM clicks "Launch This Campaign" on any 
opportunity card:
1. Store the selected measure_key and plan_key 
   in session state
2. Switch to the Run Session tab automatically
3. Pre-populate the opportunity selection with 
   the chosen measure and plan
4. Show a banner at the top of Run Session saying:
   "Launching campaign for [measure] on [plan] — 
   pre-selected from Opportunities tab"
5. Start Phase 1 of the session automatically

This is the critical connection between the 
Opportunities tab and the Run Session tab.

FIX 5 — Add scope labels to all financial figures

Next to every dollar figure add a small gray 
parenthetical scope label:

- CMS bonus at risk: "$9M/year (this plan, 
  all measures combined)"
- Net return on opportunity card: "$325K 
  (this measure only, realistic estimate)"  
- Max CMS bonus on opportunity card: "$893K 
  (if 100% of gaps closed, theoretical maximum)"
- Every $1 spent figure: "$188 return per $1 
  (estimated, based on industry average response rates)"
- Budget planner totals: "(full portfolio, 
  all funded campaigns)"

FIX 6 — ROI should vary by plan Stars rating

A plan at 4.5 stars has less Stars upside than 
a plan at 2.5 stars. The ROI calculation should 
reflect this by adjusting the CMS bonus impact:

stars_upside_factor = (5.0 - star_rating_current) / 2.5

Apply this factor to the CMS bonus calculation:
cms_bonus_impact = stars_improvement 
                   x plan_revenue 
                   x 0.05 
                   x stars_upside_factor

This means:
P005 at 2.5 stars: factor = 1.0 (full bonus potential)
P001 at 3.5 stars: factor = 0.6 (less upside)
P004 at 4.5 stars: factor = 0.2 (minimal upside)

This makes the dashboard correctly show that 
fixing gaps on a struggling plan is worth more 
than fixing gaps on an already-strong plan.

After all fixes show me:
1. Corrected open gaps and members at risk 
   for all 5 plans
2. Section 1 banner for P005 (2.5 stars) 
   with all corrected numbers
3. Top opportunity card for P005 with 
   corrected ROI using stars_upside_factor
4. Confirm Launch Campaign button navigates 
   to Run Session tab
