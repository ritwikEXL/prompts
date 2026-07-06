The Re-Evaluate button in the Evaluation tab is 
failing with "Failed to fetch" error. 

Fix two things:

1. Make sure the POST /evaluate/{run_id} endpoint 
   is working correctly. Test it by calling it 
   for the first run_id in the campaign_evaluations 
   table and show me the response.

2. Update the Re-Evaluate button in dashboard/index.html 
   to handle errors gracefully:
   - Show a loading spinner while evaluation runs
   - If fetch fails show a specific error message 
     like "API server offline - please restart" 
     instead of generic "Failed to fetch"
   - Add a retry mechanism - if first attempt fails 
     wait 2 seconds and try once more automatically
   - After successful re-evaluation refresh only 
     that campaign row without reloading the full 
     page

3. Also fix the Run Scheduled button in the top 
   right - make sure it calls 
   POST /evaluate/run-scheduled correctly and 
   shows a success message with how many 
   evaluations were run.

Restart uvicorn and test the Re-Evaluate button 
again after fixing.


Add a member outcome simulation feature to make 
the Evaluation Agent meaningful during testing 
without a real claims feed.

CHANGE 1 — Add outcome recording to member 
evaluation detail view

In the Evaluation tab when Show Member Detail 
is expanded for a campaign, add an outcome 
column to each member row with these options:

A dropdown or button group with:
- Gap Closed — member completed the care activity
- No Response — member received outreach but did 
  not respond
- Wrong Number — outreach could not reach member
- Already Completed — member had already completed 
  before outreach
- Opted Out — member requested no further contact

When the user selects an outcome:
1. Update gap_status in fact_member_gap to:
   - Closed if outcome is Gap Closed or 
     Already Completed
   - Open if outcome is No Response
   - Suppressed if outcome is Opted Out
2. Update fact_nba_outreach_plan status to:
   - COMPLETED if Gap Closed or Already Completed
   - NO_RESPONSE if No Response
   - UNREACHABLE if Wrong Number
   - OPTED_OUT if Opted Out
3. Log in fact_nba_trace:
   "Outcome recorded for [member_key]: [outcome] 
   on [date] — gap [member_gap_key] status 
   updated to [new_status]"
4. Immediately trigger a re-evaluation of that 
   campaign by calling POST /evaluate/{run_id}
5. Update the campaign row in real time — 
   closure rate, performance status, and Stars 
   impact should all recalculate and update 
   on screen without page refresh

Add a new API endpoint for this:
POST /outcome/{contact_id}
Body: outcome (one of the 5 options above)
Does all 4 database updates above and returns 
updated campaign evaluation.

CHANGE 2 — Add outcome summary to campaign row

In the campaign performance table add a new 
column called Outcomes showing:
X closed / Y no response / Z pending
In small colored text — green for closed, 
red for no response, gray for pending

CHANGE 3 — Add a simulation panel for testing

Add a small collapsible panel at the top of 
the Evaluation tab called "Outcome Simulation 
(Testing Only)" with a yellow background.

It shows:
- A dropdown to select a run_id
- A button "Simulate 50 percent response rate" 
  that randomly marks half the contacted members 
  as Gap Closed and half as No Response
- A button "Simulate 80 percent response rate" 
  for a high performing campaign simulation
- A button "Simulate 20 percent response rate" 
  for an underperforming campaign simulation
- A Reset All Outcomes button that sets all 
  members back to pending

This lets you demo different scenarios to 
Ankit without manually clicking each member.

After running simulation automatically 
re-evaluate all affected campaigns and 
refresh the dashboard.

Keep all existing visual design unchanged.
