# Rights and ethics

This page lists what has to be settled before the library goes public. It is a working checklist, not legal advice; have counsel review before launch.

**Status: hosting is ready (Cloudflare Pages), but the "Practices to add before launch" list below is not done yet.** Review it before the site goes live, not after.

## What the library contains

- Screenshots taken from public brand and creator videos, shown at small size for commentary and reference, with a link to each source.
- Short quotes (under 15 words) from public comments and captions, attributed generically.
- Original descriptions and analysis written for the library.

## Open questions

1. **Third-party content.** Screenshots belong to the video's owner. For a public site, the safest route is permission from the brand or creator (contributors give this by submitting). For boxes found by search, decide whether to seek permission, replace screenshots with the brand's own press images, or keep them only in a private or invited-access version.
2. **Platform terms.** Check TikTok's and Instagram's terms for collecting and republishing content before a public launch.
3. **People in frames.** Many frames show creators' faces. Options: crop to the box and hands, blur faces, or use only brand-posted videos for public screenshots.
4. **Removal.** Offer a visible, simple removal request path (an email address) and act within a fixed number of days.
5. **Credit.** Every card links to its source post. Contributors choose whether to be named.
6. **Third-party handling during capture — resolved.** The original pipeline sent signed storage links to the public wsrv.nl relay to get frames back. The current pipeline (`scripts/capture/RECIPE.md`) downloads videos directly with `yt-dlp` and extracts frames locally with `ffmpeg`; no third-party relay is involved.
7. **License.** Undecided. No license file is included. Separate the code (could be open) from the images and data (rights vary).

## Practices already in place

- No view, like or follower counts are shown.
- Reception is described as directional and sample-based.
- Items that never appear on camera are marked as such, not guessed.
- Quotes are short and attributed generically.

## Practices to add before launch

- A removal contact and a stated response time.
- A contributor consent record (store it in `data/submissions/submissions.csv`).
- A short methodology and limitations note on the public site (see `methodology.md`).
- A decision on faces: crop, blur, or brand-posted only.
