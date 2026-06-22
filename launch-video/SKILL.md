---
name: launch-video
description: This skill should be used when the user asks to "make a launch video", "create a product hype/sizzle film", "build a teaser/trailer for a drop", "do a reveal montage with an end-card CTA", "cut a hero video to the beat", or "export a launch film in 16:9/9:16/1:1". Covers the hook→tease→reveal→feature-montage→end-card arc and multi-aspect export.
version: 0.1.0
---

# Launch Video

Make a premium, high-energy 15–60s launch film that stops the scroll and converts. Build the classic arc — hook → tease → reveal → feature montage → end-card CTA — cut to the music, and export for every platform aspect.

## When to use

- Product drops, feature announcements, trailers, hype reels (15–60s).
- "Sizzle" hero videos for a landing page.
- Beat-synced reveals where the brand moment lands on the drop.

## The arc

| Beat | Job | Typical share (of 30s) |
|---|---|---|
| Hook (0–3s) | A striking frame/motion that stops the scroll | 0–3s |
| Tease | Hint the product; build curiosity, beat-synced | 3–9s |
| Reveal | Product/logo hits on the music drop | 9–13s |
| Feature montage | Fast, kinetic, one benefit per shot | 13–25s |
| End card | Logo + tagline + CTA, clean hold | 25–30s |

Scale the same proportions for 15s (tighter) or 60s (longer montage, never a longer hook).

## Two non-negotiable rules

1. **Sound design leads picture.** Lock the track first, mark the beats and the drop, then cut visuals to those marks. Never score a finished cut — the reveal lands on the drop, not near it.
2. **Quality over quantity.** A few flawless shots beat many mediocre ones. Cut any shot that isn't premium.

## Beat-synced reveal on the drop

Find the drop's exact timecode in the audio, then time the reveal so the brand mark snaps to full at that frame, with a micro-overshoot for impact.

```js
import gsap from "gsap";
const DROP = 9.0; // seconds, measured from the track
const tl = gsap.timeline({ paused: true });
// tease builds tension up to the drop
tl.to(".tease", { opacity: 1, scale: 1.05, duration: DROP, ease: "power1.in" })
// reveal SNAPS on the drop frame
  .set(".logo", { opacity: 1 }, DROP)
  .fromTo(".logo", { scale: 1.18 }, { scale: 1.0, duration: .18, ease: "power3.out" }, DROP)
  .fromTo(".flash", { opacity: .9 }, { opacity: 0, duration: .25 }, DROP); // white hit
audio.addEventListener("play", () => tl.play());
```

Cut on transients, not on a fixed grid: the hardest cuts go on kick/snare hits; ramp speed (time-remap) into the drop, then hard-cut out of it.

## Kinetic feature montage

One benefit per shot, ~0.6–1.0s each, hard cuts on the beat. Each shot = a bold keyword + a single supporting visual.

```css
.feature { opacity:0; }
.feature.in .kw   { animation: pop .35s cubic-bezier(.22,1,.36,1) forwards; }
.feature.in .word { display:inline-block; }
@keyframes pop { from { opacity:0; transform:translateY(24px) } to { opacity:1; transform:none } }
```

```js
// drive shots off beat marks measured from the track
const beats = [13.0, 13.8, 14.6, 15.4, 16.2]; // seconds
beats.forEach((t, i) => scheduleAt(t, () => showFeature(i)));
```

Keep one motion language across the montage (same enter curve, same exit) so speed reads as confidence, not chaos.

## End card

Hold the logo + tagline + CTA still and clean for ≥2s. No busy motion competing with the CTA; let the brand settle. Land the brand moment on the strongest remaining beat, then a crisp button/URL.

## Multi-aspect export

Compose with a center safe zone so one master crops cleanly to all aspects.

| Aspect | Use | Resolution | Safe zone |
|---|---|---|---|
| 16:9 | YouTube, landing hero, X | 1920×1080 | keep key content within center 90% |
| 9:16 | Reels, TikTok, Shorts, Stories | 1080×1920 | text within center 80% h; avoid top 12% / bottom 18% (UI) |
| 1:1 | Feed posts | 1080×1080 | center square of the 16:9 frame |

Design the hero/logo/CTA inside the **1:1 center square** so it survives every crop. Render the 16:9 master, then reframe (don't just letterbox) 9:16 and 1:1 from the same project.

## Output checklist

- Track locked first; reveal lands exactly on the drop.
- Hook earns the first 3 seconds.
- Montage: one benefit per shot, hard cuts on beats, consistent motion language.
- End card holds ≥2s with one clear CTA.
- 16:9 + 9:16 + 1:1 exports, key content in the center safe zone.
- Premium-only shots — nothing mediocre survives the cut.

## Reference files

- `references/launch-structure.md` — a full shot-by-shot template with timecodes for a 30s film (scalable to 15–60s), beat-synced reveal timing, kinetic-montage shot recipes, the sound-design-leads-picture workflow, and detailed multi-aspect safe-area maps for 16:9 / 9:16 / 1:1.
