# Ama Proxy — Deploy Instructions

## What this is
A tiny Vercel Edge Function that proxies requests from shirakates.com to the Anthropic API.
Your API key stays server-side. Visitors never see it.

---

## Step 1: Get your Anthropic API key
1. Go to console.anthropic.com
2. Click API Keys → Create Key
3. Copy it — you'll need it in Step 3

---

## Step 2: Deploy to Vercel
Option A — Vercel CLI (fastest):
```bash
npm i -g vercel
cd ama-proxy
vercel
```
Follow the prompts. When asked about the framework, choose "Other".
Vercel will give you a URL like: https://ama-proxy-abc123.vercel.app

Option B — Vercel Dashboard:
1. Go to vercel.com → New Project
2. Import from GitHub OR drag this folder in
3. Deploy

---

## Step 3: Add your API key as an environment variable
1. In your Vercel project dashboard → Settings → Environment Variables
2. Add:
   - Name:  ANTHROPIC_API_KEY
   - Value: sk-ant-... (your key from Step 1)
3. Save, then go to Deployments → Redeploy (so the variable takes effect)

---

## Step 4: Update the widget with your Vercel URL
In ama-widget.html, find this line:
```
const PROXY_URL = 'YOUR_VERCEL_URL/api/chat';
```
Replace YOUR_VERCEL_URL with your actual URL, e.g.:
```
const PROXY_URL = 'https://ama-proxy-abc123.vercel.app/api/chat';
```

---

## Step 5: Add Ama to your site
Copy everything from ama-widget.html and paste it into index.html
just before the closing </body> tag.

---

## Step 6: Test
Open shirakates.com, click the orange chat button bottom-right,
and ask Ama something. She should respond within a second or two.

---

## Costs
Anthropic API: roughly $0.003 per conversation (claude-sonnet-4-6).
At 100 conversations/month that's about $0.30. Basically free.
Vercel: free tier covers this easily.

---

## If something breaks
- "Failed to fetch": check that PROXY_URL matches your Vercel URL exactly
- 500 error: check that ANTHROPIC_API_KEY is set in Vercel env vars
- CORS error: check that your site domain matches the allowed origin in api/chat.js
  (currently set to https://www.shirakates.com)
