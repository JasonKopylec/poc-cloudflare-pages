# Cloudflare Pages — CLI (Wrangler) setup and deploy

This file shows the exact commands and steps to deploy your repo to **Cloudflare Pages** from the command line (local or CI). It covers CLI auth, required API token scopes, creating the Pages project (GUI), safely configuring `wrangler.toml` for local use, and a GitHub Actions example.

---

## Overview
- Goal: deploy `poc-cloudflare-pages` to Cloudflare Pages using the command line (Wrangler). 
- Two main flows:
  1. Local interactive auth with `wrangler login` (good for dev)
  2. Non-interactive CI deploy using `CLOUDFLARE_API_TOKEN` (recommended for automation)

---

## Prerequisites
- Node.js (v16+ recommended) and npm installed
- Git repo with `index.html` at the repository root
- Cloudflare account with access to the target Cloudflare account
- Your Cloudflare Account ID (find it on the dashboard URL or under **Accounts**)

### Install Wrangler (Cloudflare CLI)
Install the Wrangler CLI locally (you can also use `npx` without a global install).

- Install via npm (recommended):

```bash
npm install -g wrangler
```

- Install via Homebrew (macOS):

```bash
brew install cloudflare/cloudflare/wrangler
```

- Use without installing (works per-command):

```bash
npx wrangler <command>
# e.g. npx wrangler whoami
```

Verify the installation:

```bash
wrangler --version
npx wrangler whoami   # verifies auth/account (works if logged in or CLOUDFLARE_API_TOKEN set)
```

Find Account ID in the dashboard URL: `https://dash.cloudflare.com/<ACCOUNT_ID>/...`

---

## Required API token scopes (for CI / non-interactive deploy)
Minimum required permissions for `CLOUDFLARE_API_TOKEN` used by `wrangler pages deploy`:
- **Account → Cloudflare Pages → Edit**  (REQUIRED)

Optional (only if your site uses them at build/runtime):
- Account → Workers Scripts → Edit
- Account → Workers R2 Storage → Edit
- Account → D1 → Edit

Tip: use the built‑in **Cloudflare Pages** token template when creating a token in the dashboard.

---

## Option A — Local interactive deploy (quick test)
1. Authenticate interactively:

```bash
npx wrangler login
npx wrangler whoami   # verifies which account you're authenticated to
```

2. Deploy from your repo root:

```bash
npx wrangler pages deploy . --project-name poc-cloudflare-pages --branch main --account-id <ACCOUNT_ID>
```

Notes:
- `--project-name` must match an existing Pages project in the same account, or create the project first in the Cloudflare dashboard.
- If you omit `--account-id` and are logged in, Wrangler will use the authenticated account.

---

## Option B — CI / non‑interactive deploy (recommended for automation)
1. Create API token in Cloudflare Dashboard → **My Profile → API Tokens → Create Token**
   - Use **Cloudflare Pages** template
   - Copy the token immediately

2. Locally (or in CI) set the token (example — local temporary):

```bash
export CLOUDFLARE_API_TOKEN="<PASTE_TOKEN_HERE>"
npx wrangler whoami   # should return account info if token is valid
```

3. Deploy using the token (example):

```bash
export ACCOUNT_ID=<YOUR_ACCOUNT_ID>
export PROJECT_NAME=poc-cloudflare-pages
npx wrangler pages deploy . --account-id $ACCOUNT_ID --project-name $PROJECT_NAME --branch main
```

4. In CI (GitHub Actions) set `CLOUDFLARE_API_TOKEN` as a repository/organization secret (example workflow below).

---

## If the Pages project does not exist (create it)
- Easiest: create the Pages project in the Cloudflare Dashboard (**Workers & Pages → Create application → Pages → Connect repository**).
- After project creation, use `wrangler pages deploy` (CLI) to upload or update builds.

**Important:** do NOT run `npx wrangler pages deploy .` inside a Cloudflare Pages *Git-connected* build step — that causes conflicts and "Project not found" errors.

---

## Keep secrets out of git
Recommended patterns:
- Do NOT put `account_id` or tokens into `wrangler.toml` in the repo.
- Use a local file `wrangler.local.toml` (add to `.gitignore`) for any local-only config.

