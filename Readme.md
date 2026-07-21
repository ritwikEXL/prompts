Do two things in one prompt — fix the financial 
model by adding realistic data volume, then build 
the data source connector.

PART 1 — Fix financial model by adding real data volume

The core problem is our synthetic dataset is too small.
We have 5 to 30 gaps per measure per plan but our 
plan revenues assume 18,000 to 52,000 member plans.
Fix this by adding realistic gap volumes directly 
to the database.

STEP 1A — Add realistic gap volumes to fact_member_gap

First check current gap counts per plan per measure:
SELECT plan_key, measure_key, COUNT(*) as gaps,
COUNT(CASE WHEN gap_status IN 
('Open','Borderline','Partial') THEN 1 END) as open
FROM fact_member_gap
GROUP BY plan_key, measure_key
ORDER BY plan_key, measure_key

Then add synthetic gap rows to bring counts 
to realistic levels based on plan size:

Target open gap counts per measure per plan:

P001 Aetna Medicare Choice (45,000 members):
BCS: 800 open gaps (eligible: 12,600, compliance 94% closed → 6% open)
COL: 1,200 open gaps (eligible: 18,900, compliance 94% closed)
EED: 350 open gaps (eligible: 5,400, compliance 94% closed)
CDC: 900 open gaps (eligible: 14,400, compliance 94% closed)
MAD: 280 open gaps (eligible: 5,400, compliance 95% closed)
AFV: 1,600 open gaps (eligible: 33,750, compliance 95% closed)
SPC: 400 open gaps (eligible: 8,100, compliance 95% closed)

P002 Aetna Medicare Premier (38,000 members):
BCS: 650 open gaps
COL: 950 open gaps
EED: 280 open gaps
CDC: 720 open gaps
MAD: 220 open gaps
AFV: 1,300 open gaps
SPC: 320 open gaps

P003 Aetna Medicare DSNP Community (28,000 members):
BCS: 520 open gaps
COL: 780 open gaps
EED: 230 open gaps
CDC: 620 open gaps
MAD: 190 open gaps
AFV: 980 open gaps
SPC: 260 open gaps

P004 UHC Medicare Advantage Value (52,000 members):
BCS: 900 open gaps
COL: 1,400 open gaps
EED: 420 open gaps
CDC: 1,050 open gaps
MAD: 330 open gaps
AFV: 1,900 open gaps
SPC: 480 open gaps

P005 UHC Medicare Signature PPO (18,000 members):
BCS: 320 open gaps
COL: 480 open gaps
EED: 140 open gaps
CDC: 360 open gaps
MAD: 110 open gaps
AFV: 650 open gaps
SPC: 170 open gaps

For each target count check how many gaps 
already exist for that measure x plan combination.
Add only the difference needed to reach the target.

When generating new gap rows:
- Use member keys from dim_member cycling 
  through all 250 members repeatedly with 
  unique member_gap_key identifiers
- Use format G_P001_BCS_00001 etc for new keys
- gap_status: 85% Open, 10% Borderline, 5% Partial
- gap_open_date: random between 2026-01-01 and 2026-04-01
- days_open: calculated from gap_open_date to today
- clinical_risk_score: random between 0.35 and 0.85
- nba_propensity_score: random between 0.30 and 0.80
- previous_year_gap_flag: true for 25% of new gaps
- upstream_recommended_channel: distribute 
  based on measure type:
  MAD, AFV: 60% EMAIL, 30% SMS, 10% CALL
  BCS, COL: 40% EMAIL, 35% SMS, 25% CALL
  EED, CDC, SPC: 30% EMAIL, 30% SMS, 40% CALL
- upstream_recommended_incentive:
  GIFTCARD_15 for 40% of gaps
  GIFTCARD_25 for 35% of gaps
  TRANSPORT_VOUCHER for 15% of gaps
  FIT_KIT_MAILER for COL measure gaps (10%)
- is_suppressed: false for all new gaps
- measurement_year: 2026

STEP 1B — Add plan size reference table

CREATE TABLE IF NOT EXISTS plan_population (
    plan_key TEXT PRIMARY KEY,
    total_members INTEGER,
    plan_revenue INTEGER,
    last_updated TEXT
)

INSERT values:
P001: 45000 members, $450,000,000 revenue
P002: 38000 members, $380,000,000 revenue
P003: 28000 members, $280,000,000 revenue
P004: 52000 members, $520,000,000 revenue
P005: 18000 members, $180,000,000 revenue

STEP 1C — Update financial calculations in api.py

Update GET /opportunities to use real gap counts 
from fact_member_gap directly — no scale factors needed 
since we now have realistic data volumes.

Use plan_revenue from plan_population table 
instead of hardcoded values.

Stars improvement formula:
eligible_members = total gap rows for this 
measure x plan (all statuses)

stars_improvement = min(
    (expected_closures / eligible_members)
    x star_weight x 0.5,
    star_weight x 0.10
)

cms_bonus = stars_improvement x plan_revenue x 0.05

total_outreach_cost =
    (tier1_count x 2) +
    (tier2_count x 17) +
    (tier3_count x 45)

net_return = cms_bonus - total_outreach_cost
roi_ratio = net_return / total_outreach_cost

Expected ROI range after this fix: 5x to 80x
If above 100x add note: "Verify with full plan data"

STEP 1D — Update member tier breakdown

With realistic gap volumes the tier counts 
should now be meaningful:
Tier 1 (propensity above 0.70, high digital): 
  roughly 15 to 25% of open gaps
Tier 2 (propensity 0.45 to 0.70): 
  roughly 45 to 55% of open gaps
