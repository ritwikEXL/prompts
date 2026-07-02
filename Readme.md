Update api.py to add a fallback message template 
for the POST /send/message/{contact_id} endpoint 
so it works without an Anthropic API key.

Instead of calling the Claude API to generate 
the message, generate the message text using 
this template logic:

1. Read the contact details from fact_nba_outreach_plan
2. Read member details from dim_member for that contact
3. Read gap details from fact_member_gap for that contact
4. Generate message text based on channel and language:

For EMAIL channel use this template:
Subject: Important Health Reminder from Your Medicare Plan

Dear [member_key],

This is a friendly reminder that you have an outstanding 
[measure_name] that needs to be completed this year.

As a valued member of your Medicare Advantage plan, 
your health is our priority. Completing this important 
health check helps us ensure you receive the best 
possible care.

As a thank you for completing this health activity, 
you will receive [incentive_offered].

Please contact your primary care provider to schedule 
your appointment at your earliest convenience.

If you have any questions please call us at 
1-800-MEDICARE.

Thank you for taking care of your health.

Warm regards,
CareIntel Health Outreach Team

For SMS channel use this template (under 160 characters):
Hi [member_key], reminder: complete your [measure_code] 
this year and receive [incentive_offered]. 
Call 1-800-MEDICARE to schedule. Reply STOP to opt out.

For Mandarin SMS use this template:
您好 [member_key], 提醒您今年完成[measure_code]检查, 
可获得[incentive_offered]。请致电1-800-MEDICARE预约。
回复STOP退订。

For Spanish SMS use this template:
Hola [member_key], recordatorio: complete su 
[measure_code] este año y reciba [incentive_offered]. 
Llame al 1-800-MEDICARE para programar. 
Responda STOP para cancelar.

For CALL channel use the same SMS template since 
we are sending to a test SMS number.

Replace [member_key], [measure_name], [measure_code], 
and [incentive_offered] with actual values from 
the database for that contact.

After generating the message text proceed with 
sending via SendGrid for EMAIL or Twilio for SMS 
exactly as before.

Add a note in the API response indicating 
"message_source: template" so we know it used 
the fallback template rather than Claude generation.

Later when ANTHROPIC_API_KEY is available in .env 
the code should automatically switch to Claude 
generated messages instead of the template. 
Check if ANTHROPIC_API_KEY exists in environment 
variables — if yes use Claude, if no use template.

After updating restart uvicorn and test by calling 
POST /send/message/ for the first PLANNED contact 
in the latest run. Show me the generated message 
text and confirm it sent successfully.
