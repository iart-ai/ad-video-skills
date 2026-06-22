# Batch & Typography

How to feed many reviews into one card design, plus the quote-typography craft that separates a credible testimonial from a sloppy one.

## Review prop schema

One object per testimonial. Keep it flat so a CSV row maps to it directly.

```ts
type Review = {
  quote: string;        // verbatim — never paraphrased or trimmed mid-sentence
  highlight?: string;   // EXACT substring of `quote`; one phrase only
  rating: number;       // 0–max, halves allowed (4.5)
  max?: number;         // default 5
  name: string;         // required — a quote with no name is not social proof
  role?: string;
  company?: string;
  avatar?: string;      // real photo URL, or omit (never a stock face)
};
```

### Validation (fail loud, before rendering 50 videos)

```ts
function validate(r: Review): string[] {
  const errs: string[] = [];
  if (!r.quote?.trim()) errs.push("quote is empty");
  if (!r.name?.trim()) errs.push("name is required (no anonymous testimonials)");
  const max = r.max ?? 5;
  if (r.rating < 0 || r.rating > max) errs.push(`rating ${r.rating} out of 0–${max}`);
  if (r.highlight && !r.quote.includes(r.highlight))
    errs.push(`highlight "${r.highlight}" is not a substring of the quote`);
  return errs;
}
```

The `highlight ⊂ quote` check is the important one: it guarantees you only ever emphasize words the customer actually wrote. If the highlight isn't an exact substring, the emphasis silently disappears in the card — so block the render instead.

## CSV → props

Reviews usually arrive as a spreadsheet export. Map columns to the schema and coerce the rating.

```js
import { parse } from "csv-parse/sync";
import { readFileSync, writeFileSync, mkdirSync } from "node:fs";

// CSV columns: quote,highlight,rating,name,role,company,avatar
const rows = parse(readFileSync("reviews.csv"), { columns: true, trim: true });
mkdirSync("reviews", { recursive: true });

rows.forEach((row, i) => {
  const r = {
    quote: row.quote,
    highlight: row.highlight || undefined,
    rating: Number(row.rating),
    name: row.name,
    role: row.role || undefined,
    company: row.company || undefined,
    avatar: row.avatar || undefined,
  };
  const errs = validate(r);
  if (errs.length) { console.error(`Row ${i + 1} (${r.name}):`, errs); process.exit(1); }
  writeFileSync(`reviews/${String(i).padStart(3, "0")}-${slug(r.name)}.json`,
    JSON.stringify(r, null, 2));
});

const slug = (s) => s.toLowerCase().replace(/[^a-z0-9]+/g, "-").replace(/^-|-$/g, "");
```

## Batch render — one video per review

```bash
mkdir -p out
for f in reviews/*.json; do
  name=$(basename "$f" .json)
  npx remotion render Testimonial "out/$name.mp4" --props="$f"
done
```

Everything visual lives in the single `theme` object in the composition (colors, fonts, quote-mark glyph, spacing). Because only the props change per render, 50 testimonials stay pixel-for-pixel brand-locked while the words and score vary. To re-theme the whole set, edit one object and re-run the loop.

For multi-aspect (9:16 feed + 1:1 post + 16:9 site), register the composition at each size with a shared center-safe layout and loop over both `reviews/*.json` and the aspect list.

## Quote typography craft

Motion is the easy part; legible, well-set type is what makes the quote trustworthy.

### Smart quotes, always

Straight quotes (`"`) read as un-proofed. Convert to typographic quotes before rendering.

```js
const smarten = (s) => s
  .replace(/(^|[\s([{])"/g, "$1“").replace(/"/g, "”")   // “ ”
  .replace(/(^|[\s([{])'/g, "$1‘").replace(/'/g, "’")   // ‘ ’
  .replace(/--/g, "—");                                       // em dash
```

### Hanging quotation marks

A leading quote glyph indents the first line and breaks the flush-left edge. Hang it into the margin so the text block stays optically aligned — the gold standard for set quotes.

```css
blockquote { hanging-punctuation: first; }            /* Safari/WebKit */
/* Cross-engine fallback: pull the opening mark left by its own width */
blockquote::before { content: "\201C"; margin-left: -0.55em; color: var(--muted); }
```

When the card uses a large decorative quote mark (as in `quote-card.md`), drop the inline opening glyph so the mark isn't doubled.

### Splitting into lines / clauses

Reveal in reading order means splitting on meaning, not on a fixed character count. Prefer clause boundaries (`. ! ? , ; —`), then pack to a comfortable measure (~32–40 chars at card size). The `toLines` helper in `quote-card.md` does exactly this. Never split a short quote into one-word-per-line — that reads as a ransom note, not a testimonial.

### Orphans & widows

A single dangling word on the last line looks broken. Bind the last two words with a non-breaking space so they wrap together.

```js
const noOrphan = (s) => s.replace(/ ([^ ]+)\s*$/, " $1");
```

### Legibility in motion

- Use a font that stays readable while moving — avoid thin weights and tight tracking; a serif or a humanist sans at 500–600 weight holds up best on a rising line.
- Animate **opacity + a small translate** only. Blur, rotation, and scale on body text hurt readability mid-reveal.
- Contrast: body text ≥ 4.5:1 against the card; the highlight accent should pass against both the card and the ink it sits behind.
- One emphasized span per quote. A second highlight halves the weight of the first.

## Star-rating accessibility & correctness

- The visual stars are decorative; expose the value to assistive tech with `role="img"` and `aria-label="4.5 out of 5 stars"` on the container (already in `quote-card.md`).
- For fractional scores, the width-clip method (`rating / max * 100%`) is exact for any decimal — no rounding to the nearest half-star, so 4.3 renders as 4.3.
- Never animate the stars before the quote is read, and never show more filled stars than the data supports. A faked rating is the fastest way to lose the trust the testimonial was meant to build.
- If a review has no rating, omit the star row entirely rather than defaulting to 5 — an invented score is worse than none.

## Pacing notes

- Hold the finished card still for the back half of the clip (≥3s) so it's screenshot-able and re-shareable — shares are the whole point of social proof.
- For a sequence of testimonials in one video, give each card a full read (~3–4s) and hard-cut between them; cross-dissolving quotes makes both unreadable.
