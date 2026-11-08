# Data schema

`data/boxes.json` is the source of truth: a JSON array, one object per box, in the order they were added. Images are files under `data/images/<id>/NN.jpg` and are referenced by path. `scripts/build_preview.py` inlines them into one HTML file.

## Fields

| Field | Type | Notes |
|---|---|---|
| `id` | string | lowercase letters, digits, hyphens; unique; used as the page link (#id) |
| `brand` | string | brand name as shown |
| `name` | string | short name of the box |
| `cat` | string | one category (list below); a new category appears as a new filter automatically |
| `date` | string | `YYYY-MM`, month the video was posted |
| `tags` | string[] | from the tag list below; may be empty |
| `occ` | string | one occasion (list below); why the box was likely sent |
| `hero` | integer | index of the frame used as the card photo |
| `src` | string | "Brand video" or "Creator video" |
| `images` | `{s, l}`[] | `s` = path under `data/`, `l` = story caption; 4 to 9 items, in story order |
| `items` | `{n, d, r}`[] | name, description, `r` = frame index or `null` if not shown |
| `concept` | string | why it works for the brand and product |
| `learn` | string[] | 2 to 3 take-aways |
| `lvl` | integer | 4 Raving, 3 Loved, 2 Positive, 1 Low signal, 0 Not enough data, -1 Critical |
| `ev` | string | `strong`, `some` or `thin` |
| `basis` | string | one or two plain sentences on how people reacted and how much we know |
| `q` | `{t, w, top}`[] | quote text, who said it, whether it was the most-liked comment (no counts) |
| `links` | `{l, u}`[] | source links; at least one |
| `pick` | boolean | optional; omit unless `true`. Marks a box as the curator's editorial pick on the homepage ("⭐ Our pick"), independent of its reception rating. Set on at most a small handful of boxes at a time — it's meant to stay rare. |

## Homepage spotlight badges

The homepage shows at most a handful of small badges across the whole grid, computed at load time in `site/template.html` (`computeSpotlights`) — not stored in the data, except `pick` above:
- **🆕 Just added** — the single box with the most recent `date`.
- **🔥 Most raving** — the most recent box with `lvl===4` and `ev==='strong'`.
- **💗 Most loved** — up to 2 boxes with `lvl===3` and `ev` of `strong` or `some`, excluding any box that already has a badge.
- **⭐ Our pick** — any box with `pick: true`.

A box only ever shows one badge (first match wins). Kept deliberately scarce — the point is that a badge means something, not that most boxes have one.

## Categories now in use

Beauty & skincare, Drinks, Supplements & functional, Snacks & candy, Electronics & accessories, Fashion & activewear, Sports & running.

## Tags

Custom shape, Staged reveal, Printed message, Personal touch, Keepsake or bonus item, Co-brand, Themed to the product, Plain retail packaging.

## Occasions

New product launch, Brand partnership, Influencer partnership, Seasonal or event, Ongoing PR (the default when nothing else is stated or clearly implied by the brand/creator — do not guess a more specific occasion).

"Influencer partnership" means the box is built around a specific named influencer the brand collaborated with (a co-created or influencer-branded product/box) — not a general ambassador or creator-gifting program, which is "Ongoing PR" even if the brand calls it a "squad" or "crew."

## Example

```json
{
  "id": "ghia-goodie-box", "brand": "Ghia", "name": "Goodie box with die-cut bottle cradle",
  "cat": "Drinks", "date": "2024-02", "tags": ["Custom shape", "Staged reveal"], "hero": 1, "src": "Creator video",
  "images": [{"s": "images/ghia-goodie-box/01.jpg", "l": "Closed: kraft shipper with orange Ghia band"}],
  "items": [{"n": "Die-cut cradle", "d": "A scalloped cardboard insert cut out so the bottle stands up in the middle.", "r": 1}],
  "concept": "...", "learn": ["..."], "lvl": 2, "ev": "thin", "basis": "...",
  "q": [{"t": "can’t wait to get creative with these", "w": "Creator’s caption", "top": false}],
  "links": [{"l": "TikTok unboxing, @jlavs_", "u": "https://www.tiktok.com/@jlavs_/video/7338990123813457195"}]
}
```

## Validation

`python scripts/giftbox_data.py` must print OK before every commit. It checks unique ids, level and evidence values, a valid occasion, 4 to 9 frames that exist on disk, item-to-frame references in range, at least one source link, a basis for rated boxes, and that no view/like/follower counts appear in text.

## Image rules

JPEG, 9:16 frames, numbered `01.jpg` in story order, stored exactly as captured. Never resize or re-compress a frame. The frames committed so far are 320 pixels wide (the capture limit of the first pipeline); new captures should be 640 pixels wide or better, and a sharper source replaces an older frame whenever one is available. Do not commit video files.
