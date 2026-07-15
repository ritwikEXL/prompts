Check and fix the WhatsApp and email delivery 
status in the current build.

STEP 1 — Check WhatsApp conversation status

Query the database and show me:

SELECT COUNT(*) FROM whatsapp_conversations;

SELECT conversation_state, COUNT(*) 
FROM whatsapp_conversations 
GROUP BY conversation_state;

SELECT c.contact_id, c.member_phone, 
c.conversation_state, c.appointment_date,
o.channel, o.status, o.member_gap_key
FROM whatsapp_conversations c
JOIN fact_nba_outreach_plan o 
ON c.contact_id = o.contact_id
LIMIT 10;

If whatsapp_conversations is empty it means 
the simulation sessions from the big prompt 
did not create conversation records.

Fix by creating conversation records for 
all contacts in fact_nba_outreach_plan 
where channel contains SMS, WhatsApp, 
or CALL and status = SENT:

For each such contact insert a row into 
whatsapp_conversations with:
- conversation_id: unique ID
- contact_id: from outreach plan
- nba_run_id: from outreach plan
- member_phone: TEST_SMS_NUMBER from .env
- conversation_state: vary realistically:
  40% COMPLETED (member confirmed appointment)
  25% DATE_CONFIRMED (member gave a date)
  20% AWAITING_DATE (member said yes)
  10% OUTREACH_SENT (no reply yet)
  5% DECLINED (member said no)
- appointment_date: for DATE_CONFIRMED and 
  COMPLETED states set a realistic date 
  between today and 30 days ago
- follow_up_sent: 1 for COMPLETED states
- gap_closed: 1 for COMPLETED states

STEP 2 — Check email delivery status

Query:
SELECT channel, status, COUNT(*) 
FROM fact_nba_outreach_plan 
GROUP BY channel, status;

Show me how many EMAIL contacts exist 
and what their status is.

If email contacts show FAILED or PLANNED 
rather than SENT, update them to SENT 
for the historical simulation sessions 
so the dashboard shows realistic history.

UPDATE fact_nba_outreach_plan 
SET status = 'SENT', 
sent_at = datetime('now', '-' || 
(abs(random()) % 14 + 1) || ' days')
WHERE channel = 'EMAIL' 
AND status IN ('PLANNED', 'FAILED')
AND nba_run_id IN (
    SELECT DISTINCT nba_run_id 
    FROM fact_nba_trace 
    WHERE step = 'RUN_SUMMARY'
);

STEP 3 — Verify WhatsApp simulator in 
Evaluation tab

Check that the WhatsApp Conversations section 
in the Evaluation tab is reading from the 
whatsapp_conversations table correctly.

The GET /conversations/{run_id} endpoint 
should return conversations joined with 
member details showing:
- Member display name
- Gap measure
- Conversation state with colored badge
- Appointment date if confirmed
- Last message content

If the section is empty verify the endpoint 
is being called with the correct run_id 
and returning data.

STEP 4 — Fix conversation state display

Make sure the conversation state badges 
show these colors:
- OUTREACH_SENT: gray
- AWAITING_DATE: amber
- DATE_CONFIRMED: teal/orange
- FOLLOW_UP_SENT: blue
- COMPLETED: green
- DECLINED: red
- ESCALATED: red pulsing

STEP 5 — Confirm both delivery channels

After fixes show me:
1. Count of WhatsApp conversations by state
2. Count of email contacts by status
3. Sample of 3 WhatsApp conversations 
   with their current state
4. Confirmation that Evaluation tab 
   WhatsApp section shows populated data