Example `wrangler.local.toml` (DO NOT commit):
```toml
name = "poc-cloudflare-pages"
account_id = "<YOUR_ACCOUNT_ID>"
pages_build_output_dir = "."
```

Deploy with local config:
```bash
npx wrangler --config wrangler.local.toml pages deploy . --project-name poc-cloudflare-pages --branch main
```

---

## GitHub Actions example (deploy on push to main)
Create a workflow at `.github/workflows/pages-deploy.yml` with the secret `CLOUDFLARE_API_TOKEN` set in the repo settings.

```yaml
name: Deploy to Cloudflare Pages
on:
  push:
    branches: [ main ]
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Publish to Cloudflare Pages
        env:
          CLOUDFLARE_API_TOKEN: ${{ secrets.CLOUDFLARE_API_TOKEN }}
          ACCOUNT_ID: ${{ secrets.CLOUDFLARE_ACCOUNT_ID }} # optional
        run: |
          npx wrangler whoami
          npx wrangler pages deploy . --project-name poc-cloudflare-pages --branch main --account-id $ACCOUNT_ID
```

Notes:
- Store `CLOUDFLARE_API_TOKEN` (and optionally `CLOUDFLARE_ACCOUNT_ID`) as repository secrets.
- The workflow above uses `npx` to avoid committing Wrangler to your repo.

---

## Common errors & fixes
- **Authentication error [code: 10000]**
  - Cause: token missing **Account → Cloudflare Pages → Edit** scope or token incorrect.
  - Fix: create a new Pages token (Pages template) and set `CLOUDFLARE_API_TOKEN` in CI or run `wrangler login` locally.

- **Project not found [code: 8000007]**
  - Cause: `project-name` in the deploy command doesn't exist in the account used by Wrangler.
  - Fix: create the Pages project in the Cloudflare dashboard under the same account, or verify `--account-id` is correct and the token belongs to that account.

- **Running wrangler inside a Pages build**
  - Cause: your Pages Git build contains `npx wrangler pages deploy .` as the **Build command**. That will try to deploy a project from inside a Pages build and usually fails.
  - Fix: remove that line from the Pages project's Build command in the dashboard (set it blank or `echo "no build"`).

---

## Debug checklist (run these locally)
```bash
# 1) Check auth (interactive or token)
npx wrangler whoami

# 2) Verify you can list Pages projects for the account (token must have Pages scope)
curl -sS -H "Authorization: Bearer $CLOUDFLARE_API_TOKEN" \
  "https://api.cloudflare.com/client/v4/accounts/$ACCOUNT_ID/pages/projects" | jq -r '.success,.result[]?.name'

# 3) Deploy (example)
npx wrangler pages deploy . --account-id $ACCOUNT_ID --project-name poc-cloudflare-pages --branch main
```

---

## Example full local flow (exact commands)
```bash
# 1) Create token in dashboard (Cloudflare Pages template) and copy it
export CLOUDFLARE_API_TOKEN="<TOKEN>"
export ACCOUNT_ID=<YOUR_ACCOUNT_ID>
export PROJECT=poc-cloudflare-pages

# 2) Verify token works
npx wrangler whoami

# 3) Create project in dashboard (or confirm it exists)
# 4) Deploy
npx wrangler pages deploy . --account-id $ACCOUNT_ID --project-name $PROJECT --branch main
```

---

## Troubleshooting tips
- If deploy fails with "Project not found", confirm the project exists in the same account shown by `npx wrangler whoami`.
- If deploy fails with authentication errors, revoke old tokens, create a new Pages template token, and re-run `npx wrangler whoami`.
- Avoid putting secrets or `account_id` into the checked-in `wrangler.toml` — use `wrangler.local.toml` or CLI flags.

---

## Support & references
- Cloudflare Pages docs: https://developers.cloudflare.com/pages/
- Wrangler CLI docs: https://developers.cloudflare.com/pages/platform/cli-wrangler/

---

If you want, I can:
- create a `.github/workflows/pages-deploy.yml` for you, or
- add `wrangler.local.toml` and update `.gitignore` (local-only), or
- run the verification commands here (if you provide the token/account ID).

Pick one and I'll proceed.