# PR Gift Box Library: how a box gets found, captured and logged

> **Superseded.** Written October 31, 2026, describing the chat-sandbox pipeline (Aident-hosted ffmpeg, the wsrv.nl relay, the 8-second window cap). As of November 2026, capture happens in Claude Code instead: search still goes through Aident/TikHub, but download and frame extraction are local (`yt-dlp` + a static `ffmpeg` binary, no relay, no window cap, full source resolution). See `scripts/capture/RECIPE.md` for the current pipeline. This page is kept as a historical record of what was tried and why it didn't scale — most of the "tools that would speed this up" wishlist in section 6 has since been solved by moving off the chat sandbox entirely, not by new Aident actions.

Written October 31, 2026, for the team that wants to help speed this up. It describes what I actually do today, which tools I call, what failed and why I fell back, and what would make it faster. Where I am inferring rather than observing something, it says so.

## 1. The short answer: why only 1 to 3 boxes per round

One box takes about six dependent steps, and I can run them only one at a time:

| # | Step | Tool | Why it is separate |
|---|------|------|--------------------|
| 1 | Resolve a playable video file | Aident `api:redfox_hub:tool_video_download` | Needs the post URL from search |
| 2 | Turn a time window into a GIF | Aident `cli:ffmpeg:create_gif` | Needs the file URL from step 1 |
| 3 | Build a frame-fetch URL | my sandbox (bash) | Needs the signed GIF URL from step 2 |
| 4 | Fetch ~95 frames as one base64 strip | Aident `cli:curl:get` via the public service wsrv.nl | Needs the URL from step 3 |
| 5 | Decode, cut frames, build a contact sheet | my sandbox (bash) | Needs the output of step 4 |
| 6 | Look at the contact sheet and write the entry | image view, then my own writing | Needs step 5 |

Then, per round: write the data (items, concept, reception, item-to-frame mapping), rebuild the page, run a smoke test, and publish.

What slows it down, in order of impact:

