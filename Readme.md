Make two important fixes to the CareIntel system:

FIX 1 — Stop auto-closing gaps when outreach is sent

Currently in api.py the POST /send/message/{contact_id} 
endpoint closes the gap in fact_member_gap when a 
message is sent. This is incorrect — sending a message 
does not mean the member completed their care activity.

Remove the gap closing logic from POST /send/message 
and POST /send/all endpoints entirely.

Gaps should only close through two ways:
1. Manual outcome recording (which we will add next)
2. The WhatsApp reply webhook flow (which we build below)

Update the outreach plan status from PLANNED to SENT 
when message is delivered — but do NOT update gap_status.

FIX 2 — Add WhatsApp reply webhook for conversational flow

Add a new table to careintel.db:

CREATE TABLE IF NOT EXISTS whatsapp_conversations (
    conversation_id TEXT PRIMARY KEY,
    member_gap_key TEXT,
    contact_id TEXT,
    nba_run_id TEXT,
    member_phone TEXT,
    conversation_state TEXT DEFAULT 'OUTREACH_SENT',
    appointment_date TEXT,
    follow_up_sent INTEGER DEFAULT 0,
    gap_closed INTEGER DEFAULT 0,
    created_timestamp TEXT,
    last_updated TEXT
)

Conversation states:
OUTREACH_SENT — initial outreach sent, waiting for reply
AWAITING_DATE — member said yes, waiting for date
DATE_CONFIRMED — appointment date received
FOLLOW_UP_SENT — follow up sent after appointment date
COMPLETED — member confirmed they completed the activity
DECLINED — member said no or not yet after follow up
ESCALATED — no response after follow up, sent to care manager

Add a new FastAPI endpoint:

POST /webhook/whatsapp
This is the Twilio webhook endpoint that receives 
incoming WhatsApp replies from members.

It receives Twilio's standard webhook POST with fields:
From, To, Body (the message text)

Handle these conversation states:

STATE: OUTREACH_SENT
If member replies with any of: yes, yeah, sure, ok, 
okay, will do, i will, going, scheduled (case insensitive)
→ Update state to AWAITING_DATE
→ Reply via Twilio: "Great! What date are you planning 
   to go? Please reply with the date (e.g. July 15)"

If member replies with any of: no, cant, cannot, busy, 
later, not now, maybe later
→ Update state to DECLINED
→ Reply: "No problem! We will check back with you 
   in 2 weeks. Your health matters to us."
→ Schedule a follow up message 14 days later

If member replies with STOP or unsubscribe
→ Update dim_member_channel_pref set do_not_contact_flag 
   to true for this member
→ Update state to DECLINED
→ Reply: "You have been unsubscribed from health 
   reminders. Reply START to resubscribe anytime."

STATE: AWAITING_DATE
Parse the member's reply for a date. Accept formats like:
July 15, 15th July, 15/07, 07/15, next Monday, tomorrow,
this week, next week

If a date is found:
→ Store as appointment_date in whatsapp_conversations
→ Update state to DATE_CONFIRMED
→ Schedule a follow up message for appointment_date + 3 days
→ Reply: "Perfect! We have noted your appointment 
   for [date]. We will check in with you a few days 
   after to see how it went. Good luck!"

If no date found:
→ Reply: "Thanks! Could you share the date you are 
   planning to go? For example: July 15"

STATE: DATE_CONFIRMED (follow up triggered by scheduler)
When appointment_date + 3 days arrives, send:
"Hi! We wanted to check in — did you manage to 
complete your [measure_name] appointment? 
Reply YES if completed or NO if not yet."
→ Update state to FOLLOW_UP_SENT
→ Update follow_up_sent to 1

STATE: FOLLOW_UP_SENT
If member replies yes, done, completed, went, finished:
→ Update gap_status in fact_member_gap to Closed
→ Update state to COMPLETED
→ Update fact_nba_outreach_plan status to COMPLETED
→ Log in fact_nba_trace: "Gap closed via WhatsApp 
   conversation — member confirmed completion"
→ Reply: "That is wonderful news! Thank you for 
   taking care of your health. Your care team has 
   been notified."
→ Trigger re-evaluation of the campaign

If member replies no, not yet, havent, did not:
→ Update state to DECLINED
→ Reply: "No worries! Would you like to schedule 
   another appointment? Reply YES to get a new 
   reminder or STOP to opt out."
→ Flag for care manager escalation in member_evaluations
→ Recommended action: CARE_MANAGER_CALL

If no reply within 7 days of follow up:
→ Update state to ESCALATED
→ Recommended action: CARE_MANAGER_CALL

Add a GET /conversations/{run_id} endpoint that 
returns all WhatsApp conversations for a run with 
their current state and appointment dates.

Add a POST /conversations/send-followups endpoint 
that checks all DATE_CONFIRMED conversations where 
appointment_date + 3 days <= today and sends the 
follow up message. Add this to the 24 hour scheduler.

FIX 3 — Register the webhook with Twilio

After creating the endpoint, print instructions for 
how to register the webhook URL in Twilio console:
Go to Twilio console → Messaging → Settings → 
WhatsApp Sandbox Settings → set the webhook URL for 
incoming messages to:
http://[your-ngrok-url]/webhook/whatsapp

Also install ngrok instructions since Twilio needs 
a public URL to send webhooks to — the localhost:8000 
is not publicly accessible.

FIX 4 — Add conversation view to dashboard

In the Evaluation tab add a new section below the 
campaign performance table called 
"WhatsApp Conversations":

Show a table with columns:
- Member
- Gap measure
- Outreach sent date
- Conversation state (with colored badge)
- Appointment date (if confirmed)
- Follow up status
- Last reply

Color code the state badges:
- OUTREACH_SENT — gray
- AWAITING_DATE — amber (member said yes, waiting for date)
- DATE_CONFIRMED — teal (appointment scheduled)
- FOLLOW_UP_SENT — blue (checking in)
- COMPLETED — green (gap closed via conversation)
- DECLINED — red (member declined)
- ESCALATED — red pulsing (needs care manager)

This table reads from GET /conversations/{run_id} 
and auto-refreshes every 30 seconds so replies 
appear in real time during the demo.

After all changes restart uvicorn and confirm:
1. POST /webhook/whatsapp endpoint exists and returns 200
2. New whatsapp_conversations table created in database
3. Conversation state machine logic is correct
4. Show me instructions for setting up ngrok
