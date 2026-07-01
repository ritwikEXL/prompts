Make the following fixes to dashboard/index.html:

FIX 1 — Remove composite score reference from summary strip
In the Opportunities tab summary strip at the top, the 
High Priority card currently shows "composite score ≥ 8.0" 
as the criteria description. Remove this text entirely. 
Just show the count number and the label "High Priority" 
with no criteria explanation underneath. Same for the 
other summary cards — remove any formula or calculation 
references. Keep the numbers and labels only.

FIX 2 — Add back navigation between agent phases
In the Run Session tab, add a "← Go Back" button to 
each phase step after Phase 1. Specifically:

Phase 2 Cohort Design — show a "← Back to Opportunity" 
button that returns the session to Phase 1 state, clears 
the cohort assignments from the database for this run_id 
using DELETE /session/{run_id} and restarts from Phase 1 
with the same opportunity pre-selected so the user can 
choose a different one or reconfirm the same one.

Phase 3 Campaign Design — show a "← Back to Cohorts" 
button that returns to Phase 2 state and clears campaign 
data for this run_id from the database.

Phase 4 Outreach — show a "← Back to Campaign" button 
that returns to Phase 3 state and clears outreach data.

When going back, the completed phases above should 
change from green Completed state back to their 
Active state with their previous options still visible 
so the user does not have to start completely from scratch.

Show the Go Back button in a subtle gray style so it 
does not compete visually with the main confirm button 
which stays teal. Position it to the left of the 
confirm button.

FIX 3 — Add Target Cohort 2 Only option
In Phase 2 Cohort Design, add a third button alongside 
the existing "Target Both Cohorts" and "Target Cohort 1 Only" 
buttons:

Add "Target Cohort 2 Only" button with the same styling 
as the existing cohort selection buttons.

When selected it should behave identically to Target 
Cohort 1 Only but targeting only the second cohort — 
marking only Cohort 2 members as in scope in the 
fact_nba_claude_decision table and setting Cohort 1 
members as not targeted.

Also update the cohort card highlighting — when a cohort 
is selected it should show a teal border and a Selected 
badge. When it is not selected it should show a grayed 
out style so it is visually clear which cohorts are 
active for this session.

Keep all existing visual design unchanged. Only add 
these 3 fixes.
