---
name: youtube-intro-outro
description: This skill should be used when the user asks to "make a YouTube intro", "create a channel intro", "build a logo sting/bumper", "design an outro / end screen / end card", "add subscribe and next-video cards", "make an intro animation", "build a reusable intro template", or "sync the logo reveal to an audio sting". Covers the 3-5s branded intro, the last-20s end-screen outro inside YouTube's element safe zones, timing conventions, audio-sting sync, and a swap-the-brand template.
version: 0.1.0
---

# YouTube Intro & Outro

Build the two pieces of motion that bracket every video on a channel: a 3-5s branded **intro** (logo sting that says "this is my channel") and an **outro** whose last ~20s host YouTube's clickable end-screen elements (subscribe + next video). Both must respect hard platform constraints — intro length affects retention; outro layout must dodge YouTube's overlaid UI — and both should be one reusable template where only the brand changes.

## When to use

- Channel intro / bumper / logo sting (a 3-5s identity hit before content).
- Outro / end screen / end card (the last ~20s with subscribe + next-video cards).
- A reusable brand template: swap name, logo, colors, sting, and re-export.

## Two hard rules (platform, not taste)

1. **Keep the intro 3-5s, the logo hit 1-2s.** Anything longer than 5s causes measurable viewer drop-off, which hurts the video in the algorithm. The intro confirms the channel and gets out of the way.
2. **The outro must be ≥20s of held, quiet layout, and the video ≥25s total.** YouTube only allows end-screen elements in the final 5-20s, and the video must be at least 25 seconds long. Design the outro plate around where those elements land — never put your own content under them.

## Intro: the logo sting

A sting is three beats in ~3.5s: a **windup** (motion building tension), an **impact** (logo snaps to full on the audio hit), and a **settle** (micro-overshoot relaxing to rest). The brand mark must land exactly on the sound's impact frame, not near it.

| Beat | Window | Job |
|---|---|---|
| Windup | 0-0.8s | whoosh / build; logo not yet readable |
| Impact | ~0.8s | logo snaps to full on the audio hit + a white flash |
| Settle | 0.8-1.2s | overshoot relaxes to rest; tagline fades in |
| Hold | 1.2-3.5s | brand legible, then cut to content |

```jsx
import {useCurrentFrame, useVideoConfig, spring, interpolate, Img, Audio, staticFile} from "remotion";

export const LogoSting = ({brand}) => {
  const frame = useCurrentFrame();
  const {fps} = useVideoConfig();
  const IMPACT = Math.round(0.8 * fps);            // logo lands here, on the sting hit

  // overshoot scale: springs up past 1, settles to 1 (the "snap")
  const scale = spring({frame: frame - IMPACT, fps, config: {damping: 9, mass: 0.5}, from: 1.18, to: 1});
  const logoOpacity = frame < IMPACT ? 0 : 1;
  const flash = interpolate(frame - IMPACT, [0, 6], [0.9, 0], {extrapolateLeft: "clamp", extrapolateRight: "clamp"});

  return (
    <div style={{flex: 1, background: brand.bg, display: "flex", alignItems: "center", justifyContent: "center"}}>
      <Audio src={staticFile(brand.sting)} />        {/* impact of this file must sit at 0.8s */}
      <Img src={staticFile(brand.logo)} style={{width: 420, opacity: logoOpacity, transform: `scale(${scale})`}} />
      <div style={{position: "absolute", inset: 0, background: "#fff", opacity: flash}} />
    </div>
  );
};
```

`spring` overshooting from 1.18→1 *is* the snap; the white `flash` decaying over 6 frames sells the impact. Align the audio so its loudest transient sits on frame `IMPACT`. See `references/intro-outro.md` for the full composition, a CSS/GSAP variant, and how to find the impact frame in any sting.

## Outro: end-screen layout that respects YouTube's element zones

This is where most outros fail: creators put a logo or text exactly where YouTube will stamp a subscribe button. Treat the frame as **two thirds**. YouTube's clickable elements live on the **right two thirds**; keep your own branding, channel name, and any face-cam in the **left third**. Stay away from corners and the bottom — the mobile progress bar and UI overlap there.

