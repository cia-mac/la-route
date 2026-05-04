# la route

A browser-native sound machine. Five modules, scene system, live crossfade timeline. An homage to Kangding Ray's track of the same name.

**Live:** https://laroute.ciamac.com

## Modules

- **bassline** — acid 303-style sequencer (saw / square, slide-driven, low-resonance worm presets)
- **drone** — multi-oscillator pad with detune / drift / wobble / shimmer + 26 presets including organ, didge-like reedstop
- **drums** — eight-track sequencer (kick / sub / clap / hat / open / tom / clang / noise) with 36 patterns and 25 voice-variant kits
- **texture** — granular cloud with shimmer / chord / bell / drone / noise sources
- **didgeridoo** — fundamental + sub + filtered noise + breath / vocal / throat LFOs, 7 presets

## Front door

Scene grid loads a full vibe across all five modules with one click. 36 scenes, color-tagged warm / deep / bright / dark / driving / broken. Drag a scene onto the timeline strip to add it as a cue. Drag the crossfade fader to morph live — tempo lerps across the fade so beats stay locked.

## Stack

Single HTML file, no build step. Web Audio API throughout. lamejs for MP3 recording at 192 kbps. Bodoni Moda + Inter via Google Fonts.

## Files

- `studio_v2_20260503.html` — the working file (root rewrite serves this)
- `studio_v1_20260501.html` — first lock (preserved per versioning rule)
- `studio.html` — pre-v1 legacy working file (preserved)
- `archive/` — pre-v1 standalone module pages (bassline, drone, drums)
- `vercel.json` — root rewrite + cache headers
- `og.svg` — Open Graph card
- `favicon.svg` — site icon
- `404.html` — lost the route page

## Local

```bash
python3 -m http.server --directory . 8106
open http://localhost:8106/studio_v2_20260503.html
```

## Credits

Ciamac Parhizi · 2026 · Los Angeles. Homage to Kangding Ray's *La Route* (from *Sirât*, 2025).
