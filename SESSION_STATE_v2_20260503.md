---
workflow_step: v2_landed_sound_validated
agent_type: execute
token_budget: deep
last_updated: 2026-05-03
---

# la route — sound machine instrument

## What this is
A browser-native sound machine / DAW-lite. Single HTML file at `~/Developer/acid-303/studio_v2_20260503.html`, no build step, no backend. Loads in any modern browser. Originally an "can you build a 303?" experiment, now a Sirât-flavored live performance instrument: bassline + drone + drums + texture + mixer + recording + scene timeline, all on one shared Web AudioContext. Named **la route** as homage to Kangding Ray's track of the same name.

## Status
**v2 locked** at `studio_v2_20260503.html`. v1 lock and v1 working file preserved untouched (`studio_v1_20260501.html`, `studio.html`). Latest version is the working file. Open at http://localhost:8106/studio_v2_20260503.html (server: `python3 -m http.server --directory ~/Developer/acid-303 8106`). Sound validated by Ciamac on 2026-05-03 PT: "wonderful sounds, so much better".

## v2 changes (delta from v1)
1. **Sirât palette + grain.** Charred earth (#0a0805), bone white (#ebe2d4), burnt sienna (#d6541d), dust magenta (#c43a6a). SVG fractal-noise overlay across the viewport. Warm halo top, magenta dust bloom bottom. Same Bodoni Moda + Inter typography.
2. **Vertical rack layout.** Module tabs deleted. All four modules stack top-to-bottom in a rack. Each has a header with /XX number, italic name, play toggle, fold chevron. Fold state persists in localStorage.
3. **Per-module signal colors.** bassline amber (#d68c2a), drone dust magenta (#c43a6a), drums bone (#ebe2d4), texture ember red (#a8392e). Colored left edge stripes on each rack head.
4. **Clang killed.** 26 preset references zeroed. Default track volume dropped from 0.5 to 0.12. Metallic-FM percussion can't trigger by default.
5. **Drives halved + clamped at 0.32.** Every ritual + acid preset master drive and per-track drive halved (`v *= 0.5`, max 0.32). Loud master volumes capped at 0.78. Brings everything to the la route / dawn zone Ciamac liked.
6. **Per-channel mixer defaults rebalanced.** bassline 78%, drone 52%, drums 92%, texture 62%. Sustained sources (drone, texture) cut to compensate for perceived loudness vs transient sources.
7. **Church organ added to drone.** Three new presets: `organ` (cathedral pipe organ, slow attack, fifth+maj7), `orgel` (smaller chamber organ, intimate), `reedstop` (buzzy nasal oboe-like).
8. **Six new drum kits.** `rave`, `tribal`, `industrial`, `glass`, `desert`, `ritual` — all combinations of existing voice variants.
9. **Eight new scenes.** `liturgy`, `vespers`, `wadi`, `storm`, `diesel`, `moth`, `trance`, `ash`. liturgy + vespers use the new organ presets. Total 25 scenes (5×5 grid).
10. **Scene icons.** 21 monoline SVG glyphs assigned across all 25 scenes. Inline 14px, currentColor in burnt sienna.
11. **Timeline / cue sequencer (headline feature).** Drag scenes onto timeline strip below scene grid. Each cue has hold + transition durations (seconds). Transport: play / stop / loop / clear. On play: hold the current scene, then duck master volume to 35%, swap presets at midpoint, return master to 100%. Cues persist in localStorage. Live playhead sweeps. Reorder cues by dragging within timeline.

## Architecture (high level, unchanged from v1)
- Single ~5800-line HTML, four module IIFEs + Studio singleton + scene system + mixer + extractor + recording + **timeline IIFE (new)**.
- Studio singleton owns: shared AudioContext, masterBus → masterVolume → masterLimiter → destination, recording PCM tap, tempo source-of-truth, module registry. **New: `window._laRouteSetMasterGain` exposed for timeline ducking without touching user slider.**
- Modules (4): bassline, drone, drums, texture. Each registers play/stop API + a channel.
- Scene system: 25 cross-module presets that fire all four modules + tempo with one click. Now exposed via `window.LaRoute = { scenes, applyScene, iconSvg }` for timeline access.
- Beat extractor: drop audio file → 16-step pattern (unchanged).
- Mixer: 4 channels with normalized defaults (78/52/92/62), faders, mutes, solos, VU meters.
- Recording: PCM tap → lamejs mp3 192 kbps, 30-min cap, `la-route-{ts}.mp3` filename (unchanged).
- **Timeline (Phase 9):** cues array, render → DOM, drag-drop from `.scene-btn` to `.timeline-strip`, transport with requestAnimationFrame, duck-swap transition engine.

## Decisions made this session
1. Path B (full v2 in one cut) over Path A (feature-first). Coherent landing over churn.
2. Vertical rack replaces tabs. Mixer stays as a separate panel above the rack.
3. Per-module color stripes via CSS custom properties keyed off rack-bassline / rack-drone / rack-drums / rack-texture classes.
4. Drives halved globally rather than tuning each preset individually. Validated by ear.
5. Clang track default volume zeroed (opt-in only) rather than removed entirely.
6. Timeline transitions use duck-swap (master vol → 35% → swap presets at midpoint → restore) rather than parameter interpolation. Simpler, hides discrete swap behind audible duck.
7. Church organ implemented via square-wave drone preset with maj7 voicing, slow attack, huge reverb. Two more variants (orgel, reed stop) for chamber and nasal flavors.
8. 25 scenes (8 new) over keeping at 17. Fits 5×5 grid.

## Sibling files (unchanged)
- `index.html` — bassline standalone (stale, pre-v1)
- `drone.html` — drone standalone (stale)
- `ritual.html` — drums standalone (stale)

These are pre-studio versions kept for reference. The integrated experience is studio_v2.

## Open loops
- **Audio engine validation by ear at length** — Ciamac confirmed the v2 sound at first listen. Worth running through every scene over a longer session to catch any that still feel destroyed.
- **Timeline UX polish** — at preview-narrow viewport the timeline-head wraps. At desktop width it's fine. Could add a `flex-wrap: nowrap` constraint if it ever shows up at 1180px+.
- **Cue editing** — cue width is fixed at 156px. No per-cue scene swap (would need to be deleted and re-added). Inputs only allow hold + transition.
- **Mobile / iPad layout** — still desktop-only. Drag-drop on touch devices won't work without pointer-event handlers.
- **No git repo** — la route still uncommitted as a unit. Worth `git init` if v3 expands further.
- **Standalone module pages** — index.html / drone.html / ritual.html still stale. Keep or delete decision still pending.
- **Beat extractor** — kick band reliable, mid/hat fuzzy. Untouched in v2.

## Working rules (carry forward)
- File versioning is non-negotiable: every change is v1, v2, v3. v1 lock + v1 working + v2 working all preserved.
- Bodoni Moda (display) + Inter (body) typography unchanged. Don't switch without asking.
- Recording filename `la-route-{ts}.mp3` unchanged. Don't change without asking.
- Saturation curves use `k = amount * 5 + 0.5` (not `*30`). Master limiter at 5:1 / -3dB / wider knee. Both v1 settings preserved.
- Drive defaults capped at 0.32 for ritual + acid presets. Distortion is opt-in not opt-out.
- Clang muted by default (volume 0). Re-enable per track manually.
- Scene system + timeline are the front door. Preserve and extend.

## Reference materials
`~/Developer/dispatches/kr_track_Sirāt.wav` — Kangding Ray's Sirât (full track, used by extractor pipeline).