1. **Strictly sequential calls.** Each step needs the previous step's output (signed URLs), and in practice my tool calls execute one at a time. When I tried to batch several calls in one message, only one ran. I do not know the exact per-response cap on tool calls; I infer one exists because responses end after roughly 60 to 100 calls.
2. **Long outputs.** Each `create_gif` call returns several thousand tokens of ffmpeg log. A comment pull can return tens of thousands of tokens of JSON. That fills my working context quickly.
3. **Only about 8 seconds of video per fetch.** The frame service (wsrv.nl) rejects images over 71 megapixels, which caps one fetch at about 95 frames, or 8 seconds at 12 frames per second. A 30-second unboxing needs 3 or 4 fetches to cover.
4. **Blind window selection.** I cannot see the video before fetching it, so I guess where the reveal happens. About one in four first windows missed the box (Alani Nu showed only a sealed carton; Rare Beauty's first window began after the box was opened).
5. **Weak candidates.** Search results often turn out to be plain shipping cartons or retail cases (a Poppi doorstep stack, a Rare Beauty first-PR carton). I only find that out after spending the steps.
6. **Hand-written content.** I describe each item, write the concept, map each item to a frame, and judge reception myself. That is intentional, because it is the part of the work that carries judgment, but it takes time.

## 2. Current workflow, step by step

**A. Find candidates (cheap).** TikHub TikTok keyword search, one query per brand or theme, 20 results each. I filter results by caption keywords (pr, gift, unbox, package, box), duration (prefer 40 seconds or less, so the reveal is early), and whether the brand's own account or a creator posted it. Brand-posted videos are preferred for design, creator videos for reaction.

**B. Pick the video.** I prefer: short, brand-specific, a designed box rather than a plain carton, and recent. This is a judgment call made from the caption and duration only.

**C. Capture frames.** RedFox resolves a downloadable file, then Aident's hosted ffmpeg makes a 640-pixel GIF for a start time and duration, then the frames are fetched and cut in my sandbox. Default window is 0 to 8 seconds; for longer videos I add a second window.

**D. Read the frames.** I look at a contact sheet (every 4th frame, numbered) and decide which 5 to 9 frames show distinct angles: closed, side, opening, open, inside lid, each item, any card or insert.

**E. Write the entry.** For each box: brand, name, category, 5 to 9 labeled frames, an item list with a reference frame per item, "why it works for the brand", "what to take from it", a buzz rating with an evidence label, up to three quotes, and source links.

**F. Reception (kept deliberately light).** Optional TikHub comment pull (about $0.001) for the top comments; otherwise the creator's own caption or on-screen text. The rating is directional and the page says so. No view counts are shown because the collection is not exhaustive.

**G. Build and publish.** A Python script embeds the frames in one HTML page (the repo build embeds them untouched; the first live build resized them to 300 pixels and re-compressed them at quality 72, which is no longer the practice), a simulated-browser test checks every box in both view modes, and the Artifact tool publishes over the same link.

## 3. Sources: what I am and am not scraping

- **TikTok:** almost everything. Search, comments, and brand or creator posts, via TikHub.
- **Instagram:** only in the first round (keyword search and a few reels, for example MAC's reel and the Bloom giveaway reels). The Instagram hashtag lookup returned nothing for a brand gifting tag. Instagram search is noisy, so I moved to TikTok.
- **Not used:** YouTube Shorts, Reddit, brand sites, press coverage, event photos.
- **Audio:** I do not transcribe audio myself. For five early videos I used TikTok's own auto-caption files (they appear in the TikHub search results). For all other videos I rely on on-screen text and frames.
- **Vision:** I look at frames myself; there is no separate video-understanding tool in the chain.

## 4. Tools called, tools that failed, and why I fell back

### Worked
- `api:tikhub_api:tiktok_app_v3`: video search ($0.001 per call), video comments ($0.001), brand feed (one call, $0.001).
- `api:tikhub_api:instagram_v3`: general search ($0.008 per call).
- `api:redfox_hub:tool_video_download`: resolves a downloadable TikTok or Instagram file ($0.01 per video). This was the only way I found to get a file URL that the hosted ffmpeg could read.
- `cli:ffmpeg:create_gif`: free; 640-pixel-wide GIF at 12 fps for a start time and duration.
- `cli:curl:get`: free but marked high-risk; each call needed an acknowledgement. I passed `once` each time because your approval was for individual uses.
- Aident discovery and billing calls: capability search, get, preflight, audit. The audit summary showed $0.147 at the one point I checked.

### Failed, and what I did instead

| Attempt | What happened | Why I fell back |
|---|---|---|
| Download TikTok cover images from my sandbox | Blocked | My sandbox's network allowlist excludes the TikTok and Instagram image hosts |
| `composio:codeinterpreter_tools:codeinterpreter_execute_code` (planned image fetcher) | "Could not be loaded" | Needed a different route; used `cli:curl:get` |
| `cli:ffmpeg:resize_720p_mp4` and `create_gif` on TikHub's direct video URLs | HTTP 403 | The signed links are tied to TikHub's servers; RedFox links worked, usually on `v16.tokcdn.com`. One RedFox link on TikTok's own CDN also worked (Alo), so a 403 is not guaranteed |
| Download the RedFox file from my sandbox | Not reachable | Same allowlist; so ffmpeg runs hosted, not in my sandbox |
| Download the hosted GIF from `api.aident.ai` | Not reachable from my sandbox | The sandbox allowlist excludes that host; I used a public image service as a relay |
| `loadout.aident.ai/_next/image` as a relay | HTTP 400 | Not a general proxy |
| wsrv.nl on a whole 17-second GIF | "Input exceeds pixel limit" | Request frames in pages of about 95 at 320 px wide, or about 40 at 640 px wide (a JPEG strip cannot be taller than about 65,000 px) |
| Instagram hashtag lookup | Returned nothing | Used keyword search instead |
| First 8-second windows | Sometimes miss the box | No way to preview; I add a later window or skip the box |
| Dashboard artifact type | Built for metric charts, not an image library | Built a custom gallery page instead |

### A privacy note
To get frames back to me, I sent time-limited signed Aident storage links to the public wsrv.nl service, which fetched the GIFs and returned encoded frames. These are public brand and creator videos, and the links expire in about a week, but you should know third-party handling happened.

## 5. Decisions I made, and why

- **Short videos first.** A 10 to 20 second unboxing usually reveals the box in the first 8 seconds, which is all one fetch can cover.
- **Brand videos for design, creator videos for reaction.** Brands show the box cleanly; creators show it being opened, with the reaction.
- **Skip plain cartons.** A plain shipper with no designed interior adds little as inspiration. I added two on purpose (AG1, The Ordinary) as contrast.
- **No view counts.** My sample is incomplete and a single post's views can mislead, so the page shows a qualitative buzz rating with an evidence label and quotes.
- **"Not enough data" instead of guessing.** Where I had no comments or creator reaction (for example, a brand's own video), the box is not rated.
- **Honest gaps.** Items that never appeared in the sampled seconds say "Not on camera in the sampled seconds".

## 6. Tools that would speed this up (my wish list)

In order of expected impact:

1. **A video-to-frames action** that takes a video URL, a time range and a count, and returns N evenly spaced still images (or one contact sheet) as retrievable files. This would replace steps 2 to 5 with one call and remove the 8-second cap.
2. **A video-understanding action** (vision plus speech) that takes a video URL and returns structured JSON: box exterior description, items seen with timestamps, on-screen text, and transcript. Even a rough version would let me pre-screen candidates before spending frames.
3. **Parallel or batch execution**, so six boxes can move through the same step together.
4. **A quiet mode** for ffmpeg and similar actions that suppresses the long stderr log, plus a way to save large JSON (such as comments) to a file without printing it.
5. **Sandbox network access** to the media hosts (TikTok and Instagram CDNs, `api.aident.ai` storage), or a file-return option so a hosted action's output lands in my sandbox. That would let me run ffmpeg and OpenCV locally and remove the public relay.
6. **A real "download this video to the user's workspace" action** that returns a file, not just a link.
7. **Transcription** for TikTok and Instagram audio.
8. **A candidate finder** that searches by what the video shows, such as "box exterior visible in the first 3 seconds".
9. **A comment-sentiment summarizer** action that returns themes and a few representative comments from a post.
10. **More sources:** YouTube Shorts search, working Instagram hashtag search, and brand-event feeds.
11. **A persistent store** (database) the page can read, so adding a box is one row instead of a rebuild of a 4 MB page.

## 7. How you can help right now

- **Approve `cli:curl:get` for `always`** if you are comfortable limiting it to image-relay use. That removes one acknowledgement per fetch.
- **Send me a prioritized brand list** (for example, Sephora Squad, Ulta, Lululemon, Alo, Hoka) and, if you have them, direct links to brand gifting reels or event videos. Direct links skip the search and vetting steps.
- **Tell me my minimum bar** for a box (a designed box only, or retail cartons too). That saves the candidates I currently drop.
- **Share any internal footage or photos** of boxes your team has received; stills from you are better than 8-second slices of a video.
- **Run the pipeline outside this chat.** The cleanest fix is a script in Claude Code or Cowork, where the machine can reach the media hosts: search with the TikHub API, download the video, extract scene-based frames with ffmpeg, build contact sheets, and call a vision model for a first draft of the items and concept. A human reviews and approves entries. I estimate this could capture 15 to 30 boxes an hour instead of 1 to 3 per round.

## 8. Cost per box (current method)

About $0.012 to $0.02: search share ($0.001), video resolve ($0.01), optional comment pull ($0.001). The hosted ffmpeg and curl steps are free. Spend to date is roughly $0.50 of the $1 cap (estimated; the Aident audit log is the source of truth).

## 9. Files behind the page

- `boxes_data.py`, `boxes_more.py`: box content (items, concept, tags, frame labels).
- `reception2.py`: buzz rating, evidence label, quotes, and the item-to-frame mapping.
- `template3.html`, `build.py`: the page and the build script.
- Frames: stored in the sandbox as numbered JPEGs per video; the page embeds a 300-pixel version of each chosen frame.


## Update: image definition

The hosted GIF is 640 pixels wide, but the first pipeline fetched frames at 320 pixels. A later test showed that fetching at 640 pixels (about 40 frames per fetch) preserves fine print and texture, at roughly 5 times the file size per frame. A local extraction from the original 1080 x 1920 video would be sharper still. `cli:ffmpeg:thumbnail_jpeg` (one still at a timestamp) was not tested and may return full-resolution stills.
