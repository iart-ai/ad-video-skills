# Quote Card — complete runnable component

A full testimonial card you can drop into a Remotion project, plus a zero-dependency HTML/CSS variant for non-Remotion pipelines. Both reveal in reading order, emphasize one phrase, fill stars to the real score, and sign off with the author.

## Timing map (9:16, 30fps, ~10s)

| Frame | Beat |
|---|---|
| 0–8 | Quote mark scales in; card settles |
| 8–48 | Quote body, line-by-line (8 + i×6) |
| 44–54 | Key-phrase highlight sweep (after its line settles) |
| 56–70 | Stars fill left→right to `rating/max` |
| 72–88 | Author block slides up, holds |
| 88–300 | Everything holds still (screenshot-able) |

The card holds motionless for the back half — stillness is what makes a testimonial read as a fact rather than an ad.

## Remotion composition

```tsx
import {
  AbsoluteFill, useCurrentFrame, useVideoConfig, spring, interpolate, Img,
} from "remotion";

export type Review = {
  quote: string;          // verbatim testimonial
  highlight?: string;     // exact substring of `quote` to emphasize
  rating: number;         // e.g. 4.5
  max?: number;           // default 5
  name: string;
  role?: string;
  company?: string;
  avatar?: string;        // real photo URL, or omit
};

const theme = {
  bg: "#0E1116",
  ink: "#F5F7FA",
  muted: "rgba(245,247,250,.55)",
  accent: "#FFC24B",
  font: "Inter, system-ui, sans-serif",
  quoteFont: "'Newsreader', Georgia, serif",
  quoteMark: "“",     // left double quotation mark
};

// Split a quote into clauses for reading-order reveal.
// Prefer sentence/clause boundaries; fall back to ~32-char wraps.
const toLines = (q: string): string[] => {
  const parts = q.match(/[^.!?,;]+[.!?,;]?\s*/g) ?? [q];
  const lines: string[] = [];
  let buf = "";
  for (const p of parts) {
    if ((buf + p).length > 34 && buf) { lines.push(buf.trim()); buf = p; }
    else buf += p;
  }
  if (buf.trim()) lines.push(buf.trim());
  return lines;
};

// Render a line, wrapping the highlight substring in <mark> if present.
const Line: React.FC<{ text: string; highlight?: string; sweep: number }> = ({
  text, highlight, sweep,
}) => {
  if (!highlight || !text.includes(highlight)) return <>{text}</>;
  const [before, after] = text.split(highlight);
  return (
    <>
      {before}
      <mark
        style={{
          background: `linear-gradient(90deg, ${theme.accent} ${sweep}%, transparent ${sweep}%)`,
          color: "inherit",
          padding: "0 .08em",
          borderRadius: 3,
          WebkitBoxDecorationBreak: "clone",
        }}
      >
        {highlight}
      </mark>
      {after}
    </>
  );
};

export const Testimonial: React.FC<Review> = ({
  quote, highlight, rating, max = 5, name, role, company, avatar,
}) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  const lines = toLines(quote);

  // Quote mark
  const markIn = spring({ frame, fps, config: { damping: 14 } });

  // Highlight sweep, after the body has revealed
  const hlStart = 8 + lines.length * 6 + 4;
  const sweep = interpolate(frame, [hlStart, hlStart + 10], [0, 100], {
    extrapolateLeft: "clamp", extrapolateRight: "clamp",
  });

  // Stars fill, after the quote
  const fillStart = hlStart + 14;
  const target = (rating / max) * 100;
  const starW = interpolate(frame, [fillStart, fillStart + 14], [0, target], {
    extrapolateLeft: "clamp", extrapolateRight: "clamp",
  });

  // Author block slides up last
  const authStart = fillStart + 16;
  const auth = spring({ frame: frame - authStart, fps, config: { damping: 200 } });

  return (
    <AbsoluteFill style={{ background: theme.bg, fontFamily: theme.font,
      padding: 120, justifyContent: "center", color: theme.ink }}>
      <div
        aria-hidden
        style={{ fontFamily: theme.quoteFont, fontSize: 200, lineHeight: .6,
          color: theme.accent, opacity: markIn, transform: `scale(${markIn})`,
          transformOrigin: "left top" }}
      >
        {theme.quoteMark}
      </div>

      <blockquote style={{ fontFamily: theme.quoteFont, fontSize: 64,
        lineHeight: 1.25, margin: "24px 0 48px", fontWeight: 500 }}>
        {lines.map((line, i) => {
          const enter = spring({ frame: frame - (8 + i * 6), fps,
            config: { damping: 200 } });
          return (
            <span key={i} style={{ display: "block", opacity: enter,
              transform: `translateY(${(1 - enter) * 14}px)` }}>
              <Line text={line} highlight={highlight} sweep={sweep} />
            </span>
          );
        })}
      </blockquote>

      {/* Stars */}
      <div role="img" aria-label={`${rating} out of ${max} stars`}
        style={{ position: "relative", display: "inline-block", fontSize: 52,
          letterSpacing: 6, marginBottom: 56 }}>
        <div style={{ color: "rgba(245,247,250,.20)" }}>{"★".repeat(max)}</div>
        <div style={{ position: "absolute", inset: 0, overflow: "hidden",
          whiteSpace: "nowrap", width: `${starW}%`, color: theme.accent }}>
          {"★".repeat(max)}
        </div>
      </div>

      {/* Author */}
      <figcaption style={{ display: "flex", alignItems: "center", gap: 20,
        opacity: auth, transform: `translateY(${(1 - auth) * 16}px)` }}>
        {avatar && (
          <Img src={avatar} alt=""
            style={{ width: 88, height: 88, borderRadius: "50%", objectFit: "cover" }} />
        )}
        <div style={{ display: "flex", flexDirection: "column" }}>
          <span style={{ fontSize: 38, fontWeight: 700 }}>{name}</span>
          <span style={{ fontSize: 30, color: theme.muted }}>
            {[role, company].filter(Boolean).join(", ")}
          </span>
        </div>
      </figcaption>
    </AbsoluteFill>
  );
};
```

