Host CareIntel publicly using Cloudflare Tunnel
so Ankit and Rishi can access it from their laptops.

STEP 1 — Download cloudflared
Try downloading using PowerShell:

Invoke-WebRequest -Uri "https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-windows-amd64.exe" -OutFile "cloudflared.exe"

If that fails try curl:
curl -L -o cloudflared.exe https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-windows-amd64.exe

If both fail try downloading via Python:
import urllib.request
urllib.request.urlretrieve(
    "https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-windows-amd64.exe",
    "cloudflared.exe"
)

Show me which method worked and confirm 
cloudflared.exe exists in the project folder.

STEP 2 — Start API tunnel
Run the tunnel for FastAPI backend on port 8000:
.\cloudflared.exe tunnel --url http://localhost:8000

Capture the public URL it generates.
It will look like:
https://random-words.trycloudflare.com

Show me the API tunnel URL.

STEP 3 — Update dashboard API_BASE
Once you have the API tunnel URL update 
dashboard/index.html:

Find the line that says:
const API_BASE = ...

Replace with:
const API_BASE = 'https://[api-tunnel-url]';

Where [api-tunnel-url] is the actual URL 
from Step 2.

STEP 4 — Start dashboard tunnel
Run a second tunnel for the dashboard on port 8080:
.\cloudflared.exe tunnel --url http://localhost:8080

Capture this public URL as well.

STEP 5 — Verify everything works
Test both URLs:

1. Open the dashboard public URL in browser
   Confirm it loads the CareIntel dashboard
   
2. Check API is accessible:
   curl https://[api-tunnel-url]/opportunities
   Confirm it returns JSON data

3. Confirm API Connected indicator shows 
   green in the dashboard

4. Load the Opportunities tab and confirm 
   UHC Medicare Signature shows as default 
   with correct data

STEP 6 — Show me both URLs
Display both public URLs clearly:

Dashboard URL: https://xxx.trycloudflare.com
API URL: https://yyy.trycloudflare.com

These are what I will share with Ankit and Rishi.

Important: Both tunnels must stay running 
for the URLs to work. Keep them running 
in background processes.
