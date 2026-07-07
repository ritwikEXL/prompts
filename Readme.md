Since ngrok is blocked by Windows Defender on this 
corporate sandbox, implement a WhatsApp conversation 
simulator in the dashboard instead of a real webhook.

This gives the same demo experience without needing 
a public URL.

CHANGE 1 — Add conversation simulator to Evaluation tab

In the WhatsApp Conversations section of the 
Evaluation tab, add a simulator panel for each 
conversation row.

When the user clicks "Simulate Reply" on any 
conversation row, show an inline conversation 
thread panel that looks like a WhatsApp chat:

Left side — member messages (gray bubbles)
Right side — CareIntel messages (teal bubbles)

Show the initial outreach message that was sent 
as the first CareIntel bubble.

Then show 4 action buttons the demo operator 
can click to simulate member replies:

Button 1 — "Member replies: Yes I will go"
Button 2 — "Member replies: No not now"  
Button 3 — "Member replies: [Date - July 15]"
Button 4 — "Member replies: Yes completed"

When Button 1 is clicked:
- Add gray bubble "Yes I will go" to conversation
- Call POST /conversations/simulate with 
  body: {contact_id, reply: "yes"}
- Backend processes through state machine exactly 
  as webhook would
- Add teal bubble with system response asking for date
- Show Button 3 as next available action
- Hide Buttons 1 and 2

When Button 3 is clicked:
- Add gray bubble "July 15" to conversation
- Call POST /conversations/simulate with 
  body: {contact_id, reply: "july 15"}
- Backend processes date and schedules follow up
- Add teal bubble confirming appointment
- Show "Fast forward to follow up" button that 
  triggers the follow up message immediately 
  without waiting for the actual date

When "Fast forward to follow up" is clicked:
- Add teal bubble with follow up message 
  "Did you complete your appointment?"
- Show Button 4 as next available action

When Button 4 is clicked:
- Add gray bubble "Yes completed"
- Call POST /conversations/simulate with 
  body: {contact_id, reply: "yes completed"}
- Backend closes the gap in fact_member_gap
- Add teal bubble "Wonderful news! Gap closed."
- Update conversation state to COMPLETED
- Update the campaign evaluation automatically
- Show green COMPLETED badge on the row
- Trigger campaign re-evaluation

CHANGE 2 — Add POST /conversations/simulate endpoint to api.py

This endpoint takes a contact_id and a reply text 
and processes it through the exact same state machine 
logic as POST /webhook/whatsapp would.

This means the database updates, gap closures, 
and evaluation triggers are all real — only the 
input is simulated rather than coming from Twilio.

Return the full conversation state and the 
system response message so the dashboard can 
display it.

CHANGE 3 — Real time conversation display

The conversation thread should feel like a real 
WhatsApp chat:
- Bubbles appear with a slight fade in animation
- Timestamps shown on each bubble
- Member name shown above gray bubbles
- CareIntel shown above teal bubbles
- Scrollable if conversation gets long
- Show typing indicator (3 animated dots) for 
  1 second before each system response appears

CHANGE 4 — Conversation summary in campaign row

After a conversation is completed update the 
campaign row to show:
- Conversation outcome: Completed / Declined / 
  Escalated with appropriate color badge
- Appointment date that was confirmed
- Days from outreach to completion
- A small WhatsApp icon indicating this gap 
  was closed via conversational outreach

This tells the full story in the campaign view 
without needing to expand the conversation thread.

Keep all existing visual design unchanged.