Register it with a duration long enough for the back-half hold:

```tsx
<Composition id="Testimonial" component={Testimonial}
  durationInFrames={300} fps={30} width={1080} height={1920}
  defaultProps={{
    quote: "We shipped in days, not months. Onboarding saved us 10 hours a week.",
    highlight: "saved us 10 hours a week",
    rating: 4.5, name: "Dana Reyes", role: "Head of Ops", company: "Northwind",
  }} />
```

## Why these choices

- **`spring` with `damping: 200`** gives a clean rise with no overshoot — testimonials should feel composed, not bouncy. The quote mark uses lighter damping (14) for a touch of life on the one decorative element.
- **Highlight as a gradient `<mark>`** wipes a real highlighter behind the phrase without a second element or a mask, and survives line wraps via `box-decoration-break: clone`.
- **Stars as two stacked glyph rows clipped by width** render any fractional score (4.5, 4.3) with no half-star asset, and the filled row inherits the brand accent.
- **Author after stars** matches reading psychology: claim → proof → who. Reordering weakens trust.

## Zero-dependency HTML/CSS variant

For pipelines that capture a headless-browser timeline (or just a web embed). Animations are CSS `@keyframes` with per-line `animation-delay`; if rendering frame-exact video, prefer the Remotion version above so frames stay deterministic.

```html
<figure class="card" style="--accent:#FFC24B; --rating:4.5; --max:5">
  <div class="qmark" aria-hidden="true">&ldquo;</div>
  <blockquote>
    <span style="--i:0">We shipped in days, not months.</span>
    <span style="--i:1">Onboarding <mark>saved us 10 hours a week.</mark></span>
  </blockquote>
  <div class="stars" role="img" aria-label="4.5 out of 5 stars">
    <div class="track">★★★★★</div><div class="fill">★★★★★</div>
  </div>
  <figcaption>
    <img src="dana.jpg" alt="" class="avatar">
    <div><b>Dana Reyes</b><small>Head of Ops, Northwind</small></div>
  </figcaption>
</figure>
```

```css
.card { --bg:#0E1116; --ink:#F5F7FA; background:var(--bg); color:var(--ink);
  font-family:Inter,system-ui,sans-serif; padding:120px; max-width:1080px; }
.qmark { font:200px/.6 'Newsreader',Georgia,serif; color:var(--accent);
  animation:pop .4s cubic-bezier(.22,1,.36,1) both; }
blockquote { font:500 64px/1.25 'Newsreader',Georgia,serif; margin:24px 0 48px; }
blockquote span { display:block; opacity:0;
  animation:rise .5s cubic-bezier(.22,1,.36,1) both;
  animation-delay:calc(.27s + var(--i) * .2s); }
mark { background:linear-gradient(90deg,var(--accent) 100%,transparent 100%);
  color:inherit; padding:0 .08em; border-radius:3px;
  -webkit-box-decoration-break:clone; box-decoration-break:clone;
  animation:sweep .35s ease-out both; animation-delay:1s; }
@keyframes sweep { from { background-size:0 100% } to { background-size:100% 100% } }
.stars { position:relative; display:inline-block; font-size:52px; letter-spacing:6px; }
.track { color:rgba(245,247,250,.20); }
.fill { position:absolute; inset:0; overflow:hidden; white-space:nowrap;
  color:var(--accent); width:0; animation:fill .5s ease-out 1.3s forwards; }
@keyframes fill { to { width:calc(var(--rating) / var(--max) * 100%) } }
figcaption { display:flex; align-items:center; gap:20px; margin-top:56px;
  opacity:0; animation:rise .5s ease-out 1.8s both; }
.avatar { width:88px; height:88px; border-radius:50%; object-fit:cover; }
figcaption b { font-size:38px; } figcaption small { display:block; font-size:30px;
  color:rgba(245,247,250,.55); }
@keyframes pop  { from { opacity:0; transform:scale(.7) } to { opacity:1; transform:none } }
@keyframes rise { from { opacity:0; transform:translateY(14px) } to { opacity:1; transform:none } }
```

The CSS `mark` sweep uses `background-size` (animatable) rather than a gradient color-stop, since color-stop percentages don't animate in CSS. The star `--rating/--max` math is the same fractional fill as the Remotion version.

---
## Built by the team behind iart.ai

This skill is part of an open motion-graphics collection from iart.ai — the AI motion agent that turns data, scripts, and designs into editable motion graphics (Remotion → MP4). If you'd rather not hand-build this, iart.ai can turn a list of reviews/quotes into testimonial videos, swap text, and re-export. → [iart.ai](https://iart.ai/?utm_source=github&utm_medium=reference&utm_campaign=ad-video-skills&utm_content=ref_footer&utm_term=testimonial-video)
