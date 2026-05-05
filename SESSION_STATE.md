---
workflow_step: live_polish_pass_landed
agent_type: execute
token_budget: deep
last_updated: 2026-05-04
---

# la route — sound machine instrument

## What this is
A browser-native sound machine / live performance instrument. Single HTML file at `~/Developer/acid-303/studio_v2_20260503.html`, no build step, no backend. Five module surfaces (drone + drums + texture + didgeridoo + bassline) at the new rack order, 33 scenes, manual crossfade timeline, in-browser MP3 recording. Renamed to **la route** as homage to Kangding Ray's track of the same name (from *Sirât*, 2025).

## Status
**v2 deployed, custom domain live, post-launch polish pass landed.**
- Live at https://laroute.ciamac.com/ (Cloudflare proxy ON / orange cloud, working).
- Default URL still works: https://la-route-livid.vercel.app/
- All four session commits pushed to `cia-mac/la-route` main, deployed via Vercel.

## Today's session deltas (post-DNS-landing)

### UX
1. **Module rack reorder.** New order: drone → drums → texture → didge → bassline. Cia's reasoning: "turn them on one by one" with pad foundation first, rhythm second, color layers, bass last. Updated everywhere: rack-num labels (/01–/05), quick-nav anchors, keyboard shortcuts (1–5), mixer order, mixer num labels. Onboarding-card anchor updated from acid → drone.
2. **Bigger pass.** Every font tier bumped (10→12, 11→13, 12→14, 13→15, 14→16, 15→17, 16→19, 17→20, 18→22, 19→23, 22→27, 24→30, 36→44, 52→64). Container max-width 1480 → 1640. Side padding 24/32 → 32/40.
3. **First-use overlay removed.** The four-card semitransparent guides got in the way more than they helped. Pulse on play-all stays under a fresh localStorage key `sm-pulse-dismissed-v1`. Dismisses on first scene/play/rec click.

### Mix balance (v2 loudness neutralization v3)
4. **Drone level cut twice this session: 0.38 → 0.24 → 0.14.** Cia complained drone was too loud relative to drums. Drone vs drums separation is now ≈ −16 dB, drone sits firmly underneath.
5. **Didge cut from 0.40 → 0.26.** Sustained source, treated like drone.
6. **Texture nudged from 0.58 → 0.50 → 0.46** to keep the pocket as drone got quieter.
7. **Drums hold at 0.92, bassline at 0.38.** Unchanged.

### Audio bug fixes (the audit)
8. **Drone voice stacking on rapid stop→play (CRITICAL).** `stop()` schedules voice cleanup via setTimeout (1.5–5.8s). If user clicked play during that window, prior voices kept feeding the master that was ramping back up. Doubled / stuck-sounding pad. Added `killPendingStopVoices()` called from play() and stop().
9. **Drone zombie voices on scene change during stop.** `applyPitchLive`'s setTimeout always built new voices, even if user pressed stop during the 550ms duck. New voices stacked silently in a stopped master, then woke up on next play. Timer now checks `state.playing`; aborts cleanly instead of building.
10. **Drone hang from stuck voicesBus (CRITICAL).** `voicesBus.gain` ramps to 0.001 during `applyPitchLive`'s fade-out. If user pressed stop during that fade, the fade-back-up branch never ran. voicesBus stuck at 0.001 = 60 dB attenuator. Next play came up silent / "hung." Now `play()` resets voicesBus.gain to 1, and the aborted-pitch branch does the same.
11. **Grain (texture) play-after-stop deadlock.** `state.playing` was set to false inside the deferred timeout, so `play()`'s early-return guard misfired during the release window. Clicking play right after stop did nothing for ~1s. Now `state.playing = false` flips immediately; deferred timer just clears the interval and short-circuits if user resumed.
12. **Fold state for didge not persisted.** Restore list at line 2498 was `['acid','drone','ritual','grain']` — missing 'didge'. Added.

