Expand the CareIntel database with more realistic data volumes.
Read the existing schema from careintel.db and the current 
data in all tables before making any changes.

STEP 1 — Add 3 new measures to dim_measure

Add these rows keeping existing 4 measures unchanged:

M005, MAD, Medication Adherence for Diabetes, Process, 
star_weight=2, hedis_domain=Medication Management, 
age_gender_eligibility=Adults 18-75 with diabetes on 
medication, clinical_description=Percentage of members 
with diabetes who filled their diabetes medication for 
at least 80 percent of the year, 
nba_default_playbook=PB_MAD_STANDARD

M006, AFV, Annual Flu Vaccine, Process, star_weight=1.5, 
hedis_domain=Prevention and Screening, 
age_gender_eligibility=All members 6 months and older, 
clinical_description=Percentage of members who received 
an influenza vaccination between July and June, 
nba_default_playbook=PB_AFV_STANDARD

M007, SPC, Statin Use in Persons with Cardiovascular 
Disease, Process, star_weight=2, 
hedis_domain=Medication Management, 
age_gender_eligibility=Adults 21 and older with 
cardiovascular disease, 
clinical_description=Percentage of members with 
cardiovascular disease who were dispensed a statin 
medication, nba_default_playbook=PB_SPC_STANDARD

STEP 2 — Add 80 new members to dim_member

Add members MBR0021 through MBR0100 with realistic 
diversity:
- Age bands distributed across 45-49, 50-54, 55-59, 
  60-64, 65-69, 70-74, 75-79, 80+ 
  (weight toward 65-79 since this is Medicare)
- Languages: 55 percent EN, 20 percent ES, 
  15 percent ZH, 10 percent Other
- Digital literacy: 30 percent Low, 40 percent Medium, 
  30 percent High
- Socioeconomic segment: 35 percent Low, 
  40 percent Mid, 25 percent High
- Gender: mix of M and F
- PCP provider keys: distribute across PRV101 to PRV110
- Birth years consistent with age bands
- dob_year should reflect realistic Medicare age 
  (born between 1925 and 1979)

STEP 3 — Add corresponding rows to dim_member_channel_pref

For each new member MBR0021 through MBR0100 add a 
channel preference row with:
- email_allowed: true for High and Mid digital literacy, 
  false for Low digital literacy members
- sms_allowed: true for most members except very elderly 
  (80+ age band) Low digital literacy
- call_allowed: true for all members
- preferred_channel: EMAIL for High digital literacy, 
  SMS for Mid digital literacy EN speakers, 
  CALL for Low digital literacy and non-English speakers 
  who are elderly
- do_not_contact_flag: false for all except 3 random 
  members (to maintain realistic Do Not Contact rate)
- channel_risk_notes: realistic notes matching the 
  member profile

STEP 4 — Add 200 new gap rows to fact_member_gap

Add gap rows G00051 through G00250 covering all 7 
measures across all 5 plans. Distribute as follows:

Per measure distribution:
- BCS (Breast Cancer Screening): 25 gaps
- COL (Colorectal Cancer Screening): 30 gaps
- EED (Eye Exam for Patients with Diabetes): 35 gaps
- CDC (Controlling Blood Pressure): 35 gaps
- MAD (Medication Adherence for Diabetes): 40 gaps
- AFV (Annual Flu Vaccine): 20 gaps
- SPC (Statin Use): 15 gaps

Per plan distribution spread realistically across 
all 5 plans with P003 (3.0 stars) and P005 (2.5 stars) 
having proportionally more open gaps since they are 
the most at-risk plans.

For each gap row:
- gap_status: 50 percent Open, 20 percent Borderline, 
  20 percent Partial, 10 percent Closed
- gap_open_date: between 2026-01-01 and 2026-04-01
- days_open: calculated from gap_open_date to today
- clinical_risk_score: between 0.3 and 0.9, 
  higher for chronic condition measures 
  (CDC, MAD, SPC, EED)
- nba_propensity_score: between 0.3 and 0.85
- previous_year_gap_flag: true for 30 percent of gaps
- upstream_recommended_channel: distribute across 
  EMAIL, SMS, CALL based on member channel preference
- upstream_recommended_incentive: 
  GIFTCARD_15 for high propensity digital members,
  GIFTCARD_25 for medium propensity,
  TRANSPORT_VOUCHER for low socioeconomic members,
  FIT_KIT_MAILER for COL measure members,
  NONE for already partially compliant members
- upstream_recommended_priority: High for clinical 
  risk above 0.7, Medium for 0.5 to 0.7, Low below
- last_outreach_date: null for most, 
  some with dates in last 90 days
- last_outreach_channel: matching upstream channel 
  where last_outreach_date is not null
- is_suppressed: true only for Do Not Contact members

STEP 5 — Verify the data

After inserting everything run these counts and 
show me the results:

SELECT COUNT(*) FROM dim_member
SELECT COUNT(*) FROM dim_member_channel_pref  
SELECT COUNT(*) FROM dim_measure
SELECT COUNT(*) FROM fact_member_gap
SELECT gap_status, COUNT(*) FROM fact_member_gap 
GROUP BY gap_status
SELECT measure_code, COUNT(*) FROM fact_member_gap 
GROUP BY measure_code
SELECT plan_key, COUNT(*) FROM fact_member_gap 
GROUP BY plan_key
SELECT COUNT(*) FROM fact_member_gap 
WHERE is_suppressed = 0 AND gap_status != 'Closed'

The last query should show roughly 180 to 200 
addressable open gaps.

Keep all existing data completely unchanged. 
Only add new rows. Never modify or delete 
existing members, gaps, or measures.
