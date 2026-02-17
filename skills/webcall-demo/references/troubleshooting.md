# Retell Voice Demo — Troubleshooting

Quick-reference guide for common issues during setup and deployment.

---

## Prerequisites

### Node.js not installed or too old
**Symptom**: `node -v` returns an error or a version below v20.

**Fix**: Download the LTS version from https://nodejs.org/. After installing, close and reopen the terminal so the new version is picked up.

### Git not installed
**Symptom**: `git --version` returns an error.

**Fix**: Download from https://git-scm.com/downloads. On Windows, choose the default options during installation. Alternatively, download the ZIP from GitHub instead of cloning.

---

## npm install Errors

### Permission errors (macOS/Linux)
**Symptom**: `EACCES` errors during `npm install`.

**Fix**: Do NOT use `sudo npm install`. Instead, fix npm permissions:
```bash
mkdir ~/.npm-global
npm config set prefix '~/.npm-global'
echo 'export PATH=~/.npm-global/bin:$PATH' >> ~/.bashrc
source ~/.bashrc
```
Then retry `npm install`.

### Sharp / native module errors (Windows)
**Symptom**: Errors related to `sharp`, `node-gyp`, or native modules.

**Fix**: Install the Windows build tools:
```bash
npm install -g windows-build-tools
```
Or, if that fails, install Visual Studio Build Tools manually from https://visualstudio.microsoft.com/visual-cpp-build-tools/.

Then delete `node_modules` and `package-lock.json` and retry:
```bash
rm -rf node_modules package-lock.json
npm install
```

### General npm install failure
**Fix**: Clear the npm cache and retry:
```bash
npm cache clean --force
rm -rf node_modules package-lock.json
npm install
```

---

## Supabase Issues

### SQL migration fails
**Symptom**: Error when running the `CREATE TABLE` SQL.

**Common causes**:
- Table already exists: run `DROP TABLE IF EXISTS public.call_logs;` first, then retry
- Partial paste: make sure the entire SQL block was pasted, including the `CREATE POLICY` statement

### "relation call_logs does not exist"
**Symptom**: The app shows errors about `call_logs` not being found.

**Fix**: The SQL migration was not run. Go to Supabase SQL Editor and run the migration SQL from Phase 2c.

### Service role key not working
**Symptom**: Call logging fails silently or returns 500 errors.

**Common causes**:
- Wrong key: make sure you copied the `service_role` key, not the `anon` key (they look similar)
- The service_role key is under "Project API keys" and requires clicking "Reveal" to see it

---

## Google OAuth Issues

### Google sign-in redirects to localhost in production
**Symptom**: Clicking "Sign in with Google" on the production site redirects to `http://localhost:3000`.

**Fix**: Update the **Site URL** in Supabase:
1. Go to **Authentication > URL Configuration**
2. Change **Site URL** from `http://localhost:3000` to your production URL (e.g., `https://your-app.vercel.app`)
3. Click **Save**

This is the most common production issue.

### "redirect_uri_mismatch" error from Google
**Symptom**: Google shows an error about redirect URI not matching.

**Fix**: The Supabase callback URL must be listed in your Google Cloud OAuth credentials:
1. Go to Google Cloud Console > APIs & Services > Credentials
2. Edit your OAuth 2.0 Client ID
3. Under "Authorized redirect URIs", make sure the Supabase callback URL is listed exactly as shown in Supabase (Authentication > Providers > Google)
4. The URL looks like: `https://YOUR-PROJECT.supabase.co/auth/v1/callback`
5. Click **Save** — it can take a few minutes to propagate

### "auth_failed" after sign-in
**Symptom**: User signs in with Google but is redirected to `/?error=auth_failed`.

**Fix**: Add the redirect URL to Supabase:
1. Go to **Authentication > URL Configuration > Redirect URLs**
2. Add `https://your-app.vercel.app/auth/callback` (use your actual domain)
3. For local development, also add `http://localhost:3000/auth/callback`
4. Click **Save**

### OAuth consent screen says "unverified app"
**Symptom**: Google shows a warning that the app is not verified.

