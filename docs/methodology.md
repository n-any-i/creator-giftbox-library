# Methodology

How a box gets from "found a video" to "a card in the library". Written to be repeatable by a person or by Claude.

## 1. Sourcing

- **Search:** TikTok keyword search through the TikHub integration (for example "<brand> PR box unboxing", 20 results per query). Instagram was used briefly early on; it is noisier.
- **Brand-posted videos** are preferred for clean design views. **Creator videos** are preferred for reaction.
- **Contributor submissions** (video links sent by people who made the box) are the target source going forward. See `contributor-guide.md`.

## 2. Selection (see `quality-bar.md`)

A box is added only if it is a designed box, mailer, bag or insert that is worth designing from, and the video shows the exterior, the opening and the contents clearly enough to document. Prefer videos of 40 seconds or less so the reveal falls early.

## 3. Capture

Frames are taken from the public video in a window (default the first 8 seconds). Five to nine frames are chosen by eye to show distinct angles: closed, side, opening, open, inside lid, each item, and any card or insert (front and back when shown). If the first window misses the reveal, one later window may be tried; if that also fails, the video is dropped.

Current pipeline: `scripts/capture/RECIPE.md` (search via Aident/TikHub, download and frame extraction local, no relay). Earlier chat-sandbox tooling is in `workflow-and-tooling.md` (superseded, kept for history). Frames are stored exactly as captured and embedded without resizing or re-compression. See "Image definition" below.

## Image definition

Creator marketing and packaging teams need to read the design, so definition is a priority.

- **Never downsize or re-compress** a screenshot after capture. Do not add width or quality options to build scripts, and do not shrink images to fit more boxes into the preview page.
- **The current blur comes from capture, not from embedding.** The hosted converter makes a 640-pixel-wide GIF from the 1080 x 1920 video, and the frames were fetched back at 320 pixels (half of that) to fit more frames in one fetch. A test on a candy-brand video showed that fetching at 640 pixels keeps fine print and texture that the 320-pixel frames lose. A 640-pixel frame is about 5 times larger in file size (about 290 KB against about 56 KB), and a single fetch is limited to about 40 frames at that width.
- **Best source:** frames extracted locally from the original 1080 x 1920 video (Claude Code or Cowork), or high-resolution stills. Use contributor-supplied video files or photos whenever they exist.
- **Going forward:** capture new boxes at 640 pixels wide or better. Re-capture existing boxes at 640 pixels when they move to hosting.
- **Cost of sharpness:** sharper frames do not fit a single-file Claude Artifact (16 MB cap). They need hosting that serves image files. See `roadmap.md`.

## 4. Description

Written by hand from what is visible or said on camera:
- **Items:** name, a plain description, and one reference frame each. An item that never appears is listed as "Not shown in the sampled seconds" with no image.
- **Why it works for the brand:** two to three sentences tying the box to the brand and product.
- **What to take from it:** two to three things a designer can copy.
- **Tags:** from a fixed list (custom shape, staged reveal, printed message, personal touch, keepsake or bonus item, co-brand, themed to the product, plain retail packaging).

## 5. Reception rating

Reception is deliberately qualitative. It is a directional read from the creator's own words and a small sample of comments, not a count of every creator who received a box.

| Level | Meaning |
|---|---|
| Raving | Commenters repeatedly call it a dream or ask where to get the exact box |
| Loved | Comments consistently praise the packaging |
| Positive | Friendly but mild, or one creator's clear enthusiasm |
| Low signal | Little enthusiasm found in the sample |
| Not enough data | A brand's own video, or no reaction sampled; not rated |
| Critical | The sampled comments lean negative rather than simply quiet — described factually (what was said, how many comments), never as a verdict on the brand overall |

Each rating carries an evidence label: **strong** (many comments agree), **some** (a handful of comments or one creator's words) or **thin** (no comment sample). Up to three short quotes are shown, attributed ("Comment", "Creator's caption", "Most-liked comment"), with no like counts.

Internally, reach and like rate have been used as context when placing a box in a level. They are never displayed. This should be stated openly if the methodology is published.

## 6. Quality checks

- `python scripts/giftbox_data.py` validates ids, levels, frame counts, image files, item-to-frame references, source links, and that no view, like or follower numbers appear in page text.
- `scripts/test/smoke.js` opens every box in both view modes in a simulated browser and checks for errors.
- A person reviews each new box in the story and flat-lay views.

## 7. What we do not claim

- That a box "performed well". We say how creators reacted in our sample.
- That the sample is complete or representative of all creators or all brands.
- That what appears in an 8-second window is everything in the box.

## 8. Known limitations

- **Windows:** only about 8 seconds of video per fetch; some contents are never shown.
- **Platform and language bias:** mostly TikTok, mostly English-language results.
- **Visibility bias:** boxes that creators film and that surface in search are over-represented.
- **Thin reception data:** most entries have creator words only, not comments.
- **Single reviewer:** descriptions and ratings are written by one author (Claude), then reviewed by the project owner.

## 9. Cost and effort

About $0.012 to $0.02 in tool cost per box with the current method, but manual capture takes several dependent steps per box; this is the main reason the library grew slowly. See `roadmap.md` for the plan to move capture to a script.
