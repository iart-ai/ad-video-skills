---
name: testimonial-video
description: This skill should be used when the user asks to "make an animated testimonial", "create a quote video", "build a review video", "turn a customer testimonial into a graphic", "animate a customer review", "make a social proof video", "add an animated star rating", or "turn a list of reviews into videos". Covers quote typography, staggered line reveals, kinetic emphasis on key phrases, animated star ratings, the author block (name/role/avatar/company), and templating one card across many quotes.
version: 0.1.0
---

# Testimonial Video

Animate a written quote, review, or testimonial into a short, trustworthy social-proof clip. The craft is typographic, not flashy: a clean quote card where the words land in reading order, the key phrase gets emphasis exactly when it should, the stars fill to the real score, and the author signs off. One card design, driven by props, becomes a video for every review on the list.

## When to use

- Customer review → a 6–12s vertical/feed clip (the headline use case).
- A pull-quote or testimonial graphic with author name, role, avatar, company.
- An animated star rating (including half/fractional scores like 4.5).
- A batch: a list/CSV of reviews → one video each, same template.

Do **not** reach here for sale/discount promos or generic ad creative — this skill is strictly the quote / social-proof angle.

## The one rule: trust before motion

A testimonial earns its job by feeling *real*, not produced. Restraint reads as credibility; over-animation reads as an ad people skip.

1. **Reveal in reading order.** The eye must finish a line before the next arrives. Stagger lines (or clauses), never all words at once, and never reorder for visual flair.
2. **Quote the words exactly.** Do not paraphrase, trim mid-sentence, or invent a rating. If you emphasize a phrase, it must be the customer's phrase.
3. **The author is the proof.** A quote with no attributable human is just a slogan. Name + role + company (and avatar if real) is non-negotiable.

## The anatomy of a quote card

| Element | Job | Timing |
|---|---|---|
| Quote mark motif | Signals "this is a quote" before words read | Frame 0, scales/fades in first |
| Quote body | The testimonial, revealed line-by-line | Staggered, ~6–10 frames apart |
| Emphasis span | The one phrase that sells | Lands *after* its line settles |
| Star rating | The score, filled to the real value | Fills left→right after the quote |
| Author block | Name / role / company / avatar | Signs off last, slides up |

Build in that order; reveal in that order. The viewer reads the quote, believes it, sees the score, then learns who said it.

## Staggered line reveal

Split the quote into lines (or clauses), then offset each one's entrance. In Remotion, drive every line off `useCurrentFrame()` so it renders deterministically — never a CSS transition or a JS timer, which desync from the frame renderer.

```jsx
import { useCurrentFrame, spring, useVideoConfig } from "remotion";

const QuoteBody = ({ lines, startAt = 8, stagger = 6 }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  return (
    <blockquote>
      {lines.map((line, i) => {
        const enter = spring({ frame: frame - startAt - i * stagger, fps,
          config: { damping: 200 } });           // 0→1, no overshoot
        return (
          <span key={i} style={{ display: "block",
            opacity: enter,
            transform: `translateY(${(1 - enter) * 14}px)` }}>
            {line}
          </span>
        );
      })}
    </blockquote>
  );
};
```

Keep one motion language: same easing, same 14px rise, every line. Speed reads as confidence only when it's consistent. See `references/quote-card.md` for the full composition.

## Kinetic emphasis on the key phrase

Pick the single phrase that carries the testimonial ("saved us 10 hours a week") and let it arrive *after* its line has settled — a highlight sweep, a weight shift, or a color pop. Emphasis competing with the reveal cancels both.

```jsx
const sweep = interpolate(frame, [hlStart, hlStart + 10], [0, 100],
  { extrapolateLeft: "clamp", extrapolateRight: "clamp" });
// a highlighter wiping in behind the phrase, left→right
<mark style={{ background:
  `linear-gradient(90deg, var(--accent) ${sweep}%, transparent ${sweep}%)`,
  color: "inherit", padding: "0 .08em" }}>{phrase}</mark>
```

Emphasize at most one span per quote. Two highlights is no highlight.

## Animated star rating (fractional)

Render the real score, including halves. Use one full row of star glyphs as a mask, then wipe a filled layer to `rating / max` width — this handles 4.5 as cleanly as 5.0, with no half-star sprite.

```jsx
const StarRating = ({ rating, max = 5, fillStart = 30, fillDur = 14 }) => {
  const frame = useCurrentFrame();
  const target = (rating / max) * 100;
  const w = interpolate(frame, [fillStart, fillStart + fillDur], [0, target],
    { extrapolateLeft: "clamp", extrapolateRight: "clamp" });
  return (
    <div className="stars" role="img" aria-label={`${rating} out of ${max} stars`}>
      <div className="track">★★★★★</div>
      <div className="fill" style={{ width: `${w}%` }}>★★★★★</div>
    </div>
  );
};
```

```css
.stars { position: relative; display: inline-block; font-size: 40px; letter-spacing: 4px; }
.track { color: rgba(255,255,255,.22); }
.fill  { position: absolute; inset: 0; overflow: hidden; white-space: nowrap;
         color: var(--accent); }   /* clipped by width → fractional fill */
```

