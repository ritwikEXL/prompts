Prepare CareIntel for stakeholder testing. 
Do these steps in order and confirm each before 
moving to the next.

STEP 1 — Change brand color to EXL orange

Replace all instances of the current teal color 
#1F6F5C and its variants throughout dashboard/index.html 
with EXL orange #F15A22.

Specifically replace:
- Primary background color on nav bar
- Button colors (primary buttons)
- Active tab indicators
- Progress bar fills
- Badge backgrounds for High priority
- Border accents on selected cards
- Agent status active/completed indicators

Keep white and light gray backgrounds unchanged.
Keep green color for Completed/Success states.
Keep red for Failed/Underperforming states.
Only replace teal with orange.

Also update the CareIntel logo text color 
in the nav bar from teal to white on orange background.

Show me a before and after of the nav bar 
and one opportunity card.

STEP 2 — Clear all data and reseed cleanly

2a. Clear all output tables completely:
DELETE FROM fact_nba_claude_decision;
DELETE FROM dim_nba_campaign;
DELETE FROM fact_nba_outreach_plan;
DELETE FROM fact_nba_trace;
DELETE FROM campaign_evaluations;
DELETE FROM member_evaluations;
DELETE FROM evaluation_schedule;
DELETE FROM whatsapp_conversations;

Reset all gap statuses to reflect 
star-correlated compliance rates:
P005 (2.5 stars) — 38% gaps Closed
P003 (3.0 stars) — 50% gaps Closed
P001 (3.5 stars) — 60% gaps Closed
P002 (4.0 stars) — 68% gaps Closed
P004 (4.5 stars) — 73% gaps Closed

Randomly select gaps to close per plan 
as described above. Keep remaining gaps 
as mix of Open, Borderline, Partial.

2b. Update member names to be more realistic
Instead of MBR0001, MBR0002 etc update 
dim_member to have realistic first names:
Add a display_name column with realistic 
Medicare-age appropriate names like:
MBR0001 → Margaret Chen
MBR0002 → Roberto Silva  
MBR0003 → James Williams
MBR0004 → Sarah Johnson
etc.
Generate realistic names that match the 
language_preference of each member:
EN members → English names
ES members → Spanish names
ZH members → Chinese names

2c. Update plan names to sound more realistic:
Instead of Aurora MA-PD Choice use:
P001 → Aetna Medicare Choice PPO (Northeast)
P002 → Aetna Medicare Premier PPO (Southeast)
P003 → Aetna Medicare DSNP Community (Midwest)
P004 → UHC Medicare Advantage Value (West)
P005 → UHC Medicare Signature PPO (West)

Update dim_plan_contract with these names.
This makes the demo feel real to Ankit and Rishi.

STEP 3 — Run automated sessions to build history

Run 4 complete automated sessions covering 
different measures and plans to build realistic 
run history. For each session:
- Use the existing agent pipeline
- Pick a different measure x plan combination
- Run in automated mode
- Save all outputs to database

Session 1: EED x P005 (UHC Medicare Signature)
Most urgent — 2.5 star plan, diabetic eye exam gaps
Run automated mode, save all outputs

Session 2: CDC x P003 (Aetna Medicare DSNP Community)  
High weight measure on vulnerable population
Run automated mode, save all outputs

Session 3: MAD x P004 (UHC Medicare Advantage Value)
Medication adherence — easiest to close
Run automated mode, save all outputs

Session 4: BCS x P001 (Aetna Medicare Choice PPO)
Breast cancer screening — prior year repeat gaps
Run automated mode, save all outputs

After each session write trace entries and 
update outreach plan with SENT status for 
all contacts.

STEP 4 — Simulate member responses and evaluations

For each of the 4 sessions simulate realistic 
member response rates:

Session 1 EED x P005 — underperforming scenario
Simulate 20% response rate (below expected 35%)
Mark 20% of contacted members as gap Closed
Mark 80% as No Response
This triggers CARE_MANAGER_CALL recommendation

Session 2 CDC x P003 — on track scenario
Simulate 35% response rate (matches expected)
Mark 35% of contacted members as gap Closed
Mark 65% as No Response

Session 3 MAD x P004 — overperforming scenario
Simulate 55% response rate (above expected 35%)
Mark 55% of contacted members as gap Closed
Mark 45% as No Response

Session 4 BCS x P001 — on track scenario
Simulate 30% response rate
Mark 30% of contacted members as gap Closed

After simulating responses run the evaluation 
agent for all 4 sessions:
POST /evaluate/{run_id} for each run_id
Store results in campaign_evaluations and 
member_evaluations tables

Show me the evaluation summary for all 4 campaigns:
- Performance status (On Track / Under / Over)
- Actual vs expected closure rate
- Stars impact actual
- Top recommended action for non-responders

STEP 5 — End to end test

Run a complete end to end test of the full 
product flow:

1. Open Opportunities tab — verify P005 shows 
   as default with correct gap counts and 
   financial projections
2. Click Launch Campaign on top opportunity 
   for P005 — verify it switches to Run Session
3. Run one complete session in manual mode —
   go through all 4 phases, confirm each step
4. After session completes, verify outreach 
   plan shows correct contacts
5. Click Send All — verify WhatsApp and email 
   delivery attempts are logged
6. Check Evaluation tab — verify all 4 historical 
   campaigns show with correct performance status
7. Verify Sessions dropdown shows all run history

Report any errors found during testing with 
the specific step where they occurred.

STEP 6 — Prepare for hosting

Create a requirements.txt file with all 
Python dependencies:
fastapi
uvicorn
sqlite3
python-dotenv
twilio
sendgrid
anthropic
pyngrok

Create a Procfile for Render deployment:
web: uvicorn api:app --host 0.0.0.0 --port $PORT

Create a render.yaml file:
services:
  - type: web
    name: careintel-api
    env: python
    buildCommand: pip install -r requirements.txt
    startCommand: uvicorn api:app --host 0.0.0.0 --port $PORT
    envVars:
      - key: ANTHROPIC_API_KEY
        sync: false
      - key: TWILIO_ACCOUNT_SID
        sync: false
      - key: TWILIO_AUTH_TOKEN
        sync: false
      - key: TWILIO_WHATSAPP_NUMBER
        sync: false
      - key: SENDGRID_API_KEY
        sync: false
      - key: SENDGRID_SENDER_EMAIL
        sync: false
      - key: TEST_SMS_NUMBER
        sync: false
      - key: TEST_EMAIL
        sync: false

Update api.py to handle the database path 
correctly when running on Render — use an 
environment variable for DB path:
DB_PATH = os.getenv('DB_PATH', 'careintel.db')

Also update dashboard/index.html to read 
the API URL from a config rather than 
hardcoding localhost:8000:

Add at the top of the script section:
const API_BASE = window.location.hostname === 
'localhost' ? 'http://localhost:8000' : 
'https://your-render-app-name.onrender.com';

Replace all hardcoded localhost:8000 references 
with API_BASE.

Show me the requirements.txt and render.yaml 
after creating them.

After all steps complete give me:
1. Summary of what was done
2. Any errors encountered
3. Instructions for deploying to Render
4. A testing checklist for Ankit and Rishi
   with the specific flows to test
