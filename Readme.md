Both tunnels are live
Dashboard URL:  https://andrews-looked-metallica-symbols.trycloudflare.com
API URL:        https://hebrew-steps-emotional-juice.trycloudflare.com
Share the Dashboard URL with Ankit and Rishi — that's all they need to open in any browser on any device.

What was done:

Step	Result
cloudflared.exe	Downloaded (40.6 MB), method: PowerShell
API tunnel	https://hebrew-steps-emotional-juice.trycloudflare.com → port 8000
Dashboard tunnel	https://andrews-looked-metallica-symbols.trycloudflare.com → port 8080
API baked into dashboard	dashboard/index.html updated with API tunnel URL
External API check	5 plans returned correctly over public URL
Dashboard check	Loads, title confirmed, API URL confirmed present
Important notes for Ankit and Rishi:

These are temporary Cloudflare Quick Tunnels — URLs are valid only while this machine is running and the tunnel processes are active
If the machine restarts or you close this session, new URLs will be generated (run the same cloudflared commands again)
Tunnel logs are at tunnel_api.log and tunnel_dashboard.log in the project folder if anything needs debuggin
