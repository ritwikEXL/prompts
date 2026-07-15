Simplify all message delivery to use WhatsApp 
only for now. Remove SMS and email delivery 
attempts entirely for the current demo build.

STEP 1 — Update delivery logic in api.py

For POST /send/message/{contact_id}:
Regardless of what channel is assigned 
(EMAIL, SMS, CALL, Spanish SMS, Mandarin SMS)
always send via WhatsApp using Twilio sandbox.

Use:
from_ = os.getenv('TWILIO_WHATSAPP_NUMBER')
to = 'whatsapp:' + os.getenv('TEST_SMS_NUMBER')

Remove all SendGrid email code entirely.
Remove all regular Twilio SMS code entirely.
Only keep the WhatsApp send logic.

STEP 2 — Update channel display in dashboard

In the outreach plan table change all 
channel badge labels to show:
- EMAIL → WhatsApp (Email)
- SMS → WhatsApp
- CALL → WhatsApp (Call Script)
- Spanish SMS → WhatsApp (Spanish)
- Mandarin SMS → WhatsApp (Mandarin)

Keep the same color coding but add 
WhatsApp icon to all channel badges.

STEP 3 — Update historical outreach records

Update all existing outreach plan records 
to show SENT status since we are treating 
all channels as WhatsApp for the demo:

UPDATE fact_nba_outreach_plan 
SET status = 'SENT',
sent_at = datetime('now', '-' || 
(abs(random()) % 14 + 1) || ' days')
WHERE status IN ('PLANNED', 'FAILED');

STEP 4 — Test WhatsApp delivery

Call POST /send/message/ for one contact 
from the latest run and confirm:
1. WhatsApp message arrives on test phone
2. Status updates to SENT in database
3. No email or SMS errors appear

STEP 5 — Update GET /test/email and 
GET /test/whatsapp endpoints

Remove GET /test/email endpoint or make 
it return a message saying email delivery 
is disabled in demo mode.

Keep GET /test/whatsapp working as before.

Add a note in the dashboard outreach plan 
section saying:
"Demo mode: all outreach delivered via 
WhatsApp. Production deployment supports 
Email, SMS, and IVR calls."

After changes confirm:
1. No SendGrid or Twilio SMS errors
2. WhatsApp delivery working for all 
   channel types
3. All historical records showing SENT
4. Dashboard outreach table shows 
   WhatsApp badges correctly
