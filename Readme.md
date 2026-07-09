Replace all hardcoded values in api.py and 
dashboard/index.html with values computed 
dynamically from the data in careintel.db.

REPLACE 1 — Eligibility rates
Do not hardcode eligibility rates per measure.
Instead compute eligible member count directly:

For each measure x plan combination count the 
members who appear in fact_member_gap for that 
measure and plan. That IS the eligible population.
Do not estimate it from total members x a rate.

eligible_members = SELECT COUNT(DISTINCT member_key) 
FROM fact_member_gap 
WHERE measure_key = X AND plan_key = Y

compliant_members = SELECT COUNT(DISTINCT member_key)
FROM fact_member_gap
WHERE measure_key = X AND plan_key = Y
AND gap_status = 'Closed'

compliance_rate = compliant_members / eligible_members

REPLACE 2 — Plan revenue
Do not hardcode plan revenue.
Add a plan_annual_revenue column to dim_plan_contract 
with these initial values that the user can update:
P001: 450000000
P002: 380000000
P003: 280000000
P004: 520000000
P005: 180000000

But read it from the database not from hardcoded 
Python or JavaScript. If a new plan is added with 
no revenue set, show a prompt asking the user to 
enter the plan revenue before financial analysis 
can run.

REPLACE 3 — National benchmarks
Store national benchmarks in a new database table:

CREATE TABLE IF NOT EXISTS measure_benchmarks (
    measure_key TEXT,
    benchmark_year INTEGER,
    national_avg_rate REAL,
    top_quartile_rate REAL,
    bottom_quartile_rate REAL,
    source TEXT,
    last_updated TEXT
)

Seed with current values:
M001 BCS: national 74, top 82, bottom 65
M002 COL: national 68, top 78, bottom 58
M003 EED: national 72, top 81, bottom 62
M004 CDC: national 69, top 79, bottom 58
M005 MAD: national 78, top 86, bottom 70
M006 AFV: national 65, top 75, bottom 55
M007 SPC: national 80, top 88, bottom 72

Read from this table in the API. 
Add a GET /benchmarks endpoint that returns 
all benchmarks.
Add a PUT /benchmarks/{measure_key} endpoint 
that lets the user update benchmarks when 
NCQA publishes new ones each year.

REPLACE 4 — Cost per member per tier
Store outreach costs in a new table:

CREATE TABLE IF NOT EXISTS outreach_costs (
    cost_id TEXT PRIMARY KEY,
    tier INTEGER,
    channel TEXT,
    base_cost REAL,
    incentive_amount REAL,
    total_cost REAL,
    last_updated TEXT
)

Seed with:
Tier 1, EMAIL, base 1.50, incentive 0, total 1.50
Tier 1, SMS, base 0.50, incentive 0, total 0.50
Tier 2, SMS, base 0.50, incentive 15, total 15.50
Tier 2, EMAIL, base 1.50, incentive 15, total 16.50
Tier 3, CALL, base 8.00, incentive 25, total 33.00
Tier 3, WHATSAPP, base 0.10, incentive 25, total 25.10

Add a GET /costs endpoint and a settings page 
where the user can update these costs.

REPLACE 5 — Expected closure rates per tier
Store these in a new table too:

CREATE TABLE IF NOT EXISTS closure_rate_assumptions (
    measure_key TEXT,
    tier INTEGER,
    expected_rate REAL,
    basis TEXT,
    last_updated TEXT
)

Seed with current values but read from database.
When historical outreach data exists for this 
measure and plan, override the assumed rate with 
the actual historical rate computed from past runs:

historical_rate = closed_gaps_from_outreach / 
                  total_members_outreached

If historical_rate exists use it.
If not use the assumed rate from the table.
Show in the UI which rate is being used and why.

After all changes restart uvicorn and show me 
the financial analysis for EED x P002 using 
computed values — confirm compliance rate, 
gap count, Stars impact, CMS bonus, and ROI 
are all derived from database values not 
hardcoded numbers.
