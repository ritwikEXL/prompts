Replace SendGrid email delivery with Gmail SMTP 
for simplicity and reliability in the demo environment.

Remove all SendGrid code and dependencies.
Remove sendgrid from requirements.txt.

Add these to .env:
GMAIL_ADDRESS=ritwik.sharan308@gmail.com
GMAIL_APP_PASSWORD=pywr gncj ahoi dnnv


Update the send email function in api.py:

import smtplib
from email.mime.multipart import MIMEMultipart
from email.mime.text import MIMEText

def send_email_gmail(to_email: str, 
                     subject: str, 
                     body: str) -> dict:
    try:
        msg = MIMEMultipart('alternative')
        msg['Subject'] = subject
        msg['From'] = os.getenv('GMAIL_ADDRESS')
        msg['To'] = to_email
        
        html_body = f"""
        <html><body>
        <div style="font-family: Arial; 
                    max-width: 600px; margin: auto;">
            <div style="background: #F15A22; 
                        padding: 20px; 
                        color: white;">
                <h2>CareIntel Health Reminder</h2>
            </div>
            <div style="padding: 20px;">
                {body.replace(chr(10), '<br>')}
            </div>
            <div style="padding: 20px; 
                        color: gray; font-size: 12px;">
                This message is from your Medicare 
                Advantage health plan. To unsubscribe 
                reply STOP.
            </div>
        </div>
        </body></html>
        """
        
        msg.attach(MIMEText(body, 'plain'))
        msg.attach(MIMEText(html_body, 'html'))
        
        with smtplib.SMTP_SSL('smtp.gmail.com', 465) as server:
            server.login(
                os.getenv('GMAIL_ADDRESS'),
                os.getenv('GMAIL_APP_PASSWORD')
            )
            server.send_message(msg)
        
        return {
            "success": True,
            "channel": "email",
            "delivered_to": to_email,
            "provider": "Gmail SMTP"
        }
    except Exception as e:
        return {
            "success": False,
            "error": str(e)
        }

Update POST /send/message/{contact_id} to:
- Use send_email_gmail for EMAIL channel
- Keep WhatsApp Twilio for all other channels

Update GET /test/email to use Gmail SMTP.

Add GMAIL_ADDRESS and GMAIL_APP_PASSWORD 
to .env.example

Test by calling GET /test/email and confirm 
email arrives in inbox (not spam) within 
30 seconds.
