# Platform Specs & Message-Match Worksheet

Exact pixel safe zones, durations, aspect ratios, and format limits per placement (2025/2026), plus a worksheet for building message-matched hook→CTA pairs before they go into the CSV. Verify against each platform's current Ads Manager docs before a big spend — placements and safe zones shift.

## Aspect ratio → placement map

| Aspect | Resolution | Native placements |
|---|---|---|
| 9:16 | 1080×1920 | TikTok In-Feed, Instagram/Facebook Reels, Stories, YouTube Shorts |
| 4:5 | 1080×1350 | Meta Feed (largest feed real estate, recommended for Feed video) |
| 1:1 | 1080×1080 | Meta Feed, broad/automatic placements, X |
| 16:9 | 1920×1080 | YouTube in-stream, landscape feed, in-article |

Master rule: design the hook, product, and CTA inside the **1:1 center square** so a single composition reframes to every aspect by re-centering, not letterboxing. Black bars on a vertical feed read as "ad" and underperform.

## Safe zones (keep-clear bands)

Platform UI — captions, CTA chips, usernames, profile icons, progress bars — overlaps the frame. Keep all critical content out of these bands.

| Aspect / placement | Top keep-clear | Bottom keep-clear | Sides |
|---|---|---|---|
| 9:16 Reels/Stories (Meta) | ~14% (~270px on 1920h) | ~20–35% (~384–672px) | ~6% (~65px) |
| 9:16 TikTok In-Feed | ~130px | ~484px (right-side icon rail + caption) | right ~140px |
| 4:5 Meta Feed | ~5% | ~10% (keep CTA above it) | ~5% |
| 1:1 / 16:9 | center 90% safe | center 90% safe | center 90% safe |

The 9:16 bottom third is the single most violated zone — captions, the CTA button, and the handle stack there. The `bottomSafe` padding in `batch-ad-pipeline.md` reserves it.

## Duration & format limits

| Placement | Sweet spot | Hard limits | Format |
|---|---|---|---|
| Meta Feed (4:5/1:1) | 6–15s | up to 240 min, ≤4GB | MP4/MOV, H.264 |
| Meta Reels/Stories (9:16) | 8–15s | Reels ≤90s, Stories ≤2 min | MP4/MOV |
| TikTok In-Feed (9:16) | 15–30s (TikTok cites 21–34s) | up to 10 min (non-Spark) | MP4/MOV, ≥540×960 |
| YouTube in-stream (16:9) | 15–30s | skippable after 5s | MP4 |

Practical default for performance video: **15–30s, with the hook trigger landing before 2s.** Most ad-account data shows attention falls off a cliff after ~15s, so front-load everything that matters.

## Hook → CTA message-match worksheet

Fill one row per ad *before* writing the CSV. The CTA must pay off the exact promise the hook made; if it doesn't, the click is wasted and the test is noisy. Each completed row becomes one record in `variants.csv`.

| Hook (0–3s promise) | Hook type | Single benefit | Proof point | CTA (pays off the hook) |
|---|---|---|---|---|
| "Spending 2 hrs/day on reports?" | problem-first | one-click report build | "save 11 hrs/week" | "Save 2 hours — try free" |
| "Your reporting tool is lying to you." | pattern interrupt | shows real numbers instantly | "save 11 hrs/week" | "See the real numbers — free" |
| "Nobody talks about this reporting trick." | curiosity gap | one click builds it | "save 11 hrs/week" | "Try the trick — free" |
| "Cut report time in half this week." | immediate benefit | auto-built reports | "save 11 hrs/week" | "Cut it in half — free" |
| "If you build reports by hand, stop." | direct callout | automate it now | "save 11 hrs/week" | "Automate yours — free" |

Bad pairing to avoid: a "stop wasting 2 hours" hook ending on a generic "Shop now" — the promise and the payoff don't connect, so even a great hook converts poorly.

## Testing order (which field to vary first)

Run tests in funnel order; isolate one field per wave using the matrix generator:

1. **Hook** — biggest lever on CPA; test 5–8 hooks against one identical body.
2. **Offer / benefit** — take the winning hook, vary the core promise.
3. **CTA** — winning hook + offer, vary the closing action.
4. **Format / aspect** — same winning ad, compare placements.

Budget guide many teams use: ~60% to proven winners, ~30% to variations of winners, ~10% to fresh concepts. Run each test long enough for a real sample (commonly several hundred impressions per variant, ~1 week to smooth daily swings) before declaring a winner.

## Pre-flight checklist per variant

- Hook trigger lands before 2s and fits the frame (≤~60 chars in 9:16).
- CTA visibly pays off the hook's specific promise.
- Critical content inside the center square and outside every keep-clear band.
- Duration within the placement's sweet spot.
- Exported in every aspect the campaign's placements require.
- Exactly one field differs from the other variants in this test wave.
