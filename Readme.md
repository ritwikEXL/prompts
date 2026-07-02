Create a .env file in the project root with these 
placeholders and add .env to .gitignore so it never 
gets pushed to GitHub:

TWILIO_ACCOUNT_SID=paste_your_sid_here
TWILIO_AUTH_TOKEN=paste_your_token_here
TWILIO_PHONE_NUMBER=paste_your_twilio_number_here
SENDGRID_API_KEY=paste_your_sendgrid_key_here
SENDGRID_SENDER_EMAIL=paste_your_verified_sender_email_here
TEST_SMS_NUMBER=paste_your_personal_mobile_here
TEST_EMAIL=paste_your_test_email_here

Also install python-dotenv, twilio, and sendgrid 
packages using pip.

Confirm .env is in .gitignore and packages installed.
