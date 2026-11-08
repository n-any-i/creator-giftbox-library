# Publishing and updating the site

Two separate things, don't confuse them:
- **Cloudflare Pages** — hosts the live public website.
- **GitHub** — backs up and version-controls the source code and data. Not required for the site to work.

No `git` CLI is available on this machine (Xcode Command Line Tools would need more disk than is free). Instead:
- The site itself is deployed by **drag-and-drop**, no git involved at all.
- Version control uses **dulwich**, a pure-Python git implementation (`pip install dulwich`, already done in this repo) instead of the system `git` binary. Every command below is `python3 -m dulwich ...` instead of `git ...`, but otherwise works the same way.

## Publishing to Cloudflare Pages (first time)

1. Go to [dash.cloudflare.com](https://dash.cloudflare.com) and sign up for a free account (no payment info required for Pages).
2. In the left sidebar: **Workers & Pages** → **Create** → **Pages** tab → **Upload assets**.
3. Name the project (e.g. `creator-giftbox-library`) — this becomes part of your `*.pages.dev` URL.
4. Build the latest site first if you haven't already:
   ```bash
   python3 scripts/build_site.py
   ```
5. In Finder, open the `dist/site` folder, select **everything inside it** (⌘A) — `index.html`, `boxes.json`, `images/` — and drag those selected items into Cloudflare's upload area. (Drag the *contents*, not the `site` folder itself — `index.html` needs to end up at the top level.)
6. Click **Deploy site**. You'll get a live URL like `https://creator-giftbox-library.pages.dev` within a minute.

## Publishing an update (every time after)

1. Rebuild: `python3 scripts/build_site.py`
2. In the Cloudflare dashboard, open your project → **Deployments** tab → the button to upload a new deployment (labeled **Create deployment** or similar; Cloudflare's UI wording shifts occasionally — look for "upload" on the project page).
3. Drag the `dist/site` contents in again, same as before.
4. It deploys as a new version on the **same URL** — old versions stay in the Deployments history and can be rolled back to with one click if something breaks.

No account limits worth worrying about at this scale: the free tier has unlimited bandwidth and no restriction on commercial use.

## Backing up to GitHub

### First time

1. On [github.com](https://github.com), sign in and create a new repository (+ → New repository). Name it, choose **Private** (recommended until the rights-and-ethics checklist is settled — see `docs/rights-and-ethics.md`), and leave "Initialize with README" **unchecked** (this repo already has its own files; checking it would create a conflicting history).
2. Generate a Personal Access Token: avatar menu → **Settings** → **Developer settings** → **Personal access tokens** → **Tokens (classic)** → **Generate new token** → check the **repo** scope → **Generate token** → copy it immediately (GitHub only shows it once).
3. In Terminal, run (filling in your own username, repo name, and token — don't share this command with anyone once the token is in it):
   ```bash
   cd ~/Projects/creator-giftbox-library
   python3 -m dulwich push "https://YOUR_TOKEN@github.com/YOUR_USERNAME/YOUR_REPO.git" main
   ```
   The local repo and first commit are already prepared (40 boxes, 34 brands) — this command just needs to reach GitHub.

### Every time after

```bash
cd ~/Projects/creator-giftbox-library
python3 -m dulwich add .
python3 -m dulwich commit -m "describe what changed"
python3 -m dulwich push "https://YOUR_TOKEN@github.com/YOUR_USERNAME/YOUR_REPO.git" main
```

If a push is rejected as diverged (shouldn't happen unless the GitHub repo was edited directly on github.com), ask before forcing — don't overwrite history blindly.
