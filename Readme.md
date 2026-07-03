Make the following changes to dashboard/index.html 
and api.py:

CHANGE 1 — Integrate Send into Outreach confirmation

In the Run Session tab Phase 4 Outreach Agent:

For MANUAL MODE:
Keep the current behavior — show the outreach plan 
table with individual Send buttons and a Send All 
button. The user reviews and sends separately after 
confirming the plan. Add a clear label above the 
table saying "Review your outreach plan below. 
Click Send All when ready to deliver messages."

For AUTOMATED MODE:
After the Outreach Agent writes the plan, 
automatically trigger POST /send/all/{run_id} 
immediately after the plan is confirmed without 
any user interaction. Show a real time delivery 
status panel that updates as each message is sent:

Show a delivery progress section with:
- A progress bar showing X of Y messages sent
- Each contact row updating from PLANNED to 
  SENT in real time as messages go out
- A small WhatsApp icon for WhatsApp messages 
  and email icon for email messages
- Green checkmark when delivered
- Red X if failed with error reason

After all messages sent show a final summary:
"Session complete — X WhatsApp messages sent, 
Y emails sent, Z calls scheduled"

CHANGE 2 — Call scheduling instead of real calls

For any contact with channel = CALL, Spanish Call, 
or Mandarin Call:
- Do not attempt to make a real phone call
- Instead mark status as SCHEDULED 
  (different from PLANNED and SENT)
- Generate the call script using the same 
  template as SMS but formatted for a phone 
  conversation with greeting and closing
- Show a phone icon with SCHEDULED badge 
  in amber color (different from green SENT)
- In the message preview show the full call 
  script that would be read to the member
- In the audit trace log: 
  "Call scheduled for [member] — 
  script generated, pending care manager assignment"

CHANGE 3 — Message preview before Send in manual mode

In the Outreach Plan table, before the user clicks 
Send on any row, show a preview of the message 
that will be sent. When the outreach plan is first 
generated, automatically call the message generation 
logic for each contact and show the preview text 
in an expandable row below each contact. 
Label it "Message Preview" in gray italic text.
The user can read the preview and then click Send 
to deliver.

CHANGE 4 — Delivery status persistence

After messages are sent, store the delivery 
timestamp and status in the database. When the 
user comes back to a previous session via 
Session History, show the actual sent messages 
with their delivery status and timestamps rather 
than showing PLANNED status for already sent messages.

Update the GET /session/{run_id}/status endpoint 
to return sent_at timestamp and generated_message 
for each contact row.

Keep all existing visual design unchanged.
