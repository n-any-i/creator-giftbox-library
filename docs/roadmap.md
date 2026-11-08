# Roadmap

## Phase 0: private prototype (done)

29 boxes, 27 brands, 7 categories. Story and flat-lay views, qualitative reception, filters, search. Published as a private Claude Artifact.

## Phase 1: reach 100 boxes (in progress — 40 of 100)

Goal: 100 designed boxes, at least 7 categories, at least 15 contributing brands.

- Capture moved to Claude Code: search via TikHub (through Aident), download with `yt-dlp`, extract frames locally with a static `ffmpeg` binary — no relay, no 8-second window cap, no per-box resolution ceiling. See `scripts/capture/RECIPE.md`. This replaced the chat-sandbox pipeline in `docs/workflow-and-tooling.md` (kept for history).
- All 29 original boxes re-captured at 760px wide (up from 320px) from their original source links.
- Reception is no longer caption-only: TikHub's comment-pull endpoint is used by default per box, so "How the box is received" reflects actual viewer comments where available, not just the creator's own words.
- Still to do: collect contributor links (log in `data/submissions/`); keep rebalancing thin categories (electronics and sports/running are hardest — most brands there just ship the retail product, not a designed box); backfill reception for early entries that still read "thin".

## Phase 2: data architecture for scale (done)

- `data/boxes.json` and `data/images/` are the source of truth.
- `scripts/build_site.py` produces `dist/site/` (plain `index.html` + `boxes.json` + `images/`), no base64 embedding and no file-size cap — this is the build that gets deployed.
- `scripts/build_preview.py` still produces the old single-file embedded build for a quick private preview (capped at ~16MB / ~40 boxes).
- Automated validation still manual (`python scripts/giftbox_data.py`); no CI wired up yet.

## Phase 3: public website (hosting decided, launching)

- **Hosting:** Cloudflare Pages, direct upload of `dist/site/` (no git required to deploy or update). Free tier: unlimited bandwidth, no commercial-use restriction. See `docs/publishing.md` for the step-by-step.
- **Source control:** GitHub, for backup and history (separate from the Cloudflare deploy step).
- **Rights:** the checklist in `rights-and-ethics.md` has open items not yet resolved (faces in frames, license, removal contact) — decide before or at launch, not after.
- **Still open:** shareable links per box (already `#id`), accessibility review, analytics, a submission form, a custom domain.

## Phase 4: community

- A submission form and review queue.
- Brand and contributor claims and corrections.
- A visible changelog and methodology page.
- Possible additions: video clips instead of stills, a "box teardown" format, a seasonal collection (for example Black Friday and holiday boxes), taxonomy refinements.

## Open decisions

| Decision | Owner | Needed by |
|---|---|---|
| Faces: crop, blur or brand-posted only | Project owner | Before public launch |
| Licence for code and for data | Project owner | Before repo goes public on GitHub |
| What contributors get (early access, credit) | Project owner | Before the LinkedIn post goes live |
| Whether to keep AG1, The Ordinary and lululemon entries | Project owner | Any time |
| Whether the reception scale needs more than one negative tier ("Critical") | Project owner | Any time — came up with the SKIMS advent calendar, likely to recur |
| Whether to swap the Lululemon entry for a better video (current one only shows the exterior carton) | Project owner | Whenever a better source is found |
