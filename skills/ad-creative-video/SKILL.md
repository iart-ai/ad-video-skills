---
name: ad-creative-video
description: This skill should be used when the user asks to "make a video ad", "create an animated ad", "build a performance/UGC-style ad", "batch-produce ad creative variations", "generate ad variants for A/B testing", "swap headline/offer/CTA across many ad versions", or "export one ad in multiple aspect ratios for Meta/TikTok/Reels". Covers ad hook structures, hook→CTA message-match, data-driven variant generation (1 template × CSV = N ads), multi-aspect export, and platform specs.
version: 0.1.0
---

# Ad Creative Video

Build ONE motion-graphics ad template, then mass-produce variants from a data table — different hooks, offers, CTAs, and products — so a performance marketer can A/B test dozens of versions across placements and aspect ratios. The craft is making the template fully data-driven and message-matched, so 40 variants stay on-brand and only the tested variable changes.

## When to use

- Performance/UGC-style video ads where the first 1.5–3s must stop the scroll.
- Batch variant generation: one template rendered against many rows (headline/offer/CTA/product swaps) for A/B testing.
- Multi-aspect export of the same ad for Meta Feed, Reels/Stories, TikTok, YouTube.

Not for sale/discount countdown promos, and not for testimonial/review-driven ads — those are different structures. Stay on the batch-variation, test-everything angle.

## The two rules that make variants worth running

1. **Isolate one variable per variant.** A test only teaches something if exactly one thing changes. Hold layout, motion, colors, and timing constant; swap *only* the field under test (hook, or offer, or CTA). Mixing two changes makes the winner uninterpretable.
2. **Message-match the hook to the CTA.** The promise made in the first 3 seconds must be the promise the button pays off. A "stop wasting 2 hours a day" hook ends on "Save 2 hours — try free," not a generic "Shop now." The hook and CTA are a matched pair in every row of the data table.

## Ad anatomy (15–30s)

| Beat | Job | Budget |
|---|---|---|
| Hook (0–3s) | Stop the scroll; state the problem or pattern-interrupt | 0–3s |
| Context | Make the viewer feel the problem / agitate | 3–8s |
| Payoff | Show the product as the solution, one clear benefit | 8–18s |
| Proof | One concrete number, demo, or result | 18–25s |
| CTA (last 3s) | Single action, message-matched to the hook | hold ≥2s |

Land the hook's emotional trigger before the 2-second mark — judgment forms in ~1.7s and scroll speed keeps rising. Test the hook FIRST; it moves CPA more than any other element.

## Hook taxonomy (the field you vary most)

| Hook type | Template | When it wins |
|---|---|---|
| Problem-first | "I didn't realize how much X was costing me until…" | pain is felt but unnamed |
| Pattern interrupt | "This is going to sound controversial, but…" | crowded feed, generic category |
| Curiosity gap | "Nobody talks about the one thing that…" | educated, skeptical audience |
| Immediate benefit | "Cut your X in half in 7 days" | clear, quantifiable outcome |
| Direct callout | "If you do X every morning, stop." | sharp audience segment |

Keep a column of 5–10 hook strings and let the batch render produce one ad per hook against the same body — that is the cleanest hook test possible.

## Data-driven template (everything is a prop)

Hardcode nothing the marketer might A/B test. The composition reads a single `variant` object; the renderer never edits the component. Make every animated value a pure function of `useCurrentFrame()` so each frame renders deterministically (no CSS transitions, no library timers — they desync the render).

```tsx
import {useCurrentFrame, interpolate, AbsoluteFill, useVideoConfig} from "remotion";

type Variant = {hook: string; benefit: string; proof: string; cta: string; bg: string; accent: string};

export const AdTemplate: React.FC<{v: Variant}> = ({v}) => {
  const frame = useCurrentFrame(); const {fps} = useVideoConfig();
  const hookIn = interpolate(frame, [0, 8], [40, 0], {extrapolateRight: "clamp"});
  const ctaIn  = interpolate(frame, [fps * 22, fps * 23], [0, 1], {extrapolateLeft: "clamp", extrapolateRight: "clamp"});
  return (
    <AbsoluteFill style={{background: v.bg, fontFamily: "Inter, sans-serif"}}>
      <h1 style={{transform: `translateY(${hookIn}px)`, opacity: interpolate(frame,[0,8],[0,1])}}>{v.hook}</h1>
      {frame > fps * 8  && <p>{v.benefit}</p>}
      {frame > fps * 18 && <strong>{v.proof}</strong>}
      <button style={{opacity: ctaIn, background: v.accent}}>{v.cta}</button>
    </AbsoluteFill>
  );
};
```

See `references/batch-ad-pipeline.md` for the complete template, the CSV→props parser with validation, and the brand-lock theme object.

## 1 template × CSV = N variants

The payoff: author the ad once, then let a CSV drive the matrix. Each row is one fully-formed, message-matched variant. Render once per row.

```bash
# variants.csv: id,hook,benefit,proof,cta,bg,accent  → one MP4 per row
node csv-to-props.js variants.csv ./props        # writes props/<id>.json per row
for f in props/*.json; do
  id=$(basename "$f" .json)
  npx remotion render AdTemplate "out/${id}.mp4" --props="$f"
done
```

