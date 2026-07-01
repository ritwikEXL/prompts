Make the following changes to dashboard/index.html and api.py:

CHANGE 1 — Opportunity Agent filters and simplified display

Add a filter bar above the Opportunity Radar table with:
- Dropdown to filter by Plan (all 5 plans plus All Plans option)
- Dropdown to filter by Measure (all 4 measures plus All Measures)
- Dropdown to filter by Priority (High, Medium, Low, All)
- A Clear Filters button

Remove the composite Score column from the radar table.
Replace it with just Open Gaps, Priority badge, and Stars 
gap to close shown as current arrow target.

Add a summary strip above the filter bar showing:
- Total open gaps in current filtered view
- Number of High priority combinations
- Estimated Stars impact if all filtered gaps closed

CHANGE 2 — Campaign Agent inline editing

In the Run Session tab Phase 3 campaign design step, 
after the user selects Option 1 or Option 2, show an 
editable campaign details card before the confirm button.

The card should show these fields as editable inputs:
- Primary channel (dropdown: Email, SMS, Call)
- Fallback channel (dropdown: Email, SMS, Call, None)
- Incentive type (dropdown: Gift Card 15, Gift Card 25, 
  Transport Voucher, Fit Kit Mailer, None)
- Frequency plan (text input, pre-filled from agent recommendation)
- Message theme (text input, pre-filled from agent recommendation)
- Member count per cohort (read only, not editable)

Show a Confirm Campaign button below the editable card.
Any field the user edits should be highlighted in amber 
to show it was manually changed from the AI recommendation.
Log any manual changes in the audit trace with the note 
"manually overridden by user" alongside the field name 
and the original vs new value.

CHANGE 3 — Member count visibility in cohort design

In Phase 2 cohort design, make the member count per 
cohort much more prominent. Show it as a large number 
at the top of each cohort card with a label like 
"3 members targeted" in a teal badge. Also show the 
channel fit and average propensity score prominently 
on each cohort card so the user can see at a glance 
who they are reaching and how.
