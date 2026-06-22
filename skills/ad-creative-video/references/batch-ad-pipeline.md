# Batch Ad Pipeline — full template, parser, matrix, render

This is the complete, runnable implementation behind the SKILL.md overview: a data-driven Remotion ad template, a CSV→props parser with validation, a one-variable test-matrix generator, a brand-lock theme, and the batch + multi-aspect render scripts. Hardcode nothing a marketer might want to A/B test.

## 1. The brand-lock theme

Keep every brand-controlled value in one object so 40 variants stay consistent and only the *tested* fields change per row. The variant data supplies copy and per-campaign accents; the theme supplies the locked look.

```ts
// theme.ts
export const theme = {
  fontFamily: "Inter, system-ui, sans-serif",
  ink: "#0B0B0F",
  paper: "#FFFFFF",
  radius: 20,
  hookSize: 76,    // px, on the 1080-wide master
  bodySize: 44,
  proofSize: 52,
  ctaSize: 40,
  pad: 80,         // outer padding; also the side safe margin
  durationInSeconds: 25,
} as const;
```

## 2. Aspect config

Drive layout from a single aspect prop so one composition reframes to every placement. Dimensions and the keep-clear bottom band differ per aspect.

```ts
// aspects.ts
export type Aspect = "9x16" | "4x5" | "1x1" | "16x9";
export const ASPECTS: Record<Aspect, {w: number; h: number; bottomSafe: number}> = {
  "9x16": {w: 1080, h: 1920, bottomSafe: 540}, // captions + CTA + username stack here
  "4x5":  {w: 1080, h: 1350, bottomSafe: 135},
  "1x1":  {w: 1080, h: 1080, bottomSafe: 110},
  "16x9": {w: 1920, h: 1080, bottomSafe: 110},
};
```

## 3. The data-driven template

Every animated value is a pure function of `useCurrentFrame()` — no CSS transitions, no GSAP/library timers (they desync deterministic rendering and flicker in the export). The component reads one `variant` plus an `aspect`; the renderer never edits this file.

```tsx
// AdTemplate.tsx
import {
  AbsoluteFill, useCurrentFrame, useVideoConfig, interpolate, spring, Sequence,
} from "remotion";
import {theme} from "./theme";
import {ASPECTS, Aspect} from "./aspects";

export type Variant = {
  id: string;
  hook: string;      // 0–3s scroll-stopper
  benefit: string;   // the single payoff claim
  proof: string;     // one concrete number / result
  cta: string;       // message-matched to the hook
  accent: string;    // per-campaign accent color
  bg: string;        // background color
};

export const AdTemplate: React.FC<{v: Variant; aspect: Aspect}> = ({v, aspect}) => {
  const frame = useCurrentFrame();
  const {fps} = useVideoConfig();
  const {bottomSafe} = ASPECTS[aspect];

  // Hook rises + fades in over the first 8 frames, holds, then eases up and out at 8s.
  const hookY = interpolate(frame, [0, 8], [48, 0], {extrapolateRight: "clamp"});
  const hookOpacity =
    interpolate(frame, [0, 8], [0, 1], {extrapolateRight: "clamp"}) *
    interpolate(frame, [fps * 7, fps * 8], [1, 0], {extrapolateLeft: "clamp", extrapolateRight: "clamp"});

  // CTA springs in for impact in the last 3s and holds.
  const ctaProgress = spring({frame: frame - fps * 22, fps, config: {damping: 14, stiffness: 120}});

  return (
    <AbsoluteFill style={{backgroundColor: v.bg, fontFamily: theme.fontFamily, color: theme.ink}}>
      <AbsoluteFill style={{padding: theme.pad, paddingBottom: bottomSafe, justifyContent: "center"}}>

        {/* HOOK 0–8s */}
        <Sequence durationInFrames={fps * 8}>
          <h1 style={{fontSize: theme.hookSize, fontWeight: 800, lineHeight: 1.05,
                      transform: `translateY(${hookY}px)`, opacity: hookOpacity}}>
            {v.hook}
          </h1>
        </Sequence>

        {/* BENEFIT 8–18s */}
        <Sequence from={fps * 8} durationInFrames={fps * 10}>
          <FadeUp delay={0}>
            <p style={{fontSize: theme.bodySize, fontWeight: 600}}>{v.benefit}</p>
          </FadeUp>
        </Sequence>

        {/* PROOF 18–22s */}
        <Sequence from={fps * 18} durationInFrames={fps * 4}>
          <FadeUp delay={0}>
            <strong style={{fontSize: theme.proofSize, color: v.accent,
                            fontVariantNumeric: "tabular-nums"}}>{v.proof}</strong>
          </FadeUp>
        </Sequence>

        {/* CTA last 3s, message-matched to the hook */}
        <Sequence from={fps * 22}>
          <button style={{
            alignSelf: "flex-start", marginTop: 40, padding: "22px 44px",
            fontSize: theme.ctaSize, fontWeight: 700, border: "none",
            borderRadius: theme.radius, color: theme.paper, backgroundColor: v.accent,
            transform: `scale(${interpolate(ctaProgress, [0, 1], [0.85, 1])})`,
            opacity: ctaProgress,
          }}>
            {v.cta}
          </button>
        </Sequence>
      </AbsoluteFill>
    </AbsoluteFill>
  );
};

const FadeUp: React.FC<{children: React.ReactNode; delay: number}> = ({children, delay}) => {
  const frame = useCurrentFrame();
  const o = interpolate(frame - delay, [0, 10], [0, 1], {extrapolateLeft: "clamp", extrapolateRight: "clamp"});
  const y = interpolate(frame - delay, [0, 10], [24, 0], {extrapolateLeft: "clamp", extrapolateRight: "clamp"});
  return <div style={{opacity: o, transform: `translateY(${y}px)`}}>{children}</div>;
};
```

## 4. Register the composition (aspect + variant as props)

```tsx
// Root.tsx
import {Composition} from "remotion";
import {AdTemplate} from "./AdTemplate";
import {ASPECTS} from "./aspects";
import {theme} from "./theme";

const defaultVariant = {
  id: "demo", hook: "Spending 2 hrs/day on reports?", benefit: "Auto-build them in one click.",
  proof: "Teams save 11 hrs a week.", cta: "Save 2 hours — try free", accent: "#4F46E5", bg: "#FFFFFF",
};

export const RemotionRoot = () => {
  const aspect = "9x16" as const; // overridden per-render via --props
  const {w, h} = ASPECTS[aspect];
  return (
    <Composition
      id="AdTemplate"
      component={AdTemplate}
      durationInFrames={theme.durationInSeconds * 30}
      fps={30}
      width={w}
      height={h}
      defaultProps={{v: defaultVariant, aspect}}
      // Resolve real dimensions from the incoming aspect prop at render time:
      calculateMetadata={({props}) => {
        const a = ASPECTS[(props as any).aspect ?? "9x16"];
        return {width: a.w, height: a.h};
      }}
    />
  );
};
```

`calculateMetadata` lets a single composition output any aspect from the `aspect` prop — no separate composition per ratio.

## 5. CSV → typed props, with validation

The dataset is the input. Parse, validate every row (a bad hex or empty CTA must fail loudly, not render a broken ad), and write one props file per variant.

```js
// csv-to-props.js   usage: node csv-to-props.js variants.csv ./props [aspect]
const fs = require("fs");
const path = require("path");

const [, , csvPath, outDir, aspect = "9x16"] = process.argv;
const REQUIRED = ["id", "hook", "benefit", "proof", "cta", "accent", "bg"];
const isHex = (s) => /^#([0-9a-f]{3}|[0-9a-f]{6})$/i.test(s);

const parseCSV = (text) => {
  const [head, ...lines] = text.trim().split(/\r?\n/);
  const cols = head.split(",").map((c) => c.trim());
  return lines.filter(Boolean).map((line) => {
    // simple split; wrap fields containing commas in double quotes in the CSV
    const cells = line.match(/("([^"]|"")*"|[^,]*)/g).filter((_, i, a) => i < a.length - 1);
    const row = {};
    cols.forEach((c, i) => (row[c] = (cells[i] ?? "").replace(/^"|"$/g, "").replace(/""/g, '"').trim()));
    return row;
  });
};

const rows = parseCSV(fs.readFileSync(csvPath, "utf8"));
fs.mkdirSync(outDir, {recursive: true});

const seen = new Set();
let errors = 0;
rows.forEach((row, n) => {
  const where = `row ${n + 2} (${row.id || "no-id"})`;
  REQUIRED.forEach((k) => { if (!row[k]) { console.error(`✗ ${where}: missing "${k}"`); errors++; } });
  if (seen.has(row.id)) { console.error(`✗ ${where}: duplicate id`); errors++; }
  seen.add(row.id);
  if (row.hook && row.hook.length > 60) console.warn(`⚠ ${where}: hook >60 chars may clip in 9:16`);
  ["accent", "bg"].forEach((k) => { if (row[k] && !isHex(row[k])) { console.error(`✗ ${where}: "${k}" not a hex color`); errors++; } });
  if (errors) return;
  fs.writeFileSync(path.join(outDir, `${row.id}.json`), JSON.stringify({v: row, aspect}, null, 2));
});

if (errors) { console.error(`\n${errors} error(s) — no broken ads written.`); process.exit(1); }
console.log(`✓ ${rows.length} variant prop files written to ${outDir}`);
```