### What's still as-designed
- **Drone evolve runs CPU work even when drone is stopped.** Audibly nothing — params evolve silently, drone resumes with fresh values. Intentional, left as-is.
- **Drone stop fade is up to 5.6s** (release = max(1.5, attack × 0.7)). Slow by design for pads. If Cia ever asks for snappier stops, scale that down.
- **Cloudflare proxy is ON, not OFF.** Vercel-assumed SSL is replaced by Cloudflare's. Empirically working. If audio ever misbehaves (Rocket Loader / auto-minify), flip the cloud gray.

## Git this session (cia-mac/la-route, all pushed)
- `0ed2c87` fix: drone/grain stop bugs + reorder rack + bigger type
- `848c1c6` remove first-use overlay; keep play-all pulse only
- `b176df1` fix: drone hang from stuck voicesBus + cut drone/didge levels
- `1c55d69` fix: restore fold state for didge

## Open loops
- **Optional: gallery tile.** Add a `web` tile to ciamac-gallery-stage linking to laroute.ciamac.com via the stage clone, then promote to live per the gallery release protocol.
- **Optional: full didgeridoo module.** Currently presets within the drone module *plus* a separate didge module. The full-blown rack section is here — but if Cia wants more rhythmic mouth pulses / throat formant control / animal call modes, that's a feature pass.
- **Optional: URL hash deep linking.** Encode active scene + cues + tempo + master vol into the URL hash so sharing a link recreates a specific vibe. ~30–60 min of work.
- **Optional: OG image PNG.** Current `og.svg` doesn't render on Twitter / FB / LinkedIn link previews. Export a 1200×630 PNG, swap meta. Biggest single win for "share this link" stories.
- **Optional: self-host lamejs.** Currently loaded from cdn.jsdelivr.net. If blocked or down, MP3 recording fails (rest of app fine). 50 KB self-host fixes it.
- **Optional: em-dash sweep.** `<title>` and `og:title` use "la route — ciamac" (em dash). Violates CLAUDE.md "no em dashes" rule. One-line fix.
- **Strategic: styleguide v4.4 alignment.** v4.4 retired Cadmium Yellow for Persian Rust #A2462F, partially obsoleting the brand-mismatch rationale (decision #10) for la route's subdomain. Two paths drafted in conversation:
  - **A — full conformance:** treat la route as a /lab page, swap Bodoni → Fraunces, add CIAMAC wordmark top-left. Risk: instrument feels stiff.
  - **B — instrument carveout:** adopt v4.4 tokens + sanctioned faces + persistent wordmark, but preserve the dense scene grid + timeline + mixer because the instrument IS the page.
  - Cia parked this question. My recommendation was **don't redesign now** — current Bodoni + charred earth + rust is coherent on its own; spend energy on ship-readiness instead.

## Working rules (carry forward)
- File versioning is non-negotiable. Three locks preserved: studio.html (pre-v1 working), studio_v1_20260501.html, studio_v2_20260503.html. Plus SESSION_STATE_v1, v2, v3 (locked this turn before rewrite).
- Bodoni Moda (display) + Inter (body) typography unchanged.
- Recording filename `la-route-{ts}.mp3` unchanged.
- Saturation curve `k = amount * 5 + 0.5`. Master limiter 5:1 / -3dB / wider knee.
- Drive defaults capped at 0.32 for ritual + acid presets. Distortion is opt-in.
- Clang muted by default (volume 0). Re-enable per track manually.
- Subline must stay smooth (square wave, clean drive, cutoff 130, reso 0.15, envmod 0.08). The fart can't come back.
- **Mix levels (v3 baseline):** drone 0.14, drums 0.92, texture 0.46, didge 0.26, bassline 0.38. Sustained sources sit far below transient sources.
- **Module rack order:** drone → drums → texture → didge → bassline. /01–/05.
- Scene system + crossfade timeline are the front door — preserve and extend.

## Reference materials
- `~/Developer/dispatches/kr_track_Sirāt.wav` — Kangding Ray Sirât (full track, used by extractor pipeline)
- https://github.com/cia-mac/la-route — public repo, main branch
- ciamacparhizi-9083s-projects/la-route on Vercel
- Custom domain: https://laroute.ciamac.com/ (Cloudflare proxy ON, working)
- Default Vercel: https://la-route-livid.vercel.app/