**This is normal** for development and testing. Users can click "Advanced" > "Go to (app name) (unsafe)" to proceed. For a production app used by many people, you can submit the app for verification in Google Cloud Console, but it is not required for a demo.

---

## reCAPTCHA Issues

### reCAPTCHA not loading
**Symptom**: The call fails with "reCAPTCHA token required" or the reCAPTCHA badge does not appear.

**Fix**: Add your domain to the reCAPTCHA site settings:
1. Go to https://www.google.com/recaptcha/admin
2. Click on your site
3. Click the settings gear
4. Under **Domains**, add your domain (e.g., `your-app.vercel.app` and/or `localhost`)
5. Click **Save**

### "Submit" button grayed out when registering reCAPTCHA
**Symptom**: Cannot click Submit when creating a new reCAPTCHA site.

**Fix**: Scroll down and check the **Terms of Service** checkbox first. The button activates after accepting the terms.

### reCAPTCHA score too low
**Symptom**: Calls fail with "reCAPTCHA verification failed" even for real users.

**Fix**: The template requires a score of 0.5 or higher. If legitimate users are being blocked:
- Make sure they are not using a VPN or privacy extension that interferes with reCAPTCHA
- Open the reCAPTCHA admin console to check the score distribution
- As a last resort, lower the threshold in `app/api/start-call/route.ts` (search for `0.5`)

---

## Vercel Deployment Issues

### Build fails on first deploy
**Symptom**: The Vercel build fails immediately after importing the project.

**This is expected** if environment variables have not been set yet. Add all 11 environment variables in Vercel Settings > Environment Variables, then redeploy.

### MIDDLEWARE_INVOCATION_FAILED
**Symptom**: The deployed site shows a "MIDDLEWARE_INVOCATION_FAILED" error.

**Common causes**:
- **Typo in env var names**: Double-check that variables start with `NEXT_PUBLIC_` (not `EXT_PUBLIC_` or other typos)
- **Missing Supabase env vars**: Both `NEXT_PUBLIC_SUPABASE_URL` and `NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY` are required for the middleware to initialize
- After fixing, redeploy from the Vercel dashboard

### Environment variables not taking effect
**Symptom**: Changes to environment variables do not seem to apply.

**Fix**: Vercel requires a redeploy after changing environment variables. Go to Deployments > latest > Redeploy.

---

## Call Issues

### "Failed to start call"
**Symptom**: Clicking "Start Call" shows an error.

**Common causes**:
- `RETELL_API_KEY` is incorrect or expired — verify in Retell Dashboard > Settings > API Keys
- `RETELL_AGENT_ID` is incorrect — verify the agent ID on the agent page
- The Retell agent is not active or has been deleted

### Microphone not working
**Symptom**: Call connects but no audio is sent.

**Fix**:
- The browser must grant microphone permission — look for a popup or icon in the address bar
- HTTPS is required for microphone access (Vercel provides this automatically)
- For local development, `localhost` is an exception and works without HTTPS
- Try a different browser if the issue persists

### Call auto-disconnects immediately
**Symptom**: Call starts but ends within a few seconds.

**Common causes**:
- `NEXT_PUBLIC_MAX_CALL_DURATION_SECONDS` is set too low
- The Retell agent itself may have a short timeout configured — check agent settings in the Retell dashboard

### "Daily call limit reached"
**Symptom**: User cannot start a call even though they have not called today.

**Fix**: The quota resets at midnight UTC (not the user's local timezone). Check the `call_logs` table in Supabase to verify the actual count. The `started_at` column shows when each call was logged.

---

## Windows-Specific Issues

### `cp` command not found
**Symptom**: `cp .env.example .env.local` fails on Windows.

**Fix**: Use the Windows equivalent:
```powershell
copy .env.example .env.local
```
Or if using Git Bash, `cp` should work.

### Long file path errors
**Symptom**: `npm install` fails with errors about file paths being too long.

**Fix**: Enable long paths in Windows:
```powershell
# Run PowerShell as Administrator
New-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\FileSystem" -Name "LongPathsEnabled" -Value 1 -PropertyType DWORD -Force
```
Then restart the terminal and retry.