Example `variants.csv` (one fully message-matched ad per row):

```csv
id,hook,benefit,proof,cta,accent,bg
v01_problem,Spending 2 hrs/day on reports?,Auto-build them in one click.,Teams save 11 hrs a week.,Save 2 hours — try free,#4F46E5,#FFFFFF
v02_interrupt,Your reporting tool is lying to you.,See the real numbers instantly.,Teams save 11 hrs a week.,See the real numbers — free,#DC2626,#0B0B0F
v03_curiosity,Nobody talks about this reporting trick.,One click builds the whole report.,Teams save 11 hrs a week.,Try the trick — free,#059669,#FFFFFF
```

Note the CTA mirrors the hook in every row — that is the message-match discipline encoded into the data.

## 6. Test-matrix generator — one variable at a time

A test only teaches something if exactly one field changes. This expands a base variant plus a list of values for ONE field into a CSV, so a hook test or a CTA test stays clean and combinatorial mistakes are impossible.

```js
// make-matrix.js   usage: node make-matrix.js > variants.csv
const base = {benefit: "Auto-build them in one click.", proof: "Teams save 11 hrs a week.",
              cta: "Save 2 hours — try free", accent: "#4F46E5", bg: "#FFFFFF"};

// Vary ONLY this field; everything else is held identical across the test.
const FIELD = "hook";
const values = [
  "Spending 2 hrs/day on reports?",
  "Your reporting tool is lying to you.",
  "Nobody talks about this reporting trick.",
  "Cut report time in half this week.",
  "If you build reports by hand, stop.",
];

const esc = (s) => (/[",]/.test(s) ? `"${s.replace(/"/g, '""')}"` : s);
const cols = ["id", "hook", "benefit", "proof", "cta", "accent", "bg"];
console.log(cols.join(","));
values.forEach((val, i) => {
  const row = {...base, [FIELD]: val, id: `${FIELD}_${String(i + 1).padStart(2, "0")}`};
  console.log(cols.map((c) => esc(row[c])).join(","));
});
```

To run a CTA test next, set `FIELD = "cta"`, paste the *winning* hook into `base.hook`, and list CTA variants. Same one-variable rule, next stage of the funnel.

## 7. Batch + multi-aspect render

Render every variant in every aspect a campaign needs. Concurrency keeps large batches fast.

```bash
#!/usr/bin/env bash
# render-all.sh   one MP4 per (variant × aspect)
set -euo pipefail

node make-matrix.js > variants.csv          # 1) build the one-variable test matrix
ASPECTS=("9x16" "4x5" "1x1")                 # placements this campaign needs

for ar in "${ASPECTS[@]}"; do
  node csv-to-props.js variants.csv "props/$ar" "$ar"   # 2) typed, validated props per aspect
  for f in "props/$ar"/*.json; do
    id=$(basename "$f" .json)
    npx remotion render AdTemplate "out/${id}_${ar}.mp4" \
      --props="$f" --concurrency=4 --log=error              # 3) render
  done
done

echo "Done. $(ls out/*.mp4 | wc -l) ad variants in ./out"
```

5 hooks × 3 aspects = 15 MP4s from one template and one command. Add a CTA stage and the same template produces the next test wave with zero new component code.

## Common pitfalls

- **Two variables changed at once** — the winner is uninterpretable. The matrix generator prevents this; keep using it.
- **Hook clips in 9:16** — long hooks overflow; the parser warns past 60 chars. Keep hooks short and high in the frame.
- **CTA hidden behind platform UI** — anything in the 9:16 bottom third gets covered by captions and the CTA button. The `bottomSafe` padding reserves that band.
- **Drift across variants** — accent/copy live in the data, everything else in `theme`. If a variant looks off-brand, the fix is the theme, not the row.
- **Animation flicker in export** — caused by CSS transitions or library timers. Every value must derive from `useCurrentFrame()`.

---
## Built by the team behind iart.ai

This skill is part of an open motion-graphics collection from iart.ai — the AI motion agent that turns data, scripts, and designs into editable motion graphics (Remotion → MP4). If you'd rather not hand-build this, iart.ai can mass-produce ad-creative variants from one template × a data table — change the text/data and re-export. → [iart.ai](https://iart.ai/?utm_source=github&utm_medium=reference&utm_campaign=ad-video-skills&utm_content=ref_footer&utm_term=ad-creative-video)
