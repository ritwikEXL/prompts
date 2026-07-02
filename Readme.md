Read the .env file in the project root and build a 
complete SMS and email delivery layer.

STEP 1 — Install required packages
Run these commands:
pip install twilio sendgrid python-dotenv

STEP 2 — Update api.py to add delivery endpoints

Import and load the .env file at the top of api.py 
using python-dotenv. Load these variables:
TWILIO_ACCOUNT_SID, TWILIO_AUTH_TOKEN, 
TWILIO_PHONE_NUMBER, SENDGRID_API_KEY, 
SENDGRID_SENDER_EMAIL, TEST_SMS_NUMBER, TEST_EMAIL

Add these new endpoints to api.py:

POST /send/message/{contact_id}
This is the main send endpoint. It:
1. Reads the contact row from fact_nba_outreach_plan 
   in careintel.db using the contact_id
2. Reads the member details from dim_member joined 
   with dim_member_channel_pref for that contact
3. Reads the gap details from fact_member_gap for 
   that contact
4. Calls the Claude API using model claude-sonnet-4-6 
   with max_tokens 300 to generate a personalized 
   outreach message using this system prompt:
   "You are a healthcare outreach specialist writing 
   member communications for a Medicare Advantage plan. 
   Write warm, plain-language, respectful messages. 
   Keep messages under 160 characters for SMS and 
   under 200 words for email. Always include a clear 
   call to action."
   And this user prompt:
   "Write a {channel} message for {member_name}, 
   a {age_band} year old member who needs to complete 
   {measure_name}. Their preferred language is 
   {language_preference}. The incentive being offered 
   is {incentive}. Gap has been open for {days_open} 
   days. Tone should be warm and encouraging."
5. Based on the channel field in the contact row:
   - If channel is EMAIL: send via SendGrid to 
     TEST_EMAIL using SENDGRID_API_KEY with sender 
     SENDGRID_SENDER_EMAIL. Subject line should be 
     "Important health reminder from your Medicare plan"
   - If channel is SMS or CALL: send via Twilio to 
     TEST_SMS_NUMBER using TWILIO_ACCOUNT_SID and 
     TWILIO_AUTH_TOKEN from TWILIO_PHONE_NUMBER
   - If channel is Mandarin SMS or Spanish SMS or 
     Spanish Call: still send to TEST_SMS_NUMBER 
     but generate the message in the appropriate 
     language
6. After sending successfully update the contact 
   status in fact_nba_outreach_plan from PLANNED 
   to SENT and store the generated message text 
   in a new column called generated_message
7. Return a response with:
   - success: true or false
   - channel: which channel was used
   - message_text: the generated message
   - delivered_to: the test contact used
   - status: SENT or FAILED

POST /send/all/{run_id}
Sends all PLANNED contacts for a given run_id 
by calling POST /send/message/{contact_id} for 
each contact in sequence. Returns a summary of 
how many were sent successfully and how many failed.

GET /messages/{run_id}
Returns all contacts for a run_id including their 
generated_message text and current status so the 
dashboard can display what was sent.

STEP 3 — Add generated_message column to database

Run an ALTER TABLE statement on fact_nba_outreach_plan 
in careintel.db to add a generated_message TEXT column 
if it does not already exist. Also add a sent_at 
TIMESTAMP column to record when the message was sent.

STEP 4 — Update dashboard/index.html

In the Outreach Plan table add two things per row:

1. A "Send" button on each contact row that calls 
   POST /send/message/{contact_id} when clicked.
   While sending show a loading spinner on the button.
   On success change the button to a green "Sent" 
   badge and update the status column to SENT.
   On failure show a red "Failed" badge.

2. A message preview expand button (a small arrow 
   or eye icon) next to the Send button. When clicked 
   it expands a row below showing the generated 
   message text that was sent. If not yet sent show 
   "Click Send to generate and preview message."

Also add a "Send All" button at the top of the 
Outreach Plan table that calls POST /send/all/{run_id} 
for all planned contacts at once. Show a progress 
indicator while sending. When complete show a 
summary banner: "X messages sent successfully"

STEP 5 — Test the full flow

After making all changes:
1. Restart the uvicorn server
2. Call POST /send/message/ for the first contact 
   in the latest run
3. Show me the response including the generated 
   message text
4. Confirm the status updated to SENT in the database
5. Show me the generated message that would have 
   been sent

Do not hardcode any phone numbers or email addresses 
in api.py — always read from the .env file.
