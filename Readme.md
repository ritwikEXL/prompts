Update the propensity score calculation in 
setup_database.py and add a function in api.py 
called calculate_propensity that computes a 
more realistic propensity score for each member 
gap using available signals in the database.

Use this formula:

base_score = 0.50

Adjustments:
+ 0.15 if digital_literacy_segment = High
+ 0.08 if digital_literacy_segment = Medium
- 0.10 if digital_literacy_segment = Low

+ 0.10 if previous_year_gap_flag = false 
  (member closed gap last year — responsive)
- 0.08 if previous_year_gap_flag = true 
  (member missed last year too — harder to reach)

+ 0.05 if email_allowed = true AND 
  sms_allowed = true (more channels available)
- 0.05 if only one channel allowed

+ 0.10 if gap_status = Borderline 
  (member is close to compliant — easier to push over)
- 0.05 if gap_status = Partial

- 0.08 if do_not_contact_flag = true 
  (should not even be here but safety check)

+ 0.08 if socioeconomic_segment = High 
  (better access to providers)
- 0.05 if socioeconomic_segment = Low 
  (access barriers)

+ 0.05 for MAD and AFV measures 
  (easier to close — pharmacy or walk-in)
- 0.05 for COL and BCS measures 
  (harder to close — requires scheduling)

Cap final score between 0.10 and 0.95

Update all existing rows in fact_member_gap 
with recalculated propensity scores using 
this formula.

Also add a GET /member/{member_key}/propensity 
endpoint that returns the propensity score 
breakdown for a specific member showing each 
factor and its contribution so the dashboard 
can display an explainability panel.

Show me a sample of 5 member gap rows before 
and after the recalculation to verify the 
scores changed meaningfully.
