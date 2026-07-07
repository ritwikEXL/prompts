Fix the expected closure rate logic in api.py 
to be more realistic and measure-specific.

Update the POST /evaluate/{run_id} endpoint 
to calculate expected closure rate based on 
the measure type and evaluation window rather 
than a flat 20 percent.

Use these realistic expected closure rates 
based on Healthcare Effectiveness Data and 
Information Set industry benchmarks:

For evaluation_window = 7 days:
- MAD (Medication Adherence for Diabetes): 
  expected 45 percent — members refill quickly
- AFV (Annual Flu Vaccine): 
  expected 35 percent — easy to get, walk-in
- CDC (Controlling Blood Pressure): 
  expected 25 percent — requires BP check visit
- EED (Eye Exam for Patients with Diabetes): 
  expected 15 percent — requires specialist visit
- BCS (Breast Cancer Screening): 
  expected 12 percent — requires scheduling mammogram
- COL (Colorectal Cancer Screening): 
  expected 10 percent — requires prep, harder to schedule
- SPC (Statin Use in Cardiovascular Disease): 
  expected 40 percent — just needs prescription refill

For evaluation_window = 14 days:
Multiply each 7-day rate by 1.8

For evaluation_window = 30 days:
These are the final expected closure rates:
- MAD: 75 percent
- AFV: 65 percent
- CDC: 50 percent
- EED: 35 percent
- BCS: 30 percent
- COL: 25 percent
- SPC: 70 percent

Update performance thresholds to be relative 
to these measure-specific benchmarks:
- On Track: actual rate within 15 percent 
  of expected rate for that measure and window
- Underperforming: actual rate more than 
  15 percent below expected
- Overperforming: actual rate more than 
  15 percent above expected

Also update the campaign_evaluations table 
to store the measure_code so the correct 
benchmark can be applied.

Add a tooltip or info icon next to the 
Expected Rate column in the dashboard 
Evaluation tab that shows:
"Based on [measure_code] industry benchmark 
for day [window] outreach. Source: HEDIS 
national averages."

After updating restart uvicorn and re-run 
all existing evaluations by calling 
POST /evaluate/run-scheduled and show me 
the updated performance statuses.