To build a clean test matrix, hold every column constant and vary one: 5 hooks × 1 body = 5 ads (hook test); then take the winning hook × 3 CTAs (CTA test). `references/batch-ad-pipeline.md` has a matrix generator that expands variable lists into the CSV so combinatorial tests stay one-variable-at-a-time.

## Multi-aspect export

Each placement wants a native aspect. Compose against a center safe zone so one master reframes cleanly to all of them — don't letterbox, re-center.

| Aspect | Placements | Resolution | Keep-clear safe zone |
|---|---|---|---|
| 9:16 | Reels, Stories, TikTok In-Feed | 1080×1920 | top ~14%, bottom ~20–35% (UI/caption), sides ~6% |
| 4:5 | Meta Feed (best feed real estate) | 1080×1350 | center 90%; CTA out of bottom 10% |
| 1:1 | Feed, broad reach | 1080×1080 | center square survives every crop |
| 16:9 | YouTube, in-stream | 1920×1080 | center 90% |

Design the hook text, product, and CTA inside the **1:1 center square** so they survive every crop. Drive aspect from a prop and render each per variant:

```bash
for ar in 9x16 4x5 1x1; do
  npx remotion render AdTemplate "out/${id}-${ar}.mp4" --props="props/${id}.json" \
    --props-merge="{\"aspect\":\"${ar}\"}"
done
```

Keep critical content out of the 9:16 bottom third — that is where the platform stacks captions, the CTA button, and the username. `references/platform-specs.md` has exact pixel safe zones and per-platform duration/format limits.

## Output checklist

- One variable per variant; layout/motion/timing identical across a test.
- Hook lands its trigger before 2s; hook and CTA are message-matched in every row.
- Every animated value is a pure function of `useCurrentFrame()` — no CSS/library timers.
- Dataset is a prop; nothing testable is hardcoded; one template renders the whole CSV.
- Each variant exported 9:16 + 4:5 + 1:1 (+16:9 if needed), key content in the center square.
- Brand stays locked across all variants via a single theme object.

## Deliver & verify (rendered stills → MP4)

> **Packaged helper** (`scripts/`): tile your stills with `scripts/contact-sheet.sh sheet.png f-hook.png f-mid.png f-end.png`, then assert the encode with `scripts/probe-mp4.sh out.mp4 [WxH] [fps]`. See `scripts/README.md`.

**Output contract:**
- A Remotion ad template registered as `<Composition>` (+ zod `schema` + `defaultProps`), every animated value frame-driven (no CSS transitions / library timers / `Date.now()` / `Math.random()`).
- Deliverable = the rendered `out/*.mp4` per variant per aspect (plus the project + CSV, so the marketer re-renders on new rows).

**Verify loop — render stills → inspect → encode.** Cheap PNGs first, full encode only once they're clean. Render with the **shipped** props (the real row), not just `defaultProps`.

```bash
# Frame-exact stills across the hook→CTA arc, with a real variant's props
npx remotion still AdTemplate out/f-hook.png --frame=12  --props=props/v1.json   # hook readable < 2s
npx remotion still AdTemplate out/f-mid.png  --frame=300 --props=props/v1.json   # benefit/proof
npx remotion still AdTemplate out/f-cta.png  --frame=689 --props=props/v1.json   # CTA, message-matched
# inspect each: fidelity (hook / offer / proof / CTA text exact, brand bg+accent right)
#   AND artifacts (text overflow, off-canvas, CTA inside the 9:16 bottom third, missing font, wrong row binding)
```

**Multi-aspect / batch — verify one variant in EACH aspect before batch-rendering the matrix.** A layout bug repeats across every row × aspect; catch it once.

```bash
for ar in 9x16 4x5 1x1; do                                  # one representative variant, every target aspect
  npx remotion still AdTemplate "out/v1-${ar}.png" --frame=300 \
    --props=props/v1.json --props-merge="{\"aspect\":\"${ar}\"}"
done
# stills clean in all aspects? → then batch-render every row × aspect:
for f in props/*.json; do id=$(basename "$f" .json); for ar in 9x16 4x5 1x1; do
  npx remotion render AdTemplate "out/${id}-${ar}.mp4" --props="$f" --props-merge="{\"aspect\":\"${ar}\"}"
done; done
npx remotion render AdTemplate out/demo.gif --codec=gif --props=props/v1.json   # README demo
```

**Before you finish:**
1. `npx remotion still` renders cleanly at hook / mid / CTA — no errors, no missing fonts/assets.
2. Hook/offer/proof/CTA text exact and brand colors right; nothing in the 9:16 bottom third or outside the center safe zone.
3. Frame-driven only — no CSS/library timers, `Date.now()`, or `Math.random()`.
4. Shipped row's props render correctly (not just `defaultProps`); one variable per variant holds.
5. One variant verified in 9:16 + 4:5 + 1:1 before the batch; MP4s play; (optional) GIF for the README.

## Reference files

- `references/batch-ad-pipeline.md` — the full runnable Remotion ad template, the CSV→typed-props parser with validation, a one-variable test-matrix generator, the brand-lock theme object, and the complete batch + multi-aspect render script.
- `references/platform-specs.md` — exact 2025/2026 pixel safe zones, durations, aspect ratios, and format limits for Meta Feed/Reels/Stories, TikTok In-Feed, and YouTube, plus a hook→CTA message-match worksheet.
