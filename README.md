# Creator Gift Box Library

A visual swipe file of the PR gift boxes brands send to creators, built for creator marketing managers, TikTok Shop and social commerce managers, and the packaging teams who design the boxes. Each entry shows a box from several angles, what is inside, why it works for the brand, what to take from it, and a qualitative read on how creators reacted.

**Status:** 40 boxes, 34 brands, 7 categories. Preparing for public launch on Cloudflare Pages (see `docs/roadmap.md`).

**Image note:** frames in `data/images/` are captured locally (yt-dlp + ffmpeg, see `scripts/capture/RECIPE.md`) at 760px wide or the source's native resolution if narrower, never resized or re-compressed after capture. The public build (`scripts/build_site.py`) serves them as plain files with no size cap; the legacy single-file build (`scripts/build_preview.py`) still embeds everything as base64 and is capped at ~16MB (roughly 40 boxes at this quality).

## Folder structure

```
creator-giftbox-library/
├─ README.md                  this file
├─ docs/
│  ├─ product-vision.md       why this exists, who it is for, what success looks like
│  ├─ methodology.md          how boxes are found, selected, captured, described and rated
│  ├─ quality-bar.md          include / skip rules (the most important page for contributors)
│  ├─ data-schema.md          the box data format and validation rules
│  ├─ contributor-guide.md    what people send us and what they get
│  ├─ rights-and-ethics.md    permissions, faces, takedowns, open legal questions
│  ├─ publishing.md           how to deploy to Cloudflare Pages and back up to GitHub
│  ├─ roadmap.md              phases from prototype to public site, open decisions
│  ├─ workflow-and-tooling.md historical: the Aident/chat tool chain, superseded (see capture/RECIPE.md)
│  └─ continue-in-new-chat.md how to resume capture work in a fresh Claude Code session
├─ data/
│  ├─ boxes.json              source of truth: one object per box (no image bytes)
│  ├─ images/<box-id>/NN.jpg  the screenshots, numbered in story order
│  └─ submissions/            incoming links from contributors (csv tracker)
├─ scripts/
│  ├─ giftbox_data.py         load, add, remove, validate boxes
│  ├─ build_preview.py        builds one single-file HTML preview into dist/
│  ├─ giftbox_tools_artifact.py   legacy helper that edits the published Claude Artifact directly
│  ├─ bin/ffmpeg               static ffmpeg binary (no Homebrew/Xcode needed)
│  ├─ capture/RECIPE.md        current capture pipeline: yt-dlp + local ffmpeg, no relay
│  ├─ capture/{dec,ws}.py      historical: old Aident/wsrv.nl relay helpers, superseded
│  └─ test/smoke.js           simulated-browser test (needs: npm i jsdom)
├─ site/
│  └─ template.html           the page template (filters, story and flat-lay views)
└─ dist/                      build output, git-ignored
```

## Quick start

```bash
python scripts/giftbox_data.py          # validate the data; must print OK
python scripts/build_site.py            # writes dist/site/ (index.html + boxes.json + images/): the public, hostable build
python scripts/build_preview.py         # writes dist/pr-giftbox-library.html: legacy single-file build, images embedded, capped at ~43 boxes
```

To publish: build the site, then drag the `dist/site/` folder into Cloudflare Pages (or any static host) as a direct upload. No git required for this step.

To add a box: put its frames somewhere, call `make_box(...)` and `add_box(...)` from `scripts/giftbox_data.py`, run `validate`, rebuild, commit. Details: `docs/data-schema.md` and `docs/continue-in-new-chat.md`.

## Principles

1. Inspiration, not ranking. We show good designs; we do not claim which box "performs best".
2. Visual first. Every claim about a box is grounded in frames you can see.
3. Honest about evidence. Reception is qualitative, with an evidence label, and "not enough data" is allowed.
4. Designed boxes only. Plain cartons do not get a card (see `docs/quality-bar.md`).
5. Image definition comes first. Screenshots are never resized or re-compressed after capture, and the way to fit more boxes is hosting, not smaller images (see `docs/methodology.md`, "Image definition").
6. Credit and consent. Every box links to its source; anything is removed on request.

## Not yet decided

License, the faces-in-frames policy, and the final contributor intake flow. Hosting is decided (Cloudflare Pages). See `docs/roadmap.md` and `docs/rights-and-ethics.md`. No license file is included on purpose.
