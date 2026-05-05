---
workflow_step: deployed_awaiting_dns
agent_type: execute
token_budget: deep
last_updated: 2026-05-04
---

# la route — sound machine instrument

## What this is
A browser-native sound machine / live performance instrument. Single HTML file at `~/Developer/acid-303/studio_v2_20260503.html`, no build step, no backend. Five module surfaces (bassline + drone + drums + texture + didgeridoo via drone presets), 36 scenes, manual crossfade timeline, in-browser MP3 recording. Renamed to **la route** as homage to Kangding Ray's track of the same name (from *Sirât*, 2025).

## Status
**v2 deployed to production.** Working file is `studio_v2_20260503.html`. v1 lock + v1 working both preserved. Live at https://la-route-livid.vercel.app/ (Vercel default URL). Custom domain `laroute.ciamac.com` is **attached on Vercel but not yet resolving** — pending DNS CNAME setup at Cloudflare.

## Today's session deltas (post-v2 lock, all in studio_v2)
1. **Quick-access left nav.** 7 anchors (scene / timeline / mixer / 4 modules) with keyboard shortcuts 0–4. IntersectionObserver tracks active section. Visible at viewports ≥720px.
2. **Bassline channel** cut twice: 78% → 52% → 38%. Subline (la route default acid preset) rewritten as a smooth deep hum: square wave + clean drive + cutoff 130 + reso 0.15 + envmod 0.08 + decay 0.95 + slides on every step. The old "fart" character is gone.
3. **Acid preset apply** extended to honor `wave` and `drive` fields per preset (so subline switches to square + clean automatically).
4. **Drone evolve toggle** sits next to drone play. When on, walks 8 params (drift / wobble / rate / brightness / resonance / depth / shimmer / detune) over 12s windows. Sliders move with the audio. State persists.
5. **Drone level** cut from 52% to 38% so the bed sits underneath the drums.
6. **Manual crossfade timeline** replaces the timed transition. Each cue loops indefinitely. A horizontal fader morphs current → next: tempo lerps continuously (BPM stays locked across the fade), master ducks parabolically, presets swap at midpoint with undo if user drags back, finalizes at >0.985. Loop on by default. `cut →` button hard-cuts without crossfade. Active cue + next cue names render in Bodoni italic on either side of the fader.
7. **+ ~30 new presets** across all categories. Scenes 25 → 30 (added monolith / reliquary / glasshouse / eclipse / breakwater). Acid 12 → 17 (fathom / pendulum / undertow / mantra / pillar — all wormy, slides on every step, audible filter sweep). Drum patterns 31 → 36 (monolith / pulse / shuffle / hollow / ember). Drone 21 → 29 (monastery / tundra / thurible / lokus / phosphor + 3 didgeridoo presets: didge / didge·vocal / didge·growl, low D fundamental + breath pulse via LFO rate × depth). Grain 17 → 22 (crackle / breath / sand / kiln / frost). Drum kits 15 → 25 (glass-roll / sahara / brass / bunker / silk + roomy / taut / foamy / chant / machine).
8. **Deploy assets.** Vercel rewrite from `/` → `/studio_v2_20260503.html`. OG meta + Twitter card + theme color in studio_v2. og.svg (1200×630, Bodoni mark on charred earth + grain). favicon.svg (italic 'r' in burnt sienna on charred earth). 404.html ("lost the route") in same brand. README.md with module/preset summary and credits.
9. **Git + repo.** `~/Developer/acid-303` is now a git repo, public on GitHub at https://github.com/cia-mac/la-route. Three stale standalone HTMLs (index.html, drone.html, ritual.html — the pre-studio module pages) moved to `archive/`.

