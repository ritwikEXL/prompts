Build a 5th agent called the Evaluation Agent and 
add it to the CareIntel system.

STEP 1 — Create agents/evaluation-agent.md

Create a new agent instructions file with this content:

# Evaluation Agent

## Role
A healthcare campaign performance analyst that monitors 
active outreach campaigns, tracks gap closure rates 
against expected targets, and recommends corrective 
actions for underperforming campaigns.

## Key Responsibilities
- Monitor gap closure status for all members who 
  received outreach
- Compare actual closure rate vs expected closure 
  rate from the campaign design
- Flag campaigns that are underperforming after 
  defined evaluation windows (7 days, 14 days, 30 days)
- Recommend specific corrective actions per member 
  based on their profile and response pattern
- Calculate updated Stars impact projection based 
  on actual vs predicted performance
- Generate evaluation summary reports per campaign

## Evaluation Windows
- Day 7 check — early signal, expect 20 percent 
  of closures to have happened
- Day 14 check — mid campaign, expect 50 percent 
  of closures
- Day 30 check — final evaluation, full closure 
  rate assessment

## Performance Thresholds
- On track — actual closure rate within 10 percent 
  of expected
- Underperforming — actual closure rate more than 
  10 percent below expected
- Overperforming — actual closure rate more than 
  10 percent above expected

## Corrective Action Menu
- ESCALATE_INCENTIVE — increase gift card amount 
  by $10 for non-responders
- SWITCH_CHANNEL — try a different outreach channel 
  if member has not responded on current channel
- CARE_MANAGER_CALL — escalate to human care manager 
  for high risk members who have not responded
- EXTEND_CAMPAIGN — add 2 more weeks to the campaign 
  window for borderline members
- CLOSE_CAMPAIGN — close the campaign if closure 
  rate is above 80 percent
- NO_ACTION — member already closed gap or opted out

## Output
For each evaluated campaign produce:
- Overall performance status (On Track, Underperforming, 
  Overperforming)
- Per member status and recommended next action
- Updated Stars impact projection
- Executive summary suitable for a VP of Quality

STEP 2 — Add evaluation tables to careintel.db

Run these SQL statements to add new tables:

CREATE TABLE IF NOT EXISTS campaign_evaluations (
    evaluation_id TEXT PRIMARY KEY,
    nba_run_id TEXT,
    campaign_id TEXT,
    evaluation_date TEXT,
    evaluation_window INTEGER,
    total_members_contacted INTEGER,
    gaps_closed_actual INTEGER,
    gaps_closed_expected INTEGER,
    actual_closure_rate REAL,
    expected_closure_rate REAL,
    performance_status TEXT,
    stars_impact_actual REAL,
    stars_impact_projected REAL,
    executive_summary TEXT,
    created_timestamp TEXT
)

CREATE TABLE IF NOT EXISTS member_evaluations (
    member_eval_id TEXT PRIMARY KEY,
    evaluation_id TEXT,
    nba_run_id TEXT,
    contact_id TEXT,
    member_gap_key TEXT,
    member_key TEXT,
    outreach_sent_date TEXT,
    gap_status_at_evaluation TEXT,
    days_since_outreach INTEGER,
    responded INTEGER DEFAULT 0,
    recommended_action TEXT,
    action_reason TEXT,
    follow_up_scheduled TEXT,
    created_timestamp TEXT
)

CREATE TABLE IF NOT EXISTS evaluation_schedule (
    schedule_id TEXT PRIMARY KEY,
    nba_run_id TEXT,
    campaign_id TEXT,
    scheduled_date TEXT,
    evaluation_window INTEGER,
    status TEXT DEFAULT 'PENDING',
    created_timestamp TEXT
)

STEP 3 — Add evaluation endpoints to api.py

Add these new endpoints:

POST /evaluate/{run_id}
Manually trigger evaluation for a specific run.
1. Read all contacts from fact_nba_outreach_plan 
   for this run_id where status = SENT
2. For each contact check current gap_status in 
   fact_member_gap
3. If gap_status = Closed mark member as responded
4. Calculate actual closure rate vs expected 
   closure rate from dim_nba_campaign
5. Determine performance status based on thresholds
6. For each non-responding member determine 
   recommended action based on:
   - Days since outreach sent
   - Member digital literacy and socioeconomic segment
   - Clinical risk score from fact_member_gap
   - Whether they have consented to alternative channels
7. Generate executive summary text
8. Write results to campaign_evaluations and 
   member_evaluations tables
9. Auto-schedule next evaluation based on 
   evaluation window (if day 7 check passes, 
   schedule day 14 check automatically)
10. Return full evaluation report as JSON

GET /evaluate/{run_id}/latest
Return the most recent evaluation for a run_id
including campaign level and member level results.

