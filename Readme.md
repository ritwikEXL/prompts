Fix three issues in one prompt:

PART 1 — Fix ROI calculation permanently

The ROI is showing 3000x+ because outreach cost 
is tiny relative to CMS bonus. The root cause is 
Stars improvement is calculated correctly against 
the full eligible population but outreach cost 
only reflects the small tier cohort we show.

This is actually mathematically correct — you spend 
$946 to contact your best prospects and if that 
moves the needle on a $380M plan the ROI IS massive.

But it is misleading without context. Fix the display:

In the opportunity card ROI section:

CHANGE 1: Show cost and bonus separately with 
clear scope labels:

Campaign Cost: $946
(contacting 110 highest-priority members)

Expected Stars Impact: +0.018★
(across 5,400 total eligible members)

CMS Bonus Value: $1.7M
(if Stars improvement is sustained across 
measurement year — varies by plan contract)

Net Campaign Return: $1.69M estimated

Instead of showing ROI ratio as a number 
(which always looks absurd) show it as:

"Every $1 invested in this campaign could 
generate $1,786 in CMS bonus value"

Add a disclaimer below:
"CMS bonus calculations assume Stars improvement 
is maintained through measurement year end. 
Actual bonus depends on performance across 
all measures. Projections based on estimated 
plan population — connect Snowflake for 
plan-specific figures."

CHANGE 2: Add a simple visual instead of ratio

Show a bar comparison:
Campaign Cost:  ████ $946
Expected Bonus: ████████████████████████ $1.7M

This makes the value proposition clear without 
showing an absurd multiplier.

CHANGE 3: Remove ROI ratio number entirely
Never show Xx or 3110x anywhere on the dashboard.
Replace all ROI ratio displays with the 
"Every $1 generates $X in CMS bonus value" 
format with the disclaimer.

PART 2 — Fix confidence indicator

Update confidence rules based on actual 
eligible member counts from fact_member_gap:

HIGH CONFIDENCE:
- eligible_members >= 500 for this measure x plan
- AND historical outreach data exists in 
  fact_nba_trace for this measure x plan
Description: "Large member population with 
historical response data. Projections are 
reliable for budget planning."

MEDIUM CONFIDENCE:
- eligible_members between 50 and 499
- OR eligible_members >= 500 but no historical data
Description: "Moderate member population. 
Projections based on industry benchmarks. 
Connect full claims data for precise figures."

LOW CONFIDENCE:
- eligible_members < 50
Description: "Small member cohort detected. 
Projections may not be statistically reliable. 
Strongly recommend verifying with full plan data 
before committing budget."

Update the confidence display to always show 
the actual eligible member count:
"Medium confidence — 280 eligible members 
(industry benchmark rates applied)"

Not "10-20 eligible members" — use real count.

PART 3 — Fix Data Sources tab with full functionality

The Data Sources tab is currently showing 
a blank or incomplete UI. Rebuild it with 
full working functionality.

CORE CONCEPT — Multi-source workspace:

The PM can have multiple data sources loaded 
simultaneously and switch between them. All 
three tabs (Opportunities, Run Session, 
Evaluation) update based on the active source.

Add a source_id column to these tables 
if not already present:
ALTER TABLE fact_member_gap 
ADD COLUMN source_id TEXT DEFAULT 'demo';
ALTER TABLE dim_member 
ADD COLUMN source_id TEXT DEFAULT 'demo';
ALTER TABLE dim_plan_contract 
ADD COLUMN source_id TEXT DEFAULT 'demo';

Add a data_sources registry table:
CREATE TABLE IF NOT EXISTS data_sources (
    source_id TEXT PRIMARY KEY,
    source_name TEXT,
    source_type TEXT,
    file_name TEXT,
    uploaded_at TEXT,
    member_count INTEGER,
    gap_count INTEGER,
    plan_count INTEGER,
    is_active INTEGER DEFAULT 0,
    created_timestamp TEXT
)

Insert default demo source:
INSERT OR IGNORE INTO data_sources VALUES (
    'demo',
    'CareIntel Demo Database',
    'sqlite',
    'careintel.db',
    datetime('now'),
    250, 611, 5,
    1,
    datetime('now')
)

ADD API ENDPOINTS:

GET /data/sources
Returns all data sources from data_sources table
Including which one is currently active.

POST /data/sources/activate/{source_id}
Sets is_active = 1 for this source_id
Sets is_active = 0 for all others
Returns confirmation
Also updates a global app state variable 
active_source_id that all other endpoints 
read from.

Update ALL data endpoints to filter by 
active_source_id:

GET /opportunities — filter fact_member_gap 
WHERE source_id = active_source_id

GET /members — filter dim_member 
WHERE source_id = active_source_id

GET /gaps — filter fact_member_gap 
WHERE source_id = active_source_id

GET /session/latest — no source filter needed 
(sessions belong to all sources)

POST /data/upload/members — 
generate unique source_id for upload
source_id = 'upload_' + timestamp
Insert all uploaded members with this source_id
Insert row in data_sources table
Automatically activate the new source

POST /data/upload/gaps — same pattern
POST /data/upload/plans — same pattern

POST /data/upload/full — accepts a zip or 
multiple files containing members, gaps, 
and plans together. Processes all three 
and creates one source_id for the set.

GET /data/sources/{source_id}/summary
Returns summary stats for a specific source:
member_count, gap_count, plan_count,
open_gap_count, measures_below_benchmark,
top_opportunity_measure, top_opportunity_plan

DELETE /data/sources/{source_id}
Deletes all data for this source_id from 
all tables. Cannot delete 'demo' source.

