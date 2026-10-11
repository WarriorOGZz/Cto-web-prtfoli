# OmniRush Cloud — mobile-first starter

A private, mobile-first browser UI that runs on a cloud Linux workspace. It provides:

- A one-time app password gate.
- A browser terminal for `gh auth login --web` and `omnirush login`.
- Connection checks for GitHub CLI and OmniRush CLI.
- One chat record per workspace/repository, with recent conversation context replayed to `omnirush -p` on each turn.
- Existing GitHub repo selection or automatic creation of a **private** repo per new project.
- Repository file browsing/preview and a confirmation-gated commit/push button.
- SQLite chat history stored on the cloud host, not in phone storage.

## Important limitations

This is a starter implementation, not an official OmniRush client. It calls the documented OmniRush CLI (`omnirush -p "..."`) and replays the last 12 chat messages as context; it does not currently use OmniRush's native session database. The browser terminal is a general-purpose shell running on the host, so keep this app private and never expose it without a strong password and HTTPS. This app does not access the phone's local files.

**Codespaces is not a guaranteed 24/7 host.** Codespaces can stop after inactivity or at its maximum lifetime. To make this truly available 24/7, run the same Node app on an always-on VM/container host with persistent storage and a process supervisor. A static Vercel deployment is not sufficient for persistent PTY sessions. The current app does not auto-restart a stopped Codespace.

## Run in GitHub Codespaces (from the Codespaces terminal)

Node 22.19+ is required by OmniRush. In the Codespace terminal, run:

```bash
node --version
npm install -g omnirush@latest
sudo apt-get update && sudo apt-get install -y gh git python3 make g++
```

Upload/extract this folder into the Codespace workspace, then:

```bash
cp .env.example .env
node -e "const fs=require('fs'),c=require('crypto');let p='.env',s=fs.readFileSync(p,'utf8').replace('replace-this-with-a-long-random-password',c.randomBytes(32).toString('base64url')).replace('replace-this-with-a-different-long-random-secret',c.randomBytes(48).toString('base64url'));fs.writeFileSync(p,s)"
cat .env
npm install
npm start
```

Open port **3000** in the Codespaces Ports tab and keep it private. The app will ask for the generated `APP_PASSWORD` printed by `cat .env`; keep it private and do not paste it into a chat or share it. If you need to regenerate it, replace `APP_PASSWORD` with a new random string of at least 20 characters and restart the server.

In the first-run terminal, connect GitHub and OmniRush:

```bash
gh auth login --web
omnirush login
```

Follow the device-code instructions. Once both connections show as connected, choose a repo or create a private one and start chatting.


## Deploy for 24/7 operation on Railway

The repository includes a Dockerfile and Railway health-check/restart configuration. Railway is a persistent service host; use a paid Hobby plan for ongoing use. Free/trial credits are temporary and not a promise of free 24/7 hosting. Current Railway pricing: https://railway.com/pricing

1. Create a Railway account and a new project. Deploy this folder from a repository or upload/push the project source as Railway requires.
2. In the service settings, use the included `Dockerfile` build.
3. Add a **Volume** mounted at `/data`. This is essential: without it, login tokens, chat history, and workspace files can be lost on redeploy/restart.
4. Set these service variables (the Dockerfile sets the storage paths and secure-cookie defaults):
   - `APP_PASSWORD` — a long random password (at least 32 random characters).
   - `SESSION_SECRET` — a separate long random secret.
   - `PORT` — `3000` if the platform does not inject one automatically.
   - `HOME` — `/data/home`
   - `DATA_DIR` — `/data/app`
   - `WORKSPACES_DIR` — `/data/workspaces`
   - `COOKIE_SECURE` — `1`
5. Keep Railway's service running as a **persistent service**. Do not enable Serverless/Sleep mode. The included `/healthz` endpoint is used for health checks.
6. Deploy, open the generated HTTPS domain, sign in with `APP_PASSWORD`, then use the in-app terminal to run `gh auth login --web` and `omnirush login`. Complete both device-code flows.
7. Test a chat and reload the page. Then confirm the same history/repository remains after a service redeploy.

**Cost and reliability:** always-on hosting is not guaranteed to remain free. Railway's current Hobby plan has a $5/month minimum usage charge with $5 monthly usage credits; compute and storage usage may increase the bill. Set a spend limit and review the live pricing before deploying. Railway documents persistent services and volumes at https://docs.railway.com/services and https://docs.railway.com/volumes/reference.

**Security:** this app embeds a general-purpose shell. Keep the app private, use a strong password, never share auth tokens, and do not publish `.env` or `/data/home`.

## Security and operations

- Keep the forwarded port private. Never use a public port with the starter password.
- The terminal grants shell access to the host; only use it for your own workspace.
- `APP_PASSWORD` is required. The `.env` setup creates a stable random `SESSION_SECRET` and SQLite-backed sessions so sign-in survives server restarts. For production, enable secure cookies behind HTTPS (`COOKIE_SECURE=1`), and add rate limiting, CSRF protection, durable session storage, workspace quotas, and a stronger identity layer.
- The GitHub CLI authentication is stored by `gh` in the cloud user's home directory; OmniRush auth is stored under `~/.omnirush`. Do not bake these credentials into images or commit them.
- Use private repos by default. Review changes before pushing.