GET /evaluate/all
Return evaluations for all runs summarized 
at campaign level — for the portfolio overview.

POST /evaluate/schedule/{run_id}
Schedule automatic evaluations at day 7, 14, 
and 30 after outreach was sent.
Write 3 rows to evaluation_schedule table with 
scheduled_dates calculated from the sent_at 
timestamp of the first outreach contact.

GET /evaluate/schedule/due
Return all evaluations that are due today or 
overdue — where scheduled_date <= today and 
status = PENDING.
This endpoint is called by the auto-scheduler.

POST /evaluate/run-scheduled
Trigger all due evaluations from evaluation_schedule.
For each due evaluation call POST /evaluate/{run_id}.
Update evaluation_schedule status to COMPLETED.
Return summary of how many evaluations were run.

STEP 4 — Add auto-scheduler to api.py

Add a background scheduler using Python's 
threading module that runs every 24 hours 
and calls POST /evaluate/run-scheduled automatically.

Start the scheduler when uvicorn starts up.
Log to console each time the scheduler runs:
"Auto-evaluation scheduler running — checking 
for due evaluations"

STEP 5 — Add Evaluation tab to dashboard

Add a new tab called "Evaluation" between 
Opportunities and Run Session in the navigation bar.

The Evaluation tab has three sections:

SECTION 1 — Portfolio evaluation summary strip
4 stat cards at the top:
- Active campaigns being monitored
- Average closure rate across all campaigns
- Campaigns on track (green)
- Campaigns needing attention (red)

SECTION 2 — Campaign performance table
One row per evaluated campaign showing:
- Run ID and date
- Measure and plan
- Members contacted
- Expected closure rate vs actual closure rate 
  shown as two colored bars side by side
- Performance status badge: 
  On Track (green), Underperforming (red), 
  Overperforming (teal)
- Stars impact actual vs projected
- Last evaluated date
- Evaluate Now button that calls 
  POST /evaluate/{run_id}
- View Details button that expands member 
  level results below

SECTION 3 — Member level evaluation detail
Shown when View Details is clicked on a campaign row.
Shows one row per member contacted with:
- Member ID
- Gap measure
- Outreach channel and date sent
- Days since outreach with urgency color coding:
  Green for 0-7 days, Amber for 8-14 days, 
  Red for 15+ days
- Current gap status (Open, Closed, Partial)
- Responded indicator (yes/no with icon)
- Recommended action badge with color coding:
  Teal for CLOSE_CAMPAIGN
  Green for NO_ACTION
  Amber for ESCALATE_INCENTIVE or EXTEND_CAMPAIGN
  Orange for SWITCH_CHANNEL
  Red for CARE_MANAGER_CALL
- Take Action button that implements the 
  recommended action

SECTION 4 — Auto-evaluation status
A small panel at the bottom showing:
- Next scheduled auto-evaluation date and time
- Last time auto-scheduler ran
- Number of pending evaluations in queue
- A Run All Due Evaluations Now button that 
  calls POST /evaluate/run-scheduled

STEP 6 — Integrate evaluation into session completion

When a session completes in Run Session tab 
(both manual and automated mode):
Automatically call POST /evaluate/schedule/{run_id} 
to schedule the day 7, 14, and 30 evaluations.
Show a small notice below the Session Complete 
banner saying:
"Evaluation scheduled — this campaign will be 
automatically evaluated on [date+7], [date+14], 
and [date+30]"

STEP 7 — Connect evaluation to gap closing

When the Evaluation Agent determines a member 
has responded (gap is now Closed):
Update fact_member_gap gap_status to Closed 
if not already done.
Update fact_nba_outreach_plan status to COMPLETED 
for that contact.
Log in fact_nba_trace:
"Gap closed confirmed for [member_key] — 
[measure_code] gap [member_gap_key] closed 
following outreach on [sent_date]"

STEP 8 — Simulate some historical evaluation data

To make the Evaluation tab look populated on first load,
simulate evaluation data for the existing runs in 
the database:

For each run_id in fact_nba_trace:
- Create a campaign_evaluation record
- Set evaluation_window to 7
- Set gaps_closed_actual to a realistic number 
  (between 30 and 70 percent of members contacted)
- Calculate performance_status based on thresholds
- Generate a realistic executive summary
- Create member_evaluation records for each contact

This gives the Evaluation tab real data to display 
immediately rather than showing empty state.

After all changes:
1. Restart uvicorn
2. Call GET /evaluate/all and show me the response
3. Call GET /evaluate/schedule/due and show me 
   any due evaluations
4. Open dashboard at http://localhost:8080/dashboard/
5. Click the Evaluation tab and confirm it loads
6. Show me a screenshot description of what appears

Keep all existing visual design and functionality 
completely unchanged.
