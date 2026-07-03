Fix two issues in api.py and dashboard/index.html:

FIX 1 — Channel assignment must respect member preference

In the POST /send/message/{contact_id} endpoint and 
in the outreach plan generation logic, add a channel 
validation step:

Before sending any message, check the member's 
preferred_channel from dim_member_channel_pref.
If the assigned channel in fact_nba_outreach_plan 
does not match what the member prefers AND the 
member has consented to their preferred channel, 
override the assigned channel with the member's 
preferred channel.

Priority order for channel selection:
1. If preferred_channel is EMAIL and email_allowed 
   is true — use EMAIL
2. If preferred_channel is SMS and sms_allowed 
   is true — use WhatsApp SMS
3. If preferred_channel is CALL — schedule call
4. If preferred_channel is NONE or do_not_contact 
   is true — skip this contact entirely and mark 
   as SUPPRESSED not FAILED

Log any channel override in the audit trace:
"Channel overridden for [member_key]: 
assigned [original] → sending via [actual] 
based on member preference"

FIX 2 — Handle failed contacts gracefully

When a message fails to send:
- Catch the specific error from Twilio or SendGrid
- Store the error reason in the database
- Mark status as FAILED with the error reason visible
- In the dashboard show a red Failed badge with 
  a tooltip showing the error reason when hovered
- Add a Retry button next to failed contacts 
  that calls POST /send/message/{contact_id} again
- Never mark a suppressed Do Not Contact member 
  as FAILED — mark them as SUPPRESSED with a 
  different gray badge

FIX 3 — Add channel override summary to audit trace

After the outreach plan is generated add a trace 
entry showing:
- How many contacts were assigned their preferred channel
- How many were overridden to a different channel
- How many were suppressed due to Do Not Contact flag
- Format: "Channel optimization: X preferred match, 
  Y overridden, Z suppressed"

After making fixes:
1. Restart uvicorn
2. Run a new session in automated mode
3. Show me the delivery results — how many sent, 
   failed, suppressed
4. Show me the audit trace entries for channel 
   optimization
