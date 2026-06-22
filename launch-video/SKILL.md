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

## Deliver & verify (rendered stills → MP4)

> **Packaged helper** (`scripts/`): tile your stills with `scripts/contact-sheet.sh sheet.png f-hook.png f-mid.png f-end.png`, then assert the encode with `scripts/probe-mp4.sh out.mp4 [WxH] [fps]`. See `scripts/README.md`.

If the launch film is built/rendered in Remotion, treat it as the heavy tier: register a `<Composition>` (+ zod `schema` + `defaultProps`), drive every value off `useCurrentFrame()` — no GSAP timelines / audio-event callbacks / `Date.now()` / `Math.random()` at render time (bake beat + drop timecodes into props). Deliverable = the rendered `out/*.mp4` per aspect (plus the project).

**Verify loop — render stills → inspect → encode.** Sample the arc, with the **shipped** props (real headline/tagline/CTA), not just `defaultProps`.

```bash
# Frame-exact stills across hook → reveal → end-card
npx remotion still Launch out/f-hook.png   --frame=24  --props=props/launch.json   # hook earns first 3s
npx remotion still Launch out/f-reveal.png --frame=270 --props=props/launch.json   # logo SNAPS on the drop frame
npx remotion still Launch out/f-end.png    --frame=899 --props=props/launch.json   # end card: logo+tagline+CTA, clean hold
# inspect each: fidelity (product/logo, tagline, CTA text exact, brand colors right, reveal at the drop frame)
#   AND artifacts (text overflow, off-canvas, CTA in the 9:16 bottom third, missing font, flash frame stuck on)
```

**Multi-aspect — verify the master in EACH aspect before encoding all of them.** A reframe bug (hero pushed off the safe zone) repeats across every aspect; catch it once.

```bash
for ar in 16x9 9x16 1x1; do                                 # check reveal + end card per aspect
  npx remotion still Launch "out/reveal-${ar}.png" --frame=270 \
    --props=props/launch.json --props-merge="{\"aspect\":\"${ar}\"}"
done
# stills clean in all aspects? → then encode each:
for ar in 16x9 9x16 1x1; do
  npx remotion render Launch "out/launch-${ar}.mp4" --props=props/launch.json --props-merge="{\"aspect\":\"${ar}\"}"
done
npx remotion render Launch out/demo.gif --codec=gif --props=props/launch.json   # README demo
```

**Before you finish:**
1. `npx remotion still` renders cleanly at hook / reveal / end card — no errors, no missing fonts/assets.
2. Logo/tagline/CTA text exact, brand colors right; reveal lands on the drop frame; key content inside the center safe zone, clear of the 9:16 bottom third.
3. Frame-driven only — no GSAP/audio-callback timing, `Date.now()`, or `Math.random()` (beats/drop baked into props).
4. Shipped props render correctly (not just `defaultProps`); the end card holds ≥2s with one CTA.
5. Master verified in 16:9 + 9:16 + 1:1 before encoding; MP4s play; (optional) GIF for the README.

## Reference files

- `references/launch-structure.md` — a full shot-by-shot template with timecodes for a 30s film (scalable to 15–60s), beat-synced reveal timing, kinetic-montage shot recipes, the sound-design-leads-picture workflow, and detailed multi-aspect safe-area maps for 16:9 / 9:16 / 1:1.