BUILD THE DATA SOURCES TAB UI:

SECTION 1 — Active data source banner
At the very top show which source is active:

Green banner: "● Currently analyzing: 
CareIntel Demo Database | 250 members | 
15,000+ gaps | 5 plans"

With a Change Source button.

SECTION 2 — Data source switcher
Show all available sources as clickable cards:

DEMO SOURCE CARD (always first):
Title: CareIntel Demo Database
Type: SQLite badge
Members: 250 | Gaps: 15,000+ | Plans: 5
Status: Active (green) or Click to activate (gray)
Info button: shows summary of demo data

UPLOADED SOURCE CARDS (one per upload):
Title: [filename or custom name]
Type: Excel/CSV badge
Members: X | Gaps: X | Plans: X  
Uploaded: [date]
Status: Active or Click to activate
Delete button (trash icon)

When any card is clicked:
1. Call POST /data/sources/activate/{source_id}
2. Show loading spinner on all three main tabs
3. Reload Opportunities tab data for new source
4. Show success banner: "Now analyzing [source name]"

SECTION 3 — Add new data source

Title: Connect a New Data Source
Subtitle: Upload your Medicare Advantage 
member and care gap data

CARD 1 — CSV/Excel Upload (ACTIVE):
Large upload area with:
- Drag and drop zone
- "Or click to browse" link
- Accepts: .csv, .xlsx, .xls
- Max file size: 50MB

Below drop zone show three upload type buttons:
[Upload Members] [Upload Gaps] [Upload Plans]
[Upload Full Dataset (ZIP)]

When file is dropped or selected:
1. Show filename and size
2. Auto-detect file type from columns
3. Show column mapping preview
4. Validate required columns
5. Show: "Found X members / X gaps / X plans"
6. Show Upload and Activate button

COLUMN VALIDATION:

For member files check for these columns 
(case insensitive, allow common variations):
Required: member_id (or member_key, MemberID)
Required: date_of_birth (or DOB, birth_date)
Optional with defaults: gender, language_preference,
digital_literacy_segment, socioeconomic_segment,
email_allowed, sms_allowed, call_allowed,
preferred_channel, do_not_contact_flag

For gap files check for:
Required: member_id, measure_code, plan_id, 
gap_status, gap_open_date
Optional: clinical_risk_score, days_open,
nba_propensity_score, previous_year_gap_flag

For plan files check for:
Required: plan_id, plan_name, 
current_star_rating, target_star_rating
Optional: region, segment, annual_revenue,
total_members

Show clear error if required columns missing:
"Missing required column: member_id. 
Download our template to see the correct format."

SECTION 4 — Download templates

Three clean download buttons:
[↓ Member Template] [↓ Gap Template] [↓ Plan Template]

Each template CSV should include:
- Header row with all column names
- Row 2: description of each column
- Rows 3-5: 3 realistic example rows
- All using realistic values a healthcare 
  analyst would recognize

Member template example rows:
M001,1952-03-15,F,EN,High,Mid,TRUE,TRUE,FALSE,EMAIL,FALSE
M002,1948-07-22,M,ES,Low,Low,FALSE,TRUE,TRUE,CALL,FALSE
M003,1955-11-08,F,ZH,Medium,Mid,TRUE,TRUE,TRUE,SMS,FALSE

Gap template example rows:
M001,BCS,P001,2026,Open,2026-01-15,0.72,156,FALSE
M002,EED,P001,2026,Borderline,2026-02-01,0.58,139,TRUE
M003,CDC,P002,2026,Open,2026-03-10,0.81,101,FALSE

Plan template example rows:
P001,Aetna Medicare Choice PPO,Northeast,MAPD,3.5,4.0,450000000,45000
P002,Aetna Medicare Premier PPO,Southeast,MAPD,4.0,4.5,380000000,38000

SECTION 5 — Coming soon connectors

Show in a muted gray row below the upload section:

Snowflake | Databricks | SQL Server | BigQuery | 
Azure Synapse | Redshift

Each with a Coming Soon badge and 
"Notify me" button that just shows 
a toast: "We'll notify you when 
[connector] is available"

WIRE UP DASHBOARD TABS TO ACTIVE SOURCE:

In dashboard/index.html add a global variable:
let activeSourceId = 'demo';
let activeSourceName = 'CareIntel Demo Database';

When data source is switched:
activeSourceId = newSourceId;
activeSourceName = newSourceName;

Pass activeSourceId as a query parameter 
to all API calls:
fetch(`${API_BASE}/opportunities?source_id=${activeSourceId}`)
fetch(`${API_BASE}/members?source_id=${activeSourceId}`)
etc.

Update the nav bar to always show which 
source is active:
Small indicator next to CareIntel logo:
"● Demo Data" or "● [filename]"

When source changes refresh:
- Opportunities tab data
- Any open session data
- Evaluation tab campaign list

After building test the full flow:

TEST 1: Demo source
Click CareIntel Demo Database card
Confirm Opportunities tab shows demo data
Confirm member count shows 250

TEST 2: Upload a test Excel file
Create a test file with 10 members and 
20 gaps using the template format
Upload it via the upload area
Confirm it appears as a new source card
Click to activate it
Confirm Opportunities tab updates to show 
analysis of the 10 test members
Confirm member count changes to 10

TEST 3: Switch back to demo
Click demo source card
Confirm Opportunities tab returns to demo data

TEST 4: Delete uploaded source
Click delete on the test upload
Confirm it is removed from source list
Confirm demo source remains

Show me results of all 4 tests and 
any errors encountered.
