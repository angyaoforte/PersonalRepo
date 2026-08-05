# Forte — Design Your Role Around AI

A small web app for the AI Upskilling program. Each person maps the ambitious version of their role in 6 months (what they do vs. what AI does), and a **live LLM advisor** pushes them to aim higher, finds gaps in their role, drafts the prerequisites, and generates a build-and-learn plan. Output copies straight into the `#ai-upskilling` Slack thread.

It's a static page plus **one serverless function** that calls the Anthropic API. The API key lives only in a Netlify environment variable — it is never sent to the browser.

---

## What you need
- An **Anthropic API key** (console.anthropic.com → API keys).
- A free **Netlify** account.
- This folder (ideally pushed to your company GitHub).

---

## Deploy (recommended: from GitHub — ~10 min)
1. Put this folder in a repo on the **company GitHub**.
2. In Netlify: **Add new site → Import an existing project → GitHub**, pick the repo. Leave build settings as detected (the `netlify.toml` handles it). Deploy.
3. Go to **Site configuration → Environment variables** and add:
   - `ANTHROPIC_API_KEY` = your key **(required)**
   - `ACCESS_CODE` = a shared team code, e.g. `forte2026` *(optional but recommended — see Security)*
   - `MODEL_FAST` and `MODEL_SMART` *(optional — only if you want to pin specific models; see Models)*
4. **Redeploy** (Deploys → Trigger deploy) so the new variables take effect.
5. Open the site URL, type a name, and confirm the "Live ideas" panel and the Generate buttons respond.

### Quick alternative (no GitHub): drag-and-drop
Go to **app.netlify.com/drop** and drag this whole folder in. Then set the environment variables (step 3) and redeploy. Works, but the Git route is cleaner and matches company policy for shared code.

---

## Security (read this)
- **The key is server-side only.** It sits in the Netlify env var and is used only inside `netlify/functions/assist.js`. It never appears in the page. Do not paste it into `index.html`.
- **Set `ACCESS_CODE`.** You're sharing the link with the team, but links leak. With a code set, the app refuses to answer without it (people type it once in the "Team code" box). This is your simplest protection against a stray link running up API cost.
- **`.env` is gitignored.** Never commit real keys. `.env.example` only documents the variable names.
- **Cost** is small — single-digit to low-double-digit dollars for a session of ~14 people. The function caps input sizes and output length. If you want a hard ceiling, set a monthly budget/alert in the Anthropic console.

## Models
Model IDs change over time. The defaults are `claude-3-5-haiku-latest` (fast, for live ideas) and `claude-3-5-sonnet-latest` (stronger, for gaps/prerequisites/plan). If a call errors with a model-not-found message, set `MODEL_FAST` / `MODEL_SMART` to model IDs your key can access (check your Anthropic console).

## Changing things
- **Wording / layout / languages:** edit `index.html` (all copy is in the `T` object near the top of the script; EN + ES).
- **What the AI says / how it pushes ambition:** edit the prompts in `netlify/functions/assist.js` (the `CONTEXT` block and each action's prompt). The ambition ladder and the real team-build examples live there.
- **Where output goes:** right now people copy the summary into Slack. If you later want it collected automatically, that's an added storage step (a Google Sheet or a database) — not built here to keep it light.

## How it works (one glance)
```
Browser (index.html)
   │  POST /.netlify/functions/assist   { action, role, rows, lang, code }
   ▼
Netlify Function (assist.js)   ← ANTHROPIC_API_KEY (env var, secret)
   │  calls Anthropic Messages API
   ▼
returns JSON → the page renders it
```
Actions: `extract` (role → parts), `ideas` (live suggestions), `gaps` (blind spots), `prereqs` (what needs to be true), `plan` (build + learn).
