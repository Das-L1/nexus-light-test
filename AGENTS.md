# Setup guide for AI assistants

You are helping someone set up **Nexus Light**: a self-hosted office and MCP server that runs on their
own free Cloudflare account. Guide them step by step, in their language (often German), one step at a
time. Check each step before moving on. They may be a student without technical background.

## Ground rules

- **Never ask the user to paste a password, token or secret into the chat.** Secrets are entered in
  the Cloudflare dashboard, the Deploy button form, or `wrangler secret put` (which prompts locally).
- No credit card is needed. If Cloudflare asks for one, they clicked into a paid product: go back.
- Explain what each step is for in one sentence. Don't dump the whole guide at once.

## Step 1: Accounts

1. A free **Cloudflare** account (dash.cloudflare.com/sign-up). They must **confirm the email
   address**; until then deploying fails with error `10034 … verify your email address`.
2. For the Deploy button: a free **GitHub** (or GitLab) account.

## Step 2: Choose the setup path

- **Deploy button** (recommended, no computer skills needed): open
  `https://deploy.workers.cloudflare.com/?url=https://github.com/Das-L1/nexus-light`, connect GitHub,
  keep the suggested names, and set `OWNER_PASSWORD` when asked. Tell them to pick a long password
  they can remember or store in a password manager: it protects all their documents.
- **Command line** (has Node.js 20+):
  ```sh
  git clone https://github.com/Das-L1/nexus-light && cd nexus-light
  npm install
  npx wrangler login
  npx wrangler secret put OWNER_PASSWORD
  npm run deploy
  ```
  On Android/Termux: `npm install --os=linux --ignore-scripts && ./fix-termux.sh`, and run wrangler as
  `node node_modules/wrangler/bin/wrangler.js` (Termux has no `/usr/bin/env`).

**Before the first deploy**, ask whether data should stay in the EU. Default is yes
(`DATA_REGION="eu"` in `wrangler.jsonc`). This cannot be changed later without losing data.

## Step 3: Check it runs

Open `https://nexus-light.<their-subdomain>.workers.dev`. They should see the Nexus Light start page
with the MCP address. Signing in with the owner password shows the (empty) document list.

If the address does not load right after the first deploy, wait 1 to 2 minutes: a new
`workers.dev` subdomain needs a moment in DNS.

## Step 4: Connect you (the AI)

| Client | Steps |
|---|---|
| ChatGPT | chatgpt.com → Plugins → *Hinzufügen/Add* → Server URL `https://…workers.dev/mcp`, authentication **OAuth**, accept the risk notice → connect |
| Claude | Settings → Connectors → *Add custom connector* → URL `https://…workers.dev/mcp` |

A Nexus Light page opens: they enter the owner password and click *Erlauben/Allow*. Then call
`nexus_status` to confirm: it shows the version, the data region and your name as it will appear in
documents.

## Step 5: First document

Call `doc_create` with a short welcome text and give them the link. Ask them to open it, then write a
sentence with `doc_write` so they see your cursor typing live. Mention the *Teilen* button for
co-editing with others and the *Als App installieren* prompt.

## Optional: ByCS (Bavarian school cloud)

Only for users of BayernCloud Schule. In the Cloudflare dashboard (Workers → nexus-light → Settings →
Variables and Secrets) they add three **secrets**: `BYCS_USER`, `BYCS_PASS` and `BYCS_SCHOOL` (their
school number, e.g. `0284`, visible in their Drive address `https://0284.drive.bycs.de`). Then
`bycs_drives` should list their spaces, and `backup_now` saves a backup into the Drive folder
`cowork Dateien backup` (also automatic every Monday). Point out that the password is stored
encrypted in their Cloudflare account and that login breaks if ByCS turns on two-factor login.

## Troubleshooting

| Symptom | Cause and fix |
|---|---|
| Deploy error `10034` | Cloudflare email not confirmed: click the link in the confirmation mail |
| Deploy asks to register a `workers.dev` subdomain | Pick any name; it becomes part of the address |
| *Falsches Passwort* on the consent page | Wrong owner password. Reset it with `wrangler secret put OWNER_PASSWORD` or in the dashboard |
| ChatGPT/Claude: connection fails | Address must end in `/mcp`; authentication must be OAuth |
| Editor shows *verbinde …* forever | Network or ad blocker blocking WebSockets; try another network or browser |
| ByCS tools missing | One of `BYCS_USER`, `BYCS_PASS`, `BYCS_SCHOOL` not set as a secret |
| ByCS: *Login-Formular nicht gefunden* | ByCS changed its login (e.g. two-factor); the integration cannot work then |

## Good to tell them

- Their data lives only in their own Cloudflare account (EU by default). The developer cannot see it.
- When you read or write documents, the content goes to your provider (OpenAI, Anthropic, …).
- Backups: *Backup aller Dokumente (.zip)* in the document list; you will remind them every 14 days.
- Legal notes (German): https://neuro-net.net/nexus-feedback/rechtliches
