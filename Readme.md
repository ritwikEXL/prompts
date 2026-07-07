Install pyngrok using pip with --break-system-packages flag.
Then update api.py to automatically create an ngrok tunnel 
when the server starts.

Add this at the top of api.py after the imports:

from pyngrok import ngrok as pyngrok_tunnel

Add this function and call it on startup:

@app.on_event("startup")
async def startup_event():
    public_url = pyngrok_tunnel.connect(8000)
    print(f"\n{'='*50}")
    print(f"NGROK PUBLIC URL: {public_url}")
    print(f"Webhook URL: {public_url}/webhook/whatsapp")
    print(f"Copy this URL to Twilio WhatsApp Sandbox Settings")
    print(f"{'='*50}\n")

Restart uvicorn and show me the public URL that 
appears in the console output.
