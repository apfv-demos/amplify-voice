---
name: deploy-webcall-demo
description: Deploy a secure voice agent demo website using the Retell Voice Demo template. Includes Google auth, reCAPTCHA, daily quotas, and a polished UI with blob animation. Use when the user mentions deploying a Retell demo site, creating a voice agent website, setting up a web call demo, Retell web call template, voice demo with authentication, demo site for Retell agents, or building a website to demo their voice AI agent.
---

# Retell Voice Demo — Deployment Skill

Guide the user through deploying a complete voice agent demo website. The site lets visitors sign in with Google and talk to a Retell AI agent, with abuse prevention built in (auth, reCAPTCHA, daily quotas).

**Audience: Assume beginner.** Explain each step. Verify success before proceeding. If something fails, diagnose before retrying. Never leave the user stuck — always offer an alternative path.

## Phase 1: Overview and Prerequisites

Start by explaining what they are building:

> This template gives you a polished demo website for your Retell AI voice agent. Visitors sign in with Google, click "Start Call," and talk to your agent in the browser. Three layers of protection prevent abuse:
>
> 1. **Google Auth** — only signed-in users can call
> 2. **reCAPTCHA v3** — silently blocks bots (no puzzles for real users)
> 3. **Daily quotas** — configurable limit per user (default: 5 calls/day, 2 minutes each)
>
> The site includes a beautiful animated blob that pulses when the agent is speaking, plus call logging in Supabase so you can see who used your demo.

Then check prerequisites one at a time. Ask the user to confirm each before continuing:

**1. Claude Desktop**
- Download at https://claude.com/download
- This is the tool that runs the deployment skill — it is required
- A Pro or Team plan is recommended for the best experience

**2. Retell AI account with at least one agent configured**
- Sign up at https://www.retellai.com if needed
- They need an existing agent — the template connects to it, it does not create one
- If they do not have an agent yet, tell them to create one first and come back