## Architecture (high level)
- Single ~5900-line HTML, four module IIFEs + Studio singleton + scene system + mixer + extractor + recording + timeline IIFE + quick-nav IIFE.
- Studio singleton owns: shared AudioContext, masterBus → masterVolume → masterLimiter → destination, recording PCM tap, tempo source-of-truth, module registry. Exposes `window._laRouteSetMasterGain` for timeline ducking and `window.LaRoute = { scenes, applyScene, iconSvg }` for timeline access.
- Timeline IIFE owns: cues array, render → DOM, drag-drop from `.scene-btn` to `.timeline-strip`, manual crossfade fader → tempo lerp + master duck + preset swap at midpoint, finalize at >0.985.
- Quick-nav IIFE owns: anchor click handlers (scrollIntoView smooth) + keyboard 0–4 shortcuts + IntersectionObserver active highlight.
- Drone IIFE owns: evolve engine — pickTargets every 12s, smoothstep lerp, refresh sliders, updateLive.

## Decisions made this session
1. Path B (full v2 in one cut) over Path A (feature-first).
2. Vertical rack replaces tabs.
3. Per-module color stripes via CSS custom properties.
4. Drives halved globally rather than tuning each preset individually.
5. Clang track default volume zeroed (opt-in only) rather than removed.
6. Timeline transitions use duck-swap, then refactored to manual crossfade in the same session.
7. Loop on by default.
8. Bassline default switched from saw to square + clean drive via per-preset wave/drive override.
9. Didgeridoo implemented as drone presets, not a separate rack module (low risk, fast ship; full module possible if Cia wants it).
10. Subdomain `laroute.ciamac.com` over subroute on the main site (brand mismatch — la route is Bodoni / charred earth, ciamac.com v2.3 is Helvetica / Caspian Deep).
11. Public GitHub repo at cia-mac/la-route.
12. Stale standalone HTMLs moved to `archive/` rather than deleted (per the "never delete, archive" rule).

## Open loops
- **DNS CNAME at Cloudflare.** Cia needs to add `laroute` CNAME → `cname.vercel-dns.com` (proxy OFF / gray cloud / DNS only) in the ciamac.com Cloudflare zone. Once propagated (under 60s typically), https://laroute.ciamac.com/ resolves and Vercel issues SSL automatically (~30s on first request).
- **Smoke test custom domain** after DNS lands. Audio plays after first click, scenes load, recording works, OG card previews properly on iMessage / Twitter / WhatsApp.
- **Optional: gallery tile.** Add a `web` tile to the gallery (via stage clone) linking to laroute.ciamac.com so visitors find la route from ciamac.com → WORK button.
- **Optional: full didgeridoo module.** Currently presets within the drone module. A separate rack section with its own engine (more rhythmic mouth pulses, throat formant control, animal call modes) is feasible but requires HTML + JS + mixer wiring + scene routing.
- **Optional: URL hash deep linking.** Encode active scene + cues + tempo + master vol into the URL hash so sharing a link recreates a specific vibe.
- Beat extractor still has fuzzy mid/hat detection (kick band reliable). Untouched.
- No mobile / touch support. Drag-drop and keyboard shortcuts both desktop-only.
- Cue blocks fixed at 156px width. No per-cue scene swap (delete + re-add).

## Working rules (carry forward)
- File versioning is non-negotiable: every working snapshot saved as v1, v2, v3. Three locks preserved: studio.html (pre-v1 working), studio_v1_20260501.html, studio_v2_20260503.html.
- Bodoni Moda (display) + Inter (body) typography unchanged. Don't switch without asking.
- Recording filename `la-route-{ts}.mp3` unchanged.
- Saturation curve `k = amount * 5 + 0.5`. Master limiter 5:1 / -3dB / wider knee.
- Drive defaults capped at 0.32 for ritual + acid presets. Distortion is opt-in.
- Clang muted by default (volume 0). Re-enable per track manually.
- Scene system + crossfade timeline are the front door. Preserve and extend.
- Subline must stay smooth (square wave, clean drive, cutoff 130, reso 0.15, envmod 0.08). The fart can't come back.

## Reference materials
- `~/Developer/dispatches/kr_track_Sirāt.wav` — Kangding Ray Sirât (full track, used by extractor pipeline)
- https://github.com/cia-mac/la-route — public repo, main branch
- ciamacparhizi-9083s-projects/la-route on Vercel
- Live (default): https://la-route-livid.vercel.app/
- Custom domain target: https://laroute.ciamac.com/ (pending DNS)
