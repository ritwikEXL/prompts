Prepare CareIntel for deployment on Render.com.

STEP 1 — Make sure seed_demo_data.py exists 
and works completely standalone.

Run it now and show me the output summary.
If it does not exist create it with all 
demo data — plans, members, gaps, runs, 
evaluations, conversations.

STEP 2 — Create requirements.txt

Run: pip freeze > requirements.txt_full

Then create a clean requirements.txt 
with only what we need:
fastapi
uvicorn
python-dotenv
twilio
openai
httpx
pydantic

STEP 3 — Create Procfile
web: uvicorn api:app --host 0.0.0.0 --port $PORT

STEP 4 — Update api.py for Render

Make sure api.py reads PORT from environment:
import os
port = int(os.getenv('PORT', 8000))

Make sure DB_PATH reads from environment:
DB_PATH = os.getenv('DB_PATH', 'careintel.db')

Add startup seeding:
@app.on_event("startup")
async def startup_event():
    if not os.path.exists(DB_PATH) or \
       os.path.getsize(DB_PATH) < 10000:
        print("Seeding database...")
        import subprocess
        subprocess.run(
            ['python', 'seed_demo_data.py']
        )
        print("Database seeded")

STEP 5 — Create .gitignore
.env
careintel.db
__pycache__/
*.pyc
cloudflared.exe
*.log
.DS_Store

STEP 6 — Handle dashboard hosting separately

Copy dashboard/index.html to docs/index.html

In docs/index.html update API_BASE:
const API_BASE = 'RENDER_API_URL_PLACEHOLDER';

We will replace RENDER_API_URL_PLACEHOLDER 
with the actual Render URL after deployment.

STEP 7 — Push to GitHub

Stage all files:
git add .
git status

Show me what files are being added.
Then commit:
git commit -m "Deploy CareIntel to Render - demo ready for stakeholder testing"

Then push:
git push origin main

Show me the push output.

STEP 8 — Create .env.example

Create .env.example with placeholder values:
OPENROUTER_API_KEY=your_openrouter_key
TWILIO_ACCOUNT_SID=your_twilio_sid
TWILIO_AUTH_TOKEN=your_twilio_token
TWILIO_WHATSAPP_NUMBER=whatsapp:+14155238886
TEST_SMS_NUMBER=+1xxxxxxxxxx
TEST_EMAIL=your@email.com
DB_PATH=careintel.db

After all steps show me:
1. seed_demo_data.py summary output
2. requirements.txt contents
3. Procfile contents
4. Confirm git push succeeded
5. GitHub repo URL