| Element | Native size | Where to leave room (1920×1080) |
|---|---|---|
| Subscribe / channel circle | 294×294 | right side, vertically centered-ish, off the edge |
| Next-video card (thumbnail) | 615×345 | upper-right or center-right |
| Playlist / link card | 294×294 | pair beside the video card |
| Your branding (logo, name) | — | **left third only**, top-aligned |

Render **placeholders** in your outro plate where the live elements will sit, so you compose around them — then position the real elements over them in YouTube Studio (use its grid + "snap to element").

```jsx
import {useCurrentFrame, interpolate, Img, staticFile} from "remotion";

// 1920×1080 outro plate. Left third = brand; right two-thirds = YouTube element zones (drawn as guides).
export const EndScreen = ({brand}) => {
  const frame = useCurrentFrame();
  const fade = interpolate(frame, [0, 18], [0, 1], {extrapolateRight: "clamp"});
  const zone = {position: "absolute", border: "2px dashed rgba(255,255,255,.25)", borderRadius: 12};

  return (
    <div style={{flex: 1, background: brand.bg, opacity: fade}}>
      {/* LEFT THIRD: your brand — safe from YouTube's overlays */}
      <div style={{position: "absolute", left: 80, top: 90, width: 560}}>
        <Img src={staticFile(brand.logo)} style={{width: 220}} />
        <div style={{color: brand.fg, font: "700 56px Inter", marginTop: 24}}>{brand.name}</div>
        <div style={{color: brand.accent, font: "500 30px Inter", marginTop: 8}}>Subscribe for more →</div>
      </div>

      {/* RIGHT TWO-THIRDS: leave these empty; YouTube draws the live elements here */}
      <div style={{...zone, width: 615, height: 345, right: 90, top: 110}} />        {/* next-video card */}
      <div style={{...zone, width: 294, height: 294, right: 250, bottom: 150, borderRadius: 999}} /> {/* subscribe circle */}
    </div>
  );
};
```

Ship the dashed guides only in a preview build; the final render drops them, leaving a clean plate whose right side is intentionally quiet. The outro itself can hold a static brand or loop a subtle motion for the full 20s — but nothing that competes for attention with the cards.

## Reusable brand template

Both compositions read a single `brand` prop — no hardcoded names, colors, logos, or audio. Swap the object (or pass `--props`) to re-skin every channel intro/outro from one codebase.

```js
// brand.json — the only thing that changes per channel
{ "name": "PixelForge", "logo": "logo.png", "sting": "sting.mp3",
  "bg": "#0B0B14", "fg": "#FFFFFF", "accent": "#7C5CFF" }
```

```bash
# render intro + outro for every brand file in /brands
for f in brands/*.json; do
  name=$(basename "$f" .json)
  npx remotion render Intro  "out/${name}-intro.mp4"  --props="$f"
  npx remotion render Outro  "out/${name}-outro.mp4"  --props="$f"
done
```

Keep fonts, layout, and timing fixed in the template so 20 channels stay structurally consistent and only the brand identity differs.

## Output checklist

- Intro ≤5s; logo readable by ~2s; logo hit lands exactly on the audio impact frame.
- Outro is ≥20s of held layout; total video ≥25s so end-screen elements are allowed.
- Right two-thirds of the outro left quiet for YouTube's elements; own branding in the left third only.
- Nothing important within ~10% of any edge or in the bottom UI band.
- Up to 4 end-screen elements planned (1 video + subscribe is the highest-CTR layout).
- Every text/color/logo/audio comes from the `brand` prop — one template, many channels.

## Reference files

- `references/intro-outro.md` — the full runnable Remotion compositions for both intro and outro, a CSS/GSAP logo-sting variant, an exact end-screen coordinate map for 16:9 and notes for 1080×1920, the method for finding an audio sting's impact frame and syncing the reveal to it, and the brand-prop schema with the batch render pipeline.
