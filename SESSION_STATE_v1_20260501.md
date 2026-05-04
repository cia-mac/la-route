---
workflow_step: v1_locked_ready_for_v2
agent_type: execute
token_budget: deep
last_updated: 2026-05-01
---

# la route — sound machine instrument

## What this is
A browser-native sound machine / DAW-lite. Single HTML file at `~/Developer/acid-303/studio.html`, no build step, no backend. Loads in any modern browser. Originally built as an iterative experiment ("can you build a 303?"), grew into a full instrument: bassline + drone + drums + texture + mixer + recording, all running on one shared Web AudioContext. Renamed to **la route** as an homage to Kangding Ray's track of the same name (the user described it as "the most perfect beat, I can listen to it on loop for a week").

## Status
**v1 locked** at `studio_v1_20260501.html`. Working copy is `studio.html`. Open in browser at http://localhost:8106/studio.html (start preview server: `python3 -m http.server --directory ~/Developer/acid-303 8106`). Default scene is "kr · sirât" (extracted from the actual Kangding Ray track via the in-browser beat extractor).

## Architecture (high level)
- Single `studio.html`, ~5000 lines, four module IIFEs + Studio singleton + scene system + mixer + extractor + recording.
- **Studio singleton** owns: shared AudioContext, master bus → master volume → master limiter → destination, recording PCM tap (lamejs mp3 encode), tempo source-of-truth, module registry.
- **Modules** (4): bassline (acid·303-style), drone, drums (8-track sequencer), texture (granular cloud). Each registers a play/stop API + a channel (gain, mute gain, analyser) for the mixer.
- **Scene system**: 17 cross-module presets that fire all four modules + tempo with one click. Saved active scene persists in localStorage.
- **Beat extractor**: drop any audio file → Web Audio decodes → bandpass into 3 bands → onset detection → autocorrelation tempo → phase-aligned 16-step pattern. Result populates the "extracted" drum preset.
- **Mixer**: 4 channels (one per module), each with fader, mute, solo, real-time VU meter.
- **Recording**: PCM tap via ScriptProcessor → lamejs mp3 encode (192 kbps). 30-min hard cap with toast notification. Output: `la-route-{timestamp}.mp3` to Downloads.

## Sound chain (signal path)
```
Each module → channel (mixer gain + mute + analyser) → masterBus → masterVolume → masterLimiter (5:1 / -3dB / soft knee) → destination
                                                                                   ↘ recording tap (PCM → lamejs)
```
Saturation curves were rewritten this session. Old: `k = amount * 30` (hard clip even at amount=0.2). New: `k = amount * 5 + 0.5` (linear at 0, mild at 0.5, moderate at 1.0). Distortion is now intentional, not accidental.

## Sibling files (same dir, single-page versions of individual modules)
- `index.html` — bassline standalone
- `drone.html` — drone standalone
- `ritual.html` — drums standalone
- (no separate texture standalone — granular only lives in studio.html)

These are pre-studio versions, kept for reference. The integrated experience is studio.html.

## Brand / typography
- **Title**: "la *route*." in Bodoni Moda italic (display serif), with "route" highlighted in accent orange (`#e85d1a`)
- **Body**: Inter Light (300) for everything else
- **Number labels**: Bodoni serif italic for /00, /01, /02 etc. — used for scene numbers, mixer channels, module tabs, panel sections
- **Color palette**: dark surface (`#0c0c0e` / `#131316`), warm orange accent, six preset color tags (warm/deep/bright/dark/driving/broken)
- Preset legend visible globally under the scene panel
- Tooltips have a 600ms hover delay and a tips on/off toggle (T key)

## Decisions made this session
1. Renamed all four modules to descriptive names (no fantasy): **bassline / drone / drums / texture**. Was acid·303 / drone·field / ritual·box / grain·cloud.
2. Renamed the app itself to **la route** (homage to the Kangding Ray track).
3. Saturation curves rewritten — `k = amount*5+0.5` not `amount*30`. Fixes the "everything sounds distorted" issue at default settings.
4. Master limiter softened: 5:1 / -3dB / wider knee / slower attack. Was 16:1 brick wall.
5. Bus low-shelf reduced (+1dB), high-shelf softened (-2dB). Was +2.5/-3.
6. Per-track default drives halved across the board. Default drum kit is now ~clean.
7. localStorage `sm-masterVol` bumped to `sm-masterVol-v2` to discard stale hot saved values.
8. **Drum kit selector** added (808 / 909 / dub / minimal / acoustic / custom). Swaps voice variants across all tracks at once.
9. **Per-track preview** button (▶) and **per-track voice variant** dropdown added in the drums machine — kick has 3 variants (boom/punch/dub), hat has 3 (metal/shaker/noise), clap has 3 (clap/rim/snap).
10. **Beat extractor** ships in-browser. Drop audio file → 16-step pattern.
11. Three new clean dumpfe presets: **soft / muffled / velvet**. All have drive=0.
12. **la route** preset rebuilt as the canonical clean reference: drive=0 across all tracks, master drive 0.05.
13. New scene **/16 kr · sirât** auto-loads the extracted Sirât pattern + matching modules.

## Recordings location
`~/Downloads/la-route-{timestamp}.mp3`. None saved this session (recording tested but no captures kept).

## Reference materials downloaded
`~/Developer/dispatches/kr_track_Sirāt.wav` — Kangding Ray's Sirât (full track, used by the beat extractor to validate the pipeline).

## What v2 should consider
The user wants v2. They didn't specify direction. Possible directions worth scoping with them first:
1. **More modules** — melodic lead, FX bus, tape simulation, sampler
2. **MIDI in/out** — control from a hardware controller
3. **Pattern memory** — save/recall multiple patterns per module, switch live
4. **Audio import** — drag a file in and play it as a one-shot (not just analyze for beat)
5. **A/B scene morphing** — interpolate between two scenes over time
6. **Visual feedback** — frequency analyzer, oscilloscope, spectrum
7. **Performance mode** — minimal UI, big play buttons, shortcut-driven
8. **Real DAW-lite features** — track recording (capture each module's output as separate stems), basic editing
9. **Sharing** — export the current scene state as a URL hash
10. **Mobile / iPad layout** — currently desktop-only, breaks on narrow widths

Ask Ciamac which direction. He values:
- thick / dumpfe / heavy aesthetics (la route, Kangding Ray, Sirât)
- editorial typography (Bodoni serif italic, his brand work showed similar)
- Swiss-modernist layout discipline
- no fantasy names, descriptive over evocative
- distortion-free by default, saturation as deliberate choice
- one-click vibes (the scene system is his front door)

## Open loops
- Beat extractor is approximate (low-band kick is reliable, mid/hat fuzzy in dense tracks). Could be improved with proper downbeat detection.
- The four standalone module pages (index, drone, ritual) are stale — they don't have the renames or any of this session's improvements. Keep or delete? They share the brand styling but lack everything else.
- No mobile/tablet layout. Screenshots verified at 1180px desktop only.
- No git repo. If v2 work expands significantly, init one.

## Working rules (Ciamac-specific)
- All repos live in `~/Developer/`. Never replace files — version them (v1, v2, v3).
- Decisions go to Notion Decision Log (la route / acid-303 isn't currently in the project map, may need to add).
- Console JSON dashboard (`http://localhost:8800/api/claude-response`) — la route isn't a tracked client project, doesn't need updates there.
- Fonts: Bodoni Moda (display) + Inter (body), both via Google Fonts CDN. Don't switch typefaces without asking.
- Recording filename is `la-route-{ts}.mp3`. Don't change without asking.