Tier 3 (propensity below 0.45 or low digital): 
  roughly 20 to 30% of open gaps

Show member tier breakdown using actual 
counts from the new realistic data.

STEP 1E — Verify financial calculations

After adding data show me sample calculation 
for EED on P002 Aetna Medicare Premier:
- Total eligible members (EED rows on P002)
- Open gaps count
- Tier 1 count, Tier 2 count, Tier 3 count
- Expected closures
- Stars improvement
- CMS bonus
- Total outreach cost
- Net return
- ROI ratio

This should show realistic numbers like:
- 280 open EED gaps
- Tier 1: 60 members, Tier 2: 130 members, 
  Tier 3: 90 members
- Expected closures: 85 members
- Stars improvement: 0.018
- CMS bonus: $342,000
- Outreach cost: $8,670
- Net return: $333,330
- ROI: 38x

PART 2 — Build data source connector

Add a new Data Sources tab to dashboard/index.html
between the CareIntel logo and Opportunities tab.

SECTION A — Current data status panel

Show a clean status table:
Data Source    | Records | Status  | Last Updated
Members        | 250     | Demo    | [date]
Care Gaps      | [new]   | Demo    | [date]  
Plans          | 5       | Demo    | [date]
Run History    | 4       | Demo    | [date]

Below the table:
Banner in amber: "Using synthetic demo data — 
connect your real data source for accurate 
opportunity projections"

Button: Reset to Demo Data (calls POST /data/reset)

SECTION B — Connect data source

Title: "Connect Your Data"
Subtitle: "Replace demo data with your real 
Medicare Advantage member and claims data"

Show 5 cards in a grid:

CARD 1 — CSV / Excel Upload (ACTIVE - orange button)
Title: Upload CSV or Excel
Description: Upload member roster, care gap 
file, or plan configuration
Supported formats: .csv, .xlsx, .xls
Button: Upload Files

CARD 2 — Snowflake (COMING SOON)
Title: Snowflake Data Cloud
Description: Connect directly to your 
Snowflake warehouse via Snowflake MCP
Badge: Coming Soon

CARD 3 — Databricks (COMING SOON)  
Title: Databricks Lakehouse
Description: Connect to Delta tables with 
member and claims data
Badge: Coming Soon

CARD 4 — SQL Server (COMING SOON)
Title: Microsoft SQL Server
Description: Connect to on-premise or 
Azure SQL Server
Badge: Coming Soon

CARD 5 — BigQuery (COMING SOON)
Title: Google BigQuery
Description: Connect to BigQuery datasets 
with claims and eligibility data
Badge: Coming Soon

SECTION C — CSV Upload flow

When Upload Files is clicked show a modal:

Step 1 — Select data type
Radio buttons:
○ Member Roster (replaces member data)
○ Care Gap File (replaces gap data)
○ Plan Configuration (replaces plan data)
○ Full Dataset (members + gaps + plans together)

Step 2 — Download template
Show Download Template button for selected type
Templates should be CSV files with:
- Headers matching our schema
- 3 example rows with realistic sample data
- A README row at top explaining each column

Step 3 — Upload file
Drag and drop area or click to browse
Accept .csv and .xlsx files
Show file name and size after selection

Step 4 — Preview and validate
After file selected show:
- First 5 rows preview in a table
- Column mapping: their column → our column
- Any validation errors highlighted in red
- Row count and data summary

Step 5 — Confirm import
Button: Import X rows
On confirm call appropriate upload endpoint
Show progress bar during import
Show success summary or error details

SECTION D — Download templates

Three template download buttons:
Download Member Template
Download Gap Template  
Download Plan Template

Add these API endpoints to api.py:

POST /data/upload/members
Accepts multipart form file upload
Reads CSV or Excel using pandas or csv module
Validates required columns:
  member_id, date_of_birth, gender,
  language_preference, digital_literacy_segment,
  socioeconomic_segment, email_allowed,
  sms_allowed, call_allowed, preferred_channel,
  do_not_contact_flag
Maps uploaded columns to database schema
Clears existing dim_member and 
dim_member_channel_pref data
Inserts new rows
Returns: {imported: X, skipped: Y, errors: [...]}

POST /data/upload/gaps
Validates required columns:
  member_id, measure_code, plan_id,
  measurement_year, gap_status, gap_open_date,
  clinical_risk_score, days_open
Clears existing fact_member_gap data
Inserts new rows
Returns import summary

POST /data/upload/plans
Validates required columns:
  plan_id, plan_name, region, segment,
  current_star_rating, target_star_rating,
  annual_revenue, total_members
Clears existing dim_plan_contract and 
plan_population data
Inserts new rows
Returns import summary

GET /data/templates/members
Returns CSV file download with headers and 
3 example rows

GET /data/templates/gaps
Returns CSV file download

GET /data/templates/plans
Returns CSV file download

GET /data/status
Returns current data status:
{
  members: {count: X, source: "demo", updated: "date"},
  gaps: {count: X, source: "demo", updated: "date"},
  plans: {count: X, source: "demo", updated: "date"},
  runs: {count: X}
}

POST /data/reset
Runs seed_demo_data.py to restore all demo data
Returns confirmation and new row counts

After building test the full upload flow:
1. Download the member template
2. Add 5 new test rows to it
3. Upload it via the modal
4. Confirm the 5 members appear in the database
5. Check that Opportunities tab still loads correctly
6. Verify GET /data/status shows updated counts

Show me:
1. EED on P002 financial calculation with 
   realistic data (from Part 1)
2. Data Sources tab rendering in dashboard
3. Upload test results for the 5 member test
4. Template download working for all 3 types
5. Reset endpoint working correctly