**3. Google account**
- Needed for Google Cloud Console (https://console.cloud.google.com) and reCAPTCHA admin (https://www.google.com/recaptcha/admin)
- Any Gmail or Google Workspace account works

**4. Supabase account (free)**
- Sign up at https://supabase.com if needed
- Used for Google authentication and call logging/quotas

**5. Node.js v20 or later**
Check by running:
```bash
node -v
```
- If not installed or below v20: direct them to https://nodejs.org/ and have them download the LTS version

**6. A GitHub account**
- Needed to host the code and deploy to Vercel

**7. A Vercel account**
- Sign up at https://vercel.com (free tier works fine)
- Best to sign up with GitHub for seamless integration

Tell the user that Google Cloud OAuth and reCAPTCHA will be configured during the process — they just need the Google account ready.

Do NOT proceed until items 1–7 are confirmed.

## Phase 2: Supabase Project Setup

### 2a. Create the Supabase project

Walk the user through this:
1. Go to https://supabase.com/dashboard
2. If prompted to create an organization first, create one (any name works, choose the free plan)
3. Click **New project**
4. Give it a name (e.g., "retell-voice-demo")
4. Choose a database password — tell them to save it somewhere (they will not need it again for this template, but it is good practice)
5. Select the region closest to them
6. Click **Create new project** and wait for it to finish provisioning

### 2b. Get the API keys

Once the project is ready:
1. Go to **Settings > API** (in the sidebar)
2. Copy these three values and tell the user to save them in a notepad:
   - **Project URL** → this becomes `NEXT_PUBLIC_SUPABASE_URL`
   - **anon / public key** (under "Project API keys") → `NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY`
   - **service_role key** (click "Reveal" — keep this secret) → `SUPABASE_SERVICE_ROLE_KEY`

> The anon key is safe to expose to the browser — it can only do what Row Level Security allows. The service_role key bypasses RLS and should never be exposed to the client.

### 2c. Run the database migration

1. Go to **SQL Editor** in the Supabase sidebar
2. Click **New query**
3. Paste this SQL and click **Run**:

```sql
CREATE TABLE public.call_logs (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  user_id UUID NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE,
  user_email TEXT,
  retell_call_id TEXT,
  started_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  status TEXT NOT NULL DEFAULT 'initiated'
);

CREATE INDEX idx_call_logs_user_day
  ON public.call_logs (user_id, started_at);

ALTER TABLE public.call_logs ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Users can view own call logs"
  ON public.call_logs FOR SELECT
  USING (auth.uid() = user_id);
```

Verify: the output should say "Success. No rows returned." If there is an error, check that they pasted the full block. See [troubleshooting.md](references/troubleshooting.md) for common issues.

### 2d. Note the Supabase Callback URL

1. Go to **Authentication > Providers** in the sidebar
2. Find **Google** in the list and expand it
3. Copy the **Callback URL (for OAuth)** shown there — it looks like `https://YOUR-PROJECT.supabase.co/auth/v1/callback`
4. Keep this tab open — they will come back to paste the Google OAuth credentials here after the next phase

## Phase 3: Google Cloud OAuth

The demo uses Google sign-in via Supabase. This requires setting up OAuth credentials in Google Cloud Console.

Walk the user through step by step:

### 3a. Select or create a Google Cloud project
1. Go to https://console.cloud.google.com
2. If prompted about a free trial or billing, they can skip or dismiss it — OAuth credentials are completely free and do not require billing to be enabled
3. Click the project dropdown (top-left, next to "Google Cloud")
4. If they already have a project, **select it** — no need to create a new one. They can add OAuth credentials to any existing project.
5. If they do not have any project, click **New Project**, give it a name, and click **Create**

### 3b. Configure the OAuth consent screen (if not already configured)
If the project already has an OAuth consent screen configured (from a previous setup), skip to 3c.

If not:
1. Go to **APIs & Services > OAuth consent screen** (sidebar)
2. Choose **External** user type → click **Create**
3. Fill in:
   - **App name**: the name of their demo (e.g., "Voice Agent Demo")
   - **User support email**: their email
   - **Developer contact information**: their email
4. Click **Save and Continue** through the remaining tabs (Scopes, Test Users) without changing anything
5. Click **Back to Dashboard**

### 3c. Create OAuth credentials
1. Go to **APIs & Services > Credentials** (sidebar)
2. Click **Create Credentials > OAuth Client ID**
3. Choose **Web application**
4. Under **Authorized redirect URIs**, click **Add URI** and paste the **Callback URL** from Supabase (Phase 2d)
5. Click **Create**
6. Copy the **Client ID** and **Client Secret** shown in the popup

### 3d. Add credentials to Supabase
1. Go back to the Supabase tab (Authentication > Providers > Google)
2. Toggle Google to **Enabled**
3. Paste the **Client ID** and **Client Secret** from Google Cloud
4. Click **Save**

Tell the user:
> Google OAuth is now connected. Users who sign in with Google on your demo site will be authenticated through Supabase.

## Phase 4: Google reCAPTCHA v3

reCAPTCHA v3 works silently in the background — users never see a puzzle. It assigns a score (0.0 to 1.0) to each interaction. The template requires a score of 0.5 or higher.

### 4a. Register a reCAPTCHA site
1. Go to https://www.google.com/recaptcha/admin
2. Click **+** (Create) to register a new site
3. Fill in:
   - **Label**: a name for this site (e.g., "Voice Demo")
   - **reCAPTCHA type**: choose **Score based (v3)**
   - **Domains**: add the domain they will deploy to (e.g., `your-app.vercel.app` — they can add this after deployment if they do not know yet)
4. Accept the terms and click **Submit**

> **Note**: If the "Submit" button appears grayed out, make sure the terms of service checkbox is checked. This is a common gotcha.

### 4b. Copy the keys
After registration:
- **Site Key** → `NEXT_PUBLIC_RECAPTCHA_SITE_KEY`
- **Secret Key** → `RECAPTCHA_SECRET_KEY`

Tell the user to save these with their other keys.

## Phase 5: Get the Retell Keys

If they have not already gathered these:

1. Go to https://www.retellai.com
2. **API Key**: Settings > API Keys → copy it (starts with `key_`)
3. **Agent ID**: Click on the agent they want to demo → copy the Agent ID from the URL or the agent page (starts with `agent_`)

They should now have all their keys saved:
- 3 Supabase values (URL, publishable key, service role key)
- 2 Retell values (API key, agent ID)
- 2 reCAPTCHA values (site key, secret key)

## Phase 6: Get the Code and Configure

### 6a. Clone the repository

Check if git is available:
```bash
git --version
```

**If git is available:**
```bash
git clone https://github.com/apfv-demos/retell-voice-demo.git
cd retell-voice-demo
```

**If git is NOT available:**
> Open https://github.com/apfv-demos/retell-voice-demo in your browser, click the green "Code" button, then "Download ZIP". Unzip it and open a terminal in that folder.

### 6b. Install dependencies
```bash
npm install
```
Wait for completion. If errors occur, see [troubleshooting.md](references/troubleshooting.md).

### 6c. Create the environment file

```bash
cp .env.example .env.local
```

On Windows, if `cp` does not work:
```powershell
copy .env.example .env.local
```

### 6d. Fill in the environment variables

Open `.env.local` in the user's editor and fill in all the values they collected:

```env
# Supabase
NEXT_PUBLIC_SUPABASE_URL=https://YOUR-PROJECT.supabase.co
NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY=eyJ...
SUPABASE_SERVICE_ROLE_KEY=eyJ...

# Retell AI
RETELL_API_KEY=key_...
RETELL_AGENT_ID=agent_...

# reCAPTCHA v3
NEXT_PUBLIC_RECAPTCHA_SITE_KEY=6Le...
RECAPTCHA_SECRET_KEY=6Le...

# Quotas (customize as needed)
NEXT_PUBLIC_MAX_CALLS_PER_DAY=5
NEXT_PUBLIC_MAX_CALL_DURATION_SECONDS=120

# Branding
NEXT_PUBLIC_APP_NAME=Voice Agent Demo
NEXT_PUBLIC_APP_DESCRIPTION=Talk to our AI agent!
```

Help the user paste in their values. Remind them:
- The `NEXT_PUBLIC_` variables are safe for the browser — they only control what the user sees
- The server enforces the actual limits regardless of what the client displays
- `SUPABASE_SERVICE_ROLE_KEY` and `RECAPTCHA_SECRET_KEY` are secret and should never be committed to git

## Phase 7: Deploy to Vercel

### 7a. Push to GitHub

If the user cloned the repo (not forked), they need to create their own repository:

1. Create a new repository on GitHub (public or private, their choice — do NOT initialize it with a README)
2. Update the remote and push. First check which branch they are on:

```bash
git branch
```

Then push (use whichever branch name is shown — likely `master`):
```bash
git remote set-url origin https://github.com/THEIR-USERNAME/THEIR-REPO-NAME.git
git push -u origin HEAD
```

If they downloaded the ZIP, they need to initialize git first:
```bash
git init
git add .
git commit -m "Initial commit from retell-voice-demo template"
git remote add origin https://github.com/THEIR-USERNAME/THEIR-REPO-NAME.git
git branch -M main
git push -u origin main
```

### 7b. Import to Vercel and add environment variables

1. Go to https://vercel.com/new
2. Click **Import** next to their GitHub repository
3. **Before clicking Deploy**, expand the **Environment Variables** section
4. Add all 11 environment variables from `.env.local`:
   - `NEXT_PUBLIC_SUPABASE_URL`
   - `NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY`
   - `SUPABASE_SERVICE_ROLE_KEY`
   - `RETELL_API_KEY`
   - `RETELL_AGENT_ID`
   - `NEXT_PUBLIC_RECAPTCHA_SITE_KEY`
   - `RECAPTCHA_SECRET_KEY`
   - `NEXT_PUBLIC_MAX_CALLS_PER_DAY`
   - `NEXT_PUBLIC_MAX_CALL_DURATION_SECONDS`
   - `NEXT_PUBLIC_APP_NAME`
   - `NEXT_PUBLIC_APP_DESCRIPTION`

> **Common mistake**: Watch for typos like `EXT_PUBLIC_` instead of `NEXT_PUBLIC_`. This will cause a `MIDDLEWARE_INVOCATION_FAILED` error.

5. Click **Deploy**

If the build fails, check that all environment variables are set correctly. Go to **Settings > Environment Variables** to verify, fix any issues, then go to **Deployments** > click **...** on the latest deployment > **Redeploy**.

Copy the production URL (e.g., `https://your-app.vercel.app`).

## Phase 8: Configure Supabase for Production

### 8a. Set the Site URL and redirect URL

1. Go to Supabase **Authentication > URL Configuration**
2. Set **Site URL** to the Vercel production URL: `https://your-app.vercel.app`
3. Under **Redirect URLs**, add: `https://your-app.vercel.app/auth/callback`
4. Click **Save**

> **Important**: If the Site URL is still `http://localhost:3000`, Google sign-in on the production site will redirect back to localhost instead of the live site.

### 8b. Add the production domain to reCAPTCHA

1. Go to https://www.google.com/recaptcha/admin
2. Click on their reCAPTCHA site
3. Click the settings gear
4. Under **Domains**, add the Vercel domain (e.g., `your-app.vercel.app`)
5. Click **Save**

## Phase 9: Verify Production

Tell the user:
> Let's verify everything works on the live site.

Walk them through:
1. Open the production URL in a **new incognito/private window** (to avoid cached auth)
2. They should see the landing page with sign-in prompt
3. Click **Sign in with Google** — should complete successfully and show the call interface
4. Click **Start Call** — agent should respond, blob should animate
5. End the call after a few seconds
6. Check Supabase **Table Editor > call_logs** — a new row should appear

If anything fails, see [troubleshooting.md](references/troubleshooting.md).

## Completion

When everything works, summarize for the user:

> Your voice agent demo site is live! Here is what is running:
>
> - **URL**: (their production URL)
> - **Agent**: (their agent name/ID)
> - **Protection**: Google Auth + reCAPTCHA v3 + daily quotas
> - **Limits**: (their configured max calls/day) calls per user per day, (their max duration) seconds each
> - **Call logs**: visible in Supabase Table Editor > call_logs
> - **Cost**: Free hosting on Vercel, free auth on Supabase, calls billed by Retell
>
> Share the URL with anyone you want to try your agent!

**Optional customizations**:
- Change the agent: update `RETELL_AGENT_ID` in Vercel env vars and redeploy
- Change quotas: update `NEXT_PUBLIC_MAX_CALLS_PER_DAY` and `NEXT_PUBLIC_MAX_CALL_DURATION_SECONDS`
- Change branding: update `NEXT_PUBLIC_APP_NAME` and `NEXT_PUBLIC_APP_DESCRIPTION`
- Replace the logo: swap `/public/logo.png` with their own image

## Optional: Local Development

If the user wants to run the app locally for development or customization later, they need to add localhost to Supabase:

1. Go to Supabase **Authentication > URL Configuration**
2. Under **Redirect URLs**, add: `http://localhost:3000/auth/callback`
3. Click **Save**
4. Run `npm run dev` and open http://localhost:3000

> **Note**: Do NOT change the Site URL to localhost — leave it pointing to the production URL. The redirect URLs list can contain multiple entries, so both production and localhost will work simultaneously.

## Error Handling

At any step, if something fails, consult [troubleshooting.md](references/troubleshooting.md) before asking the user to retry. Common issues are covered there including: Supabase auth problems, Google OAuth misconfigurations, reCAPTCHA domain issues, Vercel deployment errors, and microphone permissions.
