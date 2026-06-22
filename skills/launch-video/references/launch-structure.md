# Launch Video Structure — Cookbook

Shot-by-shot template, beat-sync timing, montage recipes, and multi-aspect maps for a premium 15–60s launch film.

## Sound-design-leads-picture workflow

1. **Pick/commission the track first.** The track defines the film's length and energy.
2. **Mark the structure in the audio**: intro, build, the **drop** (the single biggest moment), and the kick/snare transients through the montage. Note exact timecodes.
3. **Map beats → cuts.** Hard cuts land on transients; the reveal lands on the drop frame.
4. **Build picture to the marks.** Never finish a cut and then look for music. Score-first guarantees the reveal hits.
5. **Mix last**: add risers into the drop, an impact/whoosh on the reveal, and let the end-card breathe (drop music low, or a clean tag).

## Shot-by-shot template — 30s film (scalable)

```
00:00–00:03  HOOK
  Visual : single striking frame — extreme close detail / impossible motion
  Motion : slow push-in or a fast whip that resolves; NO logo yet
  Audio  : sub-bass swell begins, sparse
  Cut    : on first audible beat at ~03s

00:03–00:09  TEASE
  Visual : fragments/silhouettes of the product, never the full thing
  Motion : light leaks, partial reveals, parallax; tension rising
  Audio  : riser building toward the drop
  Cut    : quick cuts accelerating into the drop

00:09–00:13  REVEAL  ← THE DROP at 00:09
  Visual : product/logo SNAPS to full on the drop frame
  Motion : scale 1.18→1.0 over 0.18s + white flash-out; settle clean
  Audio  : drop hits + impact stinger on the exact frame
  Hold   : let it sit ~1.2s before moving

00:13–00:25  FEATURE MONTAGE
  Visual : one benefit per shot, ~0.6–1.0s each (≈8–10 shots)
  Motion : bold keyword pop + one supporting visual; consistent enter/exit
  Audio  : hard cuts on kick/snare; energy sustained
  Pattern: KEYWORD + UI/product moment, repeat with rhythm

00:25–00:30  END CARD
  Visual : logo + tagline + CTA, centered, still
  Motion : minimal — a soft settle, then hold ≥2s
  Audio  : music resolves; clean sonic tag
  CTA    : one action (URL or button), nothing competing
```

### Re-proportioning

- **15s**: Hook 0–2s · Tease 2–5s · Reveal 5–7s · Montage 7–12s (4–5 shots) · End card 12–15s.
- **60s**: same hook (still ≤3s) · longer tease and a richer montage (14–16 shots, grouped in waves) · end card ≥3s.

The hook never gets longer — only the montage scales.

## Beat-synced reveal (timing detail)

```js
import gsap from "gsap";
const DROP = 9.0;                 // measured drop timecode (s)
const tl = gsap.timeline({ paused: true });

// build tension toward the drop
tl.fromTo(".tease", { scale: 1.0, opacity: .6 },
                    { scale: 1.08, opacity: 1, duration: DROP, ease: "power1.in" });

// REVEAL exactly on the drop
tl.set(".logo",  { opacity: 1 }, DROP);
tl.fromTo(".logo",  { scale: 1.18 }, { scale: 1.0, duration: .18, ease: "power3.out" }, DROP);
tl.fromTo(".flash", { opacity: .9 },  { opacity: 0, duration: .25, ease: "power2.out" }, DROP);
tl.to(".tease", { opacity: 0, duration: .12 }, DROP);

// hold the reveal
tl.to(".logo", { scale: 1.0, duration: 1.2 }); // dwell

audio.addEventListener("play", () => tl.play());
```

Speed-ramp into the drop (Remotion time-remap idea): play the last ~0.5s of the tease faster, then hard-cut to the reveal on the drop frame for a snap.

## Kinetic montage shot recipe

Each shot: a bold keyword + one product/UI visual, ~0.6–1.0s, hard cut on the beat.

```css
.feature { position:absolute; inset:0; opacity:0; }
.feature.in { opacity:1; }
.feature .word { display:inline-block; opacity:0; transform:translateY(26px); }
.feature.in .word { animation: pop .32s cubic-bezier(.22,1,.36,1) forwards; }
.feature.in .word:nth-child(2){ animation-delay:.04s }
.feature.in .word:nth-child(3){ animation-delay:.08s }
@keyframes pop { to { opacity:1; transform:none; } }
```

```js
// schedule shots on measured beat marks
const beats = [13.0, 13.8, 14.6, 15.4, 16.2, 17.0, 17.8, 18.6]; // seconds
let i = 0;
function loop(now){
  const t = (now - start) / 1000;
  if (i < beats.length && t >= beats[i]) { showFeature(i); i++; }
  if (i < beats.length) requestAnimationFrame(loop);
}
```

Montage discipline: keep ONE enter curve and ONE exit across all shots; vary content, never grammar. Alternate visual rhythm (full-frame ↔ detail) but keep cut cadence locked to the beat.

## Multi-aspect safe-area maps

Compose so a single master reframes to every aspect. Place hero/logo/CTA inside the smallest common safe zone — the 1:1 center square.

```
16:9  1920×1080
  Safe: center 90% (96px H / 54px V margins)
  Use : YouTube, landing hero, X timeline

9:16  1080×1920
  Safe: text within center 80% height
  Avoid: top 12% (status/clock area), bottom 18% (caption + UI buttons)
  Use : Reels, TikTok, Shorts, Stories

1:1   1080×1080
  Safe: this square == the design anchor; put logo/CTA here
  Use : feed posts
```

Export workflow:

1. Build the 16:9 master with all key content inside the 1:1 center square.
2. For 9:16, **reframe** (pan/scale per shot to keep the subject centered) — do not letterbox.
3. For 1:1, crop the center square; verify text isn't clipped.
4. Re-check captions/CTA against each platform's UI overlay zones before export.

## Final QA checklist

- [ ] Reveal lands on the exact drop frame, with overshoot + flash.
- [ ] Hook holds attention in the first 3s without showing the product.
- [ ] Every montage shot is one benefit, premium quality, on a beat.
- [ ] End card holds ≥2s, single CTA, nothing competing.
- [ ] 16:9 / 9:16 / 1:1 each pass safe-area and caption checks.
- [ ] Audio mixed: riser → impact on drop → clean tag on end card.

---
Nail the reveal beat and the sound-leads-picture cut, and a launch film lands. Built by **[iart.ai](https://iart.ai/?utm_source=github&utm_medium=readme&utm_campaign=ad-video-skills&utm_content=skill_footer&utm_term=launch-video)** — the AI motion agent for editable, on-brand motion graphics.
