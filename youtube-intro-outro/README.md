# YouTube Intro & Outro

Build the two pieces of motion that bracket every video on a channel: a 3-5s branded intro (a logo sting that says "this is my channel") and an outro whose last ~20s host YouTube's clickable end-screen elements (subscribe + next video). Both respect hard platform constraints — intro length affects retention, and outro layout must dodge YouTube's overlaid UI — and both come from one reusable template where only the brand changes. For YouTubers packaging a consistent channel identity.

## When it activates

- "Make a YouTube intro," "create a channel intro," or "build a logo sting/bumper."
- "Design an outro / end screen / end card" with subscribe and next-video cards.
- "Make an intro animation" or "sync the logo reveal to an audio sting."
- "Build a reusable intro template" where only the brand swaps.

## Example prompts

- "Make a 3.5s intro where my logo snaps in on the sting's impact frame with a white flash."
- "Build a 1920×1080 outro plate that leaves the right two-thirds quiet for YouTube's elements."
- "Give me one intro/outro template that re-skins from a brand.json per channel."

## What's inside

- `SKILL.md` — the two hard platform rules, the windup/impact/settle logo sting, end-screen layout that respects YouTube's element zones, the reusable `brand` prop, a batch render pipeline, and an output checklist.
- `references/intro-outro.md` — full runnable Remotion compositions for intro and outro, a CSS/GSAP sting variant, an exact end-screen coordinate map for 16:9 (and 1080×1920 notes), the method for finding an audio sting's impact frame, and the brand-prop schema.

---
Part of **[Creator Channel Skills](../)** · Built by **[iart.ai](https://iart.ai)** — controllable Motion Graphics MP4 from a prompt or data.