Fill left→right after the quote reads. Never animate stars before the words — the score must feel earned, not pre-loaded. Always set `aria-label` to the real value.

## The author block

Sign off last. The block slides up under the stars and holds still — stillness is what makes attribution read as fact.

```jsx
<figcaption className="author">
  {avatar && <img src={avatar} alt="" className="avatar" />}
  <div>
    <span className="name">{name}</span>
    <span className="role">{role}{company && `, ${company}`}</span>
  </div>
</figcaption>
```

Use a real avatar or none — a generic placeholder face actively destroys trust. Name is the loudest line in the block; role/company is secondary weight.

## Template × reviews — one video per quote

The payoff: make every field a prop, hardcode nothing, then render once per review.

```jsx
export const Testimonial = ({ quote, highlight, rating, name, role, company, avatar }) => { /* … */ };
```
```bash
for f in reviews/*.json; do
  npx remotion render Testimonial "out/$(basename "$f" .json).mp4" --props="$f"
done
```

Keep colors/fonts/quote-mark/layout in one theme object so 50 testimonials stay brand-locked and only the words and score change. See `references/batch-and-typography.md` for the review schema, CSV→props parsing, quote-typography craft (smart quotes, hanging punctuation, line-splitting, orphans), and the batch script.

## Output checklist

- Lines reveal in reading order; one consistent enter motion, ~6–10 frame stagger.
- The quote is verbatim; exactly one emphasized span, landing after its line settles.
- Stars fill to the *real* score (halves supported) after the quote, with a correct `aria-label`.
- Author block (name + role + company, real avatar or none) signs off last and holds still.
- Every animated value derives from `useCurrentFrame()` — no CSS/JS timers.
- Every field is a prop; one template renders every review in the folder.

## Deliver & verify (rendered stills → MP4)

> **Packaged helper** (`scripts/`): tile your stills with `scripts/contact-sheet.sh sheet.png f-hook.png f-mid.png f-end.png`, then assert the encode with `scripts/probe-mp4.sh out.mp4 [WxH] [fps]`. See `scripts/README.md`.

**Output contract:**
- A Remotion `Testimonial` registered as `<Composition>` (+ zod `schema` + `defaultProps`), every value frame-driven (no CSS transitions / JS timers / `Date.now()` / `Math.random()`).
- Deliverable = the rendered `out/*.mp4` per review per aspect (plus the project + review data, so the user re-renders on new quotes).

**Verify loop — render stills → inspect → encode.** Cheap PNGs first; render with the **shipped** review's props, not just `defaultProps`.

```bash
# Frame-exact stills across the reveal order: quote → stars → author
npx remotion still Testimonial out/f-quote.png  --frame=30  --props=reviews/r1.json  # lines in reading order, emphasis span
npx remotion still Testimonial out/f-stars.png  --frame=48  --props=reviews/r1.json  # stars filled to the REAL score (e.g. 4.5)
npx remotion still Testimonial out/f-author.png --frame=215 --props=reviews/r1.json  # author block signed off, holding still
# inspect each: fidelity (quote VERBATIM, exactly one emphasized span, star fill matches rating, name/role/company exact)
#   AND artifacts (text overflow/orphans, off-canvas, author in the 9:16 bottom third, missing font, broken avatar)
```

**Multi-aspect / batch — verify one review in EACH target aspect before batch-rendering the list.** A layout bug (overflowing quote, clipped author) repeats across every review × aspect; catch it once.

```bash
for ar in 9x16 1x1 16x9; do                                  # one representative review, every aspect
  npx remotion still Testimonial "out/r1-${ar}.png" --frame=215 \
    --props=reviews/r1.json --props-merge="{\"aspect\":\"${ar}\"}"
done
# stills clean in all aspects? → then batch-render the list:
for f in reviews/*.json; do id=$(basename "$f" .json); for ar in 9x16 1x1 16x9; do
  npx remotion render Testimonial "out/${id}-${ar}.mp4" --props="$f" --props-merge="{\"aspect\":\"${ar}\"}"
done; done
npx remotion render Testimonial out/demo.gif --codec=gif --props=reviews/r1.json   # README demo
```

**Before you finish:**
1. `npx remotion still` renders cleanly at quote / stars / author — no errors, no missing fonts/avatar.
2. Quote is verbatim with one emphasized span; stars fill to the real score; name/role/company exact and inside the safe zone (clear of the 9:16 bottom third).
3. Frame-driven only — no CSS/JS timers, `Date.now()`, or `Math.random()`.
4. Shipped review's props render correctly (not just `defaultProps`); `aria-label` carries the real rating.
5. One review verified in 9:16 + 1:1 + 16:9 before the batch; MP4s play; (optional) GIF for the README.

## Reference files

- `references/quote-card.md` — a complete runnable Remotion `Testimonial` composition: staggered line reveal, key-phrase highlight sweep, fractional star fill, author block, theme object, and the timing map for a 9:16 card. Plus a dependency-free HTML/CSS variant.
- `references/batch-and-typography.md` — the review prop schema, CSV/JSON→props parsing and validation, quote typography craft (smart quotes, hanging quotation marks, line/clause splitting, orphan control, legibility in motion), star-rating accessibility, and the template×reviews batch render script for N videos.
