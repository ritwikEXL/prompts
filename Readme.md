Rebuild the Opportunities tab to show genuine 
financial intelligence rather than just a ranked 
table. This is the most important change to make 
the product credible to a health plan product manager.

CHANGE 1 — Replace the opportunity radar table 
with a financial opportunity analysis view

For each measure x plan combination calculate 
and display:

Financial impact card showing:
- Current compliance rate (gaps closed / total eligible)
- National benchmark for this measure 
  (use these realistic benchmarks:
   BCS: 74 percent, COL: 68 percent, 
   EED: 72 percent, CDC: 69 percent,
   MAD: 78 percent, AFV: 65 percent, 
   SPC: 80 percent)
- Gap to benchmark (benchmark minus current rate)
- Number of members with open gaps
- Estimated Stars impact of closing all gaps
  (use the corrected formula from earlier)
- CMS bonus payment impact in dollars
  (use this formula: 
   stars_improvement x plan_revenue x 0.05
   where plan_revenue is estimated as:
   P001: 450 million, P002: 380 million,
   P003: 280 million, P004: 520 million,
   P005: 180 million)

Member tier breakdown showing:
Tier 1 High likelihood (members with 
nba_propensity_score above 0.70 AND 
digital_literacy_segment = High):
- Count, recommended channel: digital only
- Cost per member: $2
- Expected closure rate: 60 percent

Tier 2 Medium likelihood (propensity 0.45 to 0.70 
OR digital literacy Medium):
- Count, recommended channel: SMS plus incentive
- Cost per member: $17 (includes $15 gift card)
- Expected closure rate: 35 percent

Tier 3 Low likelihood (propensity below 0.45 
OR digital literacy Low OR language barrier):
- Count, recommended channel: call plus voucher
- Cost per member: $45 (includes $25 voucher)
- Expected closure rate: 18 percent

Return on investment summary showing:
- Total outreach cost (sum across all tiers)
- Total expected closures
- Expected Stars improvement
- Expected CMS bonus payment increase
- Net return (bonus increase minus outreach cost)
- Return on investment ratio

CHANGE 2 — Add a budget optimizer above the table

Add a section at the top called 
"Q3 Outreach Budget Optimizer"

Show a budget slider from $50,000 to $5,000,000
that the PM can drag to set their available budget.

As they drag the slider, automatically recalculate:
- Which opportunities can be fully funded
- Which can only be partially funded
  (e.g. only Tier 1 and Tier 2, not Tier 3)
- Total expected Stars improvement within budget
- Total expected CMS bonus payment increase
- Net return on investment

Show a ranked list of which campaigns to fund 
first to maximize return on investment within 
the budget constraint.

This is a simple greedy allocation — sort 
opportunities by return on investment ratio 
descending, fund them in order until budget 
is exhausted.

CHANGE 3 — Add national benchmark comparison chart

For each measure show a simple horizontal bar 
comparison:
- Gray bar: national average compliance rate
- Teal bar: this plan's current compliance rate
- Difference labeled clearly as the gap to close

This makes it immediately visual that the plan 
is below average on specific measures and by 
how much.

CHANGE 4 — Add a Stars forecast panel

Show three scenarios for end of measurement year:

Scenario 1 — Do nothing
Current Stars trajectory if no outreach is run.
Expected Stars at year end based on natural 
closure rate of 5 percent without intervention.

Scenario 2 — Standard outreach
Digital only, Tier 1 members only.
Expected Stars at year end.
Cost and return on investment.

Scenario 3 — Full optimized campaign
All tiers, all opportunities, full budget.
Expected Stars at year end.
Cost and return on investment.

Show these as three side-by-side cards with 
a clear recommendation of which scenario 
maximizes return on investment.

CHANGE 5 — Add data source indicator

Add a small banner at the top of the 
Opportunities tab saying:
"Currently showing synthetic demo data — 
connect your Snowflake, Databricks, or 
SQL data source to see your real opportunity pipeline"

With a button "Connect Data Source" that for 
now opens a modal showing the data connection 
roadmap — Snowflake, Databricks, BigQuery, 
SQL Server, CSV upload — with a "Coming soon" 
badge on each except CSV upload which shows 
a working upload button.

Keep all existing visual design and teal color 
scheme. This replaces the current opportunity 
radar table entirely.
