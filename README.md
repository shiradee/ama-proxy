# Ama — Deploy Guide (v2, corrected)

## The mental model (read this once)
Two separate things live in two separate places:

1. **Your website** — static files on GitHub Pages at shirakates.com. GitHub Pages
   can't run server code, so it can't hold your secret API key.
2. **The Ama proxy** — one tiny serverless function on **Vercel**. It holds your
   API key server-side and talks to Anthropic. Visitors never see the key.

The widget on your site calls the Vercel function. That's the whole architecture.

> What broke last time: the old `vercel.json` declared `"runtime": "edge"`, which
> Vercel rejects at deploy time. **This version has no `vercel.json` at all** — one
> less thing to break. If you still have the old `vercel.json`, delete it.

---

## Part A — Deploy the proxy to Vercel

You only need the folder `ama-proxy/` (it contains just `api/chat.js`).

### A1. Get your Anthropic API key
console.anthropic.com → API Keys → Create Key → copy it (starts with `sk-ant-...`).

### A2. Deploy the folder
**Easiest (dashboard, no terminal):**
1. vercel.com → **Add New… → Project**
2. Choose **"Deploy a folder"** / drag the `ama-proxy` folder in (or import it from
   a GitHub repo if you push it there). Framework preset: **Other**.
3. Click **Deploy**. Vercel gives you a URL like
   `https://ama-proxy-xxxx.vercel.app`. **Copy it.**

**Or terminal:**
```bash
npm i -g vercel
cd ama-proxy
vercel
```
Framework: **Other**. Copy the URL it prints.

### A3. Add your API key as an environment variable
In the Vercel project → **Settings → Environment Variables** → add:
- **Name:** `ANTHROPIC_API_KEY`
- **Value:** `sk-ant-...` (your key)
- Apply to: Production (and Preview is fine too)

Save.

### A4. Redeploy so the key takes effect
Deployments tab → latest deployment → **⋯ → Redeploy**.
(Env vars only apply to deployments created *after* they're added — this step is
the one people skip.)

### A5. Quick sanity check (optional but smart)
In a browser, open: `https://YOUR-VERCEL-URL/api/chat`
You should see `{"error":"Method not allowed"}`. That's correct — it means the
function is alive (it only accepts POST). A 404 here means the file isn't at
`api/chat.js`; a different error usually means the key isn't set.

---

## Part B — Add Ama to your site

### B1. Point the widget at your proxy
Open `ama-widget.html`. Find this line near the bottom:
```js
const PROXY_URL = 'YOUR_VERCEL_URL/api/chat';
```
Replace it with your real URL, e.g.:
```js
const PROXY_URL = 'https://ama-proxy-xxxx.vercel.app/api/chat';
```
(Keep the `/api/chat` on the end.)

### B2. Paste the widget into your site
Copy **everything** in `ama-widget.html` and paste it into `index.html`
**right before the closing `</body>` tag.**

### B3. Push to GitHub
Commit + push your updated `index.html`. GitHub Pages redeploys in ~1 minute.

### B4. Test
Open shirakates.com. Within a few seconds the **"Ask Ama"** pill animates in
bottom-right, and a little nudge bubble pops up. Click it, ask a question, and
Ama should answer in a second or two.

---

## Troubleshooting

| Symptom | Cause | Fix |
|---|---|---|
| Vercel deploy fails on "runtime" | leftover old `vercel.json` | delete `vercel.json` — it's not needed |
| `/api/chat` returns 404 | file isn't at `api/chat.js` | confirm the path is exactly `api/chat.js` |
| Ama says "Connection hiccup" | `PROXY_URL` typo | must match your Vercel URL exactly, ending in `/api/chat` |
| Ama says "Something went sideways" | server-side error (usually the key) | confirm `ANTHROPIC_API_KEY` is set, then **redeploy** |
| Browser console shows a CORS error | site served from an origin the proxy doesn't allow | this version already allows both `shirakates.com` and `www.shirakates.com`; if you use a *different* domain, add it to `ALLOWED_ORIGINS` in `api/chat.js` |
| 401 / authentication_error in response | bad or expired API key | regenerate the key, update the Vercel env var, redeploy |

---

## Cost
Roughly $0.003 per conversation on claude-sonnet-4-6. ~100 chats/month ≈ $0.30.
Vercel's free tier covers this easily.
