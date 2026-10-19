# Lull: build notes

Personal, offline iPhone web app (installed to the home screen) for sleep, mantra and polyrhythm listening. Built 2026-10-08. Source files are saved alongside this note in `claude/lull-source/`.

## What it does
- **Voice**: record mantras (raw mic capture, trimmed and normalized), or import audio files; tick several to loop in order. Pause between repeats, speed (tape-style pitch), 1–4 copies that drift apart (Reich-style phasing), brightness (low-pass), ping-pong echo, shared convolution reverb (generated impulse).
- **Drone**: partials of the key in just intonation; each partial's volume swells at a rate taken from the rhythm (e.g. 3:4:5 → 1,3,4,5,1.5,2,2.5 swells per cycle), all peaking together at the top of each cycle. Depth, shape (smooth → pulsing), reverb.
- **Pulse**: one ring per rhythm number; tuned strikes per cycle on harmony notes. Soft/glass/bell, register, density, ring, variation (seeded drops over up to 4 cycles), reverb. Pre-rendered into a looping buffer aligned to the cycle clock.
- **Beats**: binaural (headphones), monaural, or pulsed (AM). Optional glide over the sleep timer. Framed honestly: texture, not brainwave claims.
- **Noise**: brown/pink, brightness, waves (one swell per cycle).
- **Global**: cycle length, rhythm (2:3 … 3:5:7:8), key, harmony (overtone, open fifths, minor, pentatonic).
- **Sleep**: timer with exponential fade, wind-down (darkens + voice recedes), overall brightness, lucid cues (TLR-style; chime or a mantra; start/interval/count/volume; practice button), dark screen (pure black, double-tap to return).
- **Presets**: Descent (default), Lucid night, Phase study, Clear mind, Deep drone; user presets saved locally.

## Architecture
- `engine.js`: Web Audio engine. All rhythm runs on the audio clock (oscillators + looped buffers + scheduled automation), so no JS timers are needed for sound. LFOs use phase-rotated PeriodicWaves so rebuilds stay aligned to the shared cycle origin `t0`. Live changes crossfade. Inaudible keep-alive noise (~-74 dB). Layer balance via `LAYER_MAX`.
- `app.js`: UI (schema-driven panels), state in localStorage, recorder, wake lock, resume overlay, visualizer (canvas orbit), presets.
- `store.js`: IndexedDB for recordings (Int16 PCM at 48 kHz), decode/trim/normalize, WAV encode.
- `sw.js`: offline cache (cache-first, background refresh). **Bump `VERSION` on every update.**
- Assets: Spectral 300/300-italic/400 woff2 from npm `@fontsource/spectral`; icons drawn with PIL (orbit motif).

## iOS constraints (checked Oct 2026)
- Home-screen web apps stop audio when the phone locks or the app leaves the screen. Workaround: Screen Wake Lock (works in home-screen apps since iOS 18.4) + black screen; session end releases the lock.
- `navigator.audioSession.type = 'playback'` so the silent switch doesn't mute Web Audio.
- iOS 26.0–26.1 had a home-screen-app Web Audio bug, reported fixed in 26.2.
- Resume after interruption restarts the engine at the same elapsed session time (more reliable than resume()).

## Hosting / updating
- Hosted via Netlify Drop (drag `lull.zip`, claim with a free account). Open the URL in Safari on iPhone → Share → Add to Home Screen. Open once online; then it works offline.
- To update: edit files, bump `VERSION` in `sw.js`, zip with `index.html` at the root, drag onto the site's Deploys page in Netlify. On the phone, open the app twice to pick up the new version.

## Tested (headless Chromium)
Offline renders of every layer (no NaN, no clipping, levels balanced: full mix ≈ -21 dB RMS); 13 live parameter changes mid-render with no clicks (max sample jump 0.019 without voice); binaural verified at 194/198 Hz; UI flows at 390×844: record (raw path), play, presets, dark screen, timer end, lucid cue then stop, offline reload with saved recordings. Not testable here: real iPhone audio, mic permission UI, wake lock.

## Ideas for later
- Export a seamless loop to a file for lock-screen playback in another player.
- Reverse-voice layer, octave-down shadow copy, per-mantra gaps.
- Sync voice loop length to whole cycles (fully periodic option).
