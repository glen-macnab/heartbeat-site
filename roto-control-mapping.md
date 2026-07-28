# HeartBeat × ROTO-Control — live-jam mapping

A mapping of HeartBeat onto the Melbourne Instruments ROTO-Control (8 motorized,
touch-sensitive knobs + 8 RGB buttons per page, 4 pages per MIDI setup). Two ROTO setups
that coexist in one project — **HEARTBEAT** (performance, channel 14) and
**HB CREATE** (music creation, channel 15, see below) — with one rule for every CC:

```
CC = 16 × (page − 1) + position        knobs = positions 1–8, buttons = 9–16
```

Page 1 is CC 1–16, page 2 is 17–32, page 3 is 33–48, page 4 is 49–64; the two setups reuse
the same numbers on their own channels. HeartBeat's learn table keys on channel + number,
so overlaps with conventional CC meanings (1 = mod wheel, 7 = volume) are harmless on
dedicated channels, and the two setups' bindings never collide — import both and flip
setups on the ROTO.

Two ready-to-import files ship with each setup (CREATE's are listed in its section):

- [`docs/perform-bindings/roto-heartbeat-bindings.json`](perform-bindings/roto-heartbeat-bindings.json) — all 64 HeartBeat
  bindings. Import in the app: Settings → MIDI Learn Bindings → ••• → Import Bindings.
  Imports merge (incoming wins on collisions) and land in the current project.
- [`docs/perform-bindings/roto-heartbeat-setup.json`](perform-bindings/roto-heartbeat-setup.json) — the ROTO-Setup side
  (labels, haptics, LED behavior). Import via ROTO-Setup → File → Import. Targets setup
  slot 64 (`index: 63`); format reverse-engineered from the official MIDI-HELPER template
  and verified structurally identical. The `colorScheme` palette indices (0–82) map to
  the ROTO-Setup color sheet in display order (row-major, 14 per row). The VOL 1–16 knobs
  and MUTE 1–16 buttons carry HeartBeat's default track colors (red, orange, amber,
  yellow, lime, green, mint, teal, cyan, sky, blue, violet, orchid, magenta, rose, pink —
  sheet indices 14, 15, 1, 17, 4, 19, 6, 20, 21, 22, 23, 24, 11, 26, 12, 0); other
  controls' colors are unverified guesses — retint in ROTO-Setup to taste.

The full design rationale and research notes are in the project artifact; this file is the
working reference.

## Why selection-relative

The ROTO has no SysEx: labels can't change at runtime and the app can't switch its
pages/setups. So page 1 doesn't map "track 3's cutoff" — it maps the *selected-track*
targets (`.knob(0–7)`). You pick the track in the app; the same eight knobs always mean the
same eight things for whatever track is in focus. The motors make this work: on track
change, HeartBeat pushes the new track's values and all eight knobs physically jump — no
pickup mode, no parameter jumps.

## Page 1 — SOUND (selected track)

| Control | Label | CC | Target | ROTO config |
|---|---|---|---|---|
| Knob 1 | `CUTOFF` | 1 | `.knob(0)` | smooth 300° |
| Knob 2 | `RESO` | 2 | `.knob(1)` | smooth 300° |
| Knob 3 | `TUNE/DETUNE` | 3 | `.knob(2)` | center detent (sampler TUNE centers at 64) |
| Knob 4 | `LVL/WIDTH` | 4 | `.knob(3)` | smooth 300° |
| Knob 5 | `DECAY/SUB` | 5 | `.knob(4)` | smooth 300° |
| Knob 6 | `PAN/DECAY` | 6 | `.knob(5)` | smooth 300° |
| Knob 7 | `DLY SEND` | 7 | `.knob(6)` | smooth 300° |
| Knob 8 | `REV SEND` | 8 | `.knob(7)` | smooth 300° |
| Button 1 | `PLAY` | 9 | `.playStop` | PUSH, LED white |
| Button 2 | `REC` | 10 | `.record` | PUSH, LED red |
| Button 3 | `FILL` | 11 | `.fill` | PUSH — **hold to fill**, LED amber |
| Button 4 | `TAP` | 12 | `.tapTempo` | PUSH |
| Button 5 | `DJ REPEAT` | 13 | `.djFX(5)` | PUSH — **hold to beat-repeat the master**, LED teal |
| Button 6 | `DJ TAPE` | 14 | `.djFX(6)` | PUSH — **hold to tape-brake, release to snap back**, LED violet |
| Button 7 | `UNDO` | 15 | `.undo` | PUSH (LED lights while undo is available) |
| Button 8 | `STEP PAGE` | 16 | `.pageNext` | PUSH |

Dual labels exist because sampler and synth machines assign knob slots 3–6 differently
(sampler: TUNE / LEVEL / DECAY / PAN; synths: DETUNE / WIDTH / SUB / DECAY). Slots 1–2 and
7–8 are cutoff, resonance, delay send, reverb send on every internal machine.

## Pages 2 & 3 — MIX (tracks 1–8, 9–16)

| Control | Label | CC (p2 / p3) | Target | ROTO config |
|---|---|---|---|---|
| Knobs 1–8 | `VOL 1`…`VOL 16` | 17–24 / 33–40 | `.mixerFader(0–15)` | smooth, **custom indent at value 100** (HeartBeat unity) |
| Buttons 1–8 | `MUTE 1`…`MUTE 16` | 25–32 / 41–48 | `.trackMute(0–15)` | TOGGLE (on 127 / off 0), LED red = muted |

Mark the mute bindings **TOG** in Settings → MIDI Learn Bindings so the app follows button
state. Mutes dispatch on the CoreMIDI thread — zero latency. Internal tracks fade via the
VOL param; MIDI tracks send CC 7 on their own channel.

If tracks 9–16 go unused, page 3 is the natural candidate to repurpose (master compressor
`.masterComp(0–4)` + pattern launch `.patternSlot(0–7)`), as a second ROTO setup.

## Page 4 — MASTER + MORPH (v3, 2026-07-27)

| Control | Label | CC | Target | ROTO config |
|---|---|---|---|---|
| Knob 1 | `DLY RET` | 49 | `.delayReturn` | smooth |
| Knob 2 | `REV RET` | 50 | `.reverbReturn` | smooth |
| Knob 3 | `MORPH` | 51 | `.sceneFader` | smooth (morph A→B crossfade; landing on an end snaps that morph's mutes) |
| Knob 4 | `MST VOL` | 52 | `.masterVolume` | smooth |
| Knob 5 | `LIM CEIL` | 53 | `.masterComp(5)` | smooth (0 → −24 dBFS) |
| Knob 6 | `LIM REL` | 54 | `.masterComp(6)` | smooth (5–305 ms) |
| Knob 7 | `VEL OFS` | 55 | `.velocityOffset` | **center detent** at 64 (±0; up un-hides ghost trigs — no on-screen control anymore) |
| Knob 8 | `DJ FILT` | 56 | `.djFX(0)` | **center detent** at 64 (off; down LP sweep, up HP sweep) |
| Buttons 1–8 | `MORPH 1–8` | 57–64 | `.scene(0–7)` | PUSH, LED lights when the slot holds a morph (per-pattern) |

Displaced to the touchscreen (sound design, not performance): delay
time/feedback/tone, reverb size/damp/predelay/width, compressor detail,
sidechain routing, delay ping-pong.

---

# Setup 2 — HB CREATE (music creation, channel 15)

Where the HEARTBEAT setup is built for jamming a finished kit (mix, mutes, scenes, global
FX), **HB CREATE** is for building the music. It leans on all three learn modes: page 1
is a **visible-page** deck (mode 1 — 8 knobs that always control whatever the param area
shows), page 2 shapes the **selected track** (mode 2), and pages 3–4 carry **pinned
macros** (mode 3 — one knob per track, selection-independent). All controls on **MIDI
channel 15**, same CC rule. Files:

- [`docs/create-bindings-2/roto-heartbeat-create-bindings.json`](create-bindings-2/roto-heartbeat-create-bindings.json) —
  the 64 HeartBeat bindings (channel 15). Import alongside the performance bindings; no
  target overlaps, both stay live.
- [`docs/create-bindings-2/roto-heartbeat-create-setup.json`](create-bindings-2/roto-heartbeat-create-setup.json) — the
  ROTO-Setup side. Targets setup slot 63 (`index: 62`), name `HB CREATE`.

One EDIT page replaces v1's three mirrored param pages: its 8 knobs are `.pageSlot`
bindings, so they control **whatever the param area is showing** — flip pages from the
PG buttons (or the phone) and the same 8 knobs become that page. The app still follows
the hardware the other way too: turning a mode-2 knob pulls its page forward on screen.

## Page 1 — EDIT (follows the visible page)

| Control | Label | CC | Target | ROTO config |
|---|---|---|---|---|
| Knobs 1–8 | `KNOB 1`…`KNOB 8` | 1–8 | `.pageSlot(0–7)` | smooth — the visible page's controls, reading order |
| Button 1 | `PAGE <` | 9 | `.paramPagePrev` | PUSH — param carousel one page left (TRG at the far left) |
| Button 2 | `PAGE >` | 10 | `.paramPageNext` | PUSH — param carousel one page right |
| Button 3 | `Q REC` | 11 | `.quantizeRecord` | TOGGLE — record quantization, LED tracks state |
| Button 4 | `UNDO` | 12 | `.undo` | PUSH (LED lights while undo is available) |
| Button 5 | `REDO` | 13 | `.redo` | PUSH (pairs with UNDO beside it; PLAY lives on the co-loaded ch14 PERFORM setup) |
| Button 6 | `COPY` | 14 | `.copyStep` | PUSH — copy the held/latched step |
| Button 7 | `PASTE` | 15 | `.pasteStep` | PUSH — paste onto the held/latched step |
| Button 8 | `2X LEN` | 16 | `.doubleLength` | PUSH — double the loop, duplicating content |

What the 8 slots mean per visible page:

- **Sampler/synth param page**: its 2×4 grid (knob 1 = top-left … knob 8 = bottom-right).
- **MIDI machine SRC page**: knobs 1–4 = `CHANNEL BANK SUB PROG`, knobs 5–8 =
  `BEND MOD AT BREATH`; CC pages 1–3 = the visible 2×4 grid.
- **LFO page** (last page on every machine): knobs 1–4 = LFO-1's `WAVE RATE MULT DEPTH`,
  knobs 5–8 = LFO-2's. (The `PG` buttons only reach the first four pages — flip to the
  LFO page on the phone, or bind `.paramPage(4)`/`.trackLFO(0–7)` directly.)
- **TRG page, step latched**: `NOTE VEL GATE PROB NUDGE COND RETRIG RAMP`. PROB
  sweeps 1–100 %, COND steps through the condition list, RETRIG through off/2/3/4/6/8 hits.
- **TRG page, nothing latched**: all 8 knobs edit the track defaults — the same
  `NOTE VEL GATE PROB NUDGE COND RETRIG RAMP` layout.
- With a step latched and a knob page forward, slots write **locks** — exactly what
  touching the visible knob would do.

Feedback re-seats all eight motors on every page flip, track switch, and step latch.

## Page 2 — TRACK (selected track) + TRACK 1–8

| Control | Label | CC | Target | ROTO config |
|---|---|---|---|---|
| Knob 1 | `NOTE` | 17 | `.trackDefault(0)` | smooth — track default note |
| Knob 2 | `VEL` | 18 | `.trackDefault(1)` | smooth — track default velocity |
| Knob 3 | `GATE` | 19 | `.trackDefault(2)` | smooth — exponential, ⅛ step → 128 steps |
| Knob 4 | `LOOP LEN` | 20 | `.trackLength` | smooth — sweeps the length choices (1…128) |
| Knob 5 | `DLY SND` | 21 | `.samplerParam(27)` | smooth |
| Knob 6 | `REV SND` | 22 | `.samplerParam(28)` | smooth |
| Knob 7 | `CRUSH` | 23 | `.samplerParam(24)` | smooth |
| Knob 8 | `CHORUS` | 24 | `.samplerParam(30)` | smooth |
| Buttons 1–8 | `TRACK 1`…`TRACK 8` | 25–32 | `.trackSelect(0–7)` | PUSH, track colors, LED = selected |

## Page 3 — MACROS A (pinned, tracks 1–8) + TRACK 9–16

| Control | Label | CC | Target | ROTO config |
|---|---|---|---|---|
| Knobs 1–8 | `T1 MACRO`…`T8 MACRO` | 33–40 | `.pinnedKnob(track 0–7, slot 0)` | smooth, track colors |
| Buttons 1–8 | `TRACK 9`…`TRACK 16` | 41–48 | `.trackSelect(8–15)` | PUSH, track colors, LED = selected |

## Page 4 — MACROS B (pinned, tracks 9–16) + PTN 1–8

| Control | Label | CC | Target | ROTO config |
|---|---|---|---|---|
| Knobs 1–8 | `T9 MACRO`…`T16 MACRO` | 49–56 | `.pinnedKnob(track 8–15, slot 0)` | smooth, track colors |
| Buttons 1–8 | `PTN 1`…`PTN 8` | 57–64 | `.patternSlot(0–7)` | PUSH, LED = active pattern |

Notes:

- **The macro knobs never retarget**: `T‹n› MACRO` is track *n*'s knob slot 1 — CUTOFF by
  default on internal machines, CC knob 1 on MIDI tracks (re-point it per track in the
  app and the macro follows). Sixteen tracks' primary sound knob, two pages, no selection
  changes, no Control All spread, no record capture.
- **Track select is still the steering wheel** for pages 1–2: pick a track and the EDIT
  deck, the shaping knobs, and the on-screen param area all retarget. Feedback re-seats
  the motors to the new track's values.
- Defaults + `LOOP LEN` recompile the pattern on a trailing edge (150 ms after the sweep
  settles), so motorized sweeps don't flood the compiler.
- `PAGE </>` flip Perform-screen state the engine doesn't mirror, so their LEDs won't
  track app-side taps — treat them as one-shot pushes. (2026-07-24: they replaced the
  removed `.stepPage`/`.paramPage(i)` targets — TRG is now just the carousel's leftmost
  page; `Q REC`/`UNDO`/`PLAY` fill the freed buttons.)
- Dropped from v1: `MUTES/KEYS/SCENES` latches (mutes live on the HEARTBEAT setup's
  buttons) and `PPONG` (touchscreen). Importing the new bindings file cleanly replaces
  the v1 bindings — it reuses every channel-15 CC, and incoming bindings win.

## Motorized feedback (implemented)

HeartBeat mirrors the state of every learned CC binding out to a **Control Surface**
destination — pick `ROTO CONTROL` in Settings → Control. What it does:

- Any state change (knob edits, scene recall, morph, mutes, pattern switch, track
  selection, project load, undo) coalesces into one refresh ~30 ms later that sends only
  changed values.
- Selecting a track re-sends `.knob(0–7)` — the eight motors snap to the new track's sound.
- Inbound mapped CCs are recorded as "the surface already shows this", so the app never
  echoes a gesture back against the performer's hand, and a mis-routed loop converges
  instead of oscillating.
- Feedback sends bypass the outbound echo-suppression table on purpose (see
  `MIDIIO.sendFeedbackCC`).

Key code: `SequencerEngine` "Control-surface feedback" section (`feedbackValue(for:)`,
`performFeedbackRefresh`, `resendControlSurfaceState`), `MIDIIO.setFeedbackDestination` /
`sendFeedbackCC`.

## Rig setup

- **Power**: feed the ROTO's POWER USB-C from a PD supply; data USB-C to the iPhone/iPad.
  Bus power alone can't reliably drive the motors.
- **ROTO routing** (SYSTEM → MIDI ROUTING, fw 2.1+): USB IN → LOCAL ONLY; LOCAL MIDI → USB
  (unless gear hangs off the DIN outs).
- **HeartBeat**: MIDI Input default (`Both`) already hears the ROTO. Set Settings →
  Control → Control Surface to `ROTO CONTROL`.
- **Building the setup**: haptic steps, step names, detents, indents, and LED colors are
  only editable in the ROTO-Setup desktop app. Import `docs/perform-bindings/roto-heartbeat-setup.json`, or
  bind by hand via MIDI Learn (Perform screen ••• menu) — every target above already exists.
- HeartBeat is always clock master (24 PPQN while playing).

## Verify on hardware

Things the documentation couldn't fully confirm:

- [ ] Incoming CC moves motors over **USB** from iOS (Loopy Pro users confirm; a desktop
      Max user only got DIN working — if it fails, check MIDI ROUTING).
- [ ] Button **LEDs follow incoming CC** in TOGGLE mode (needed for mute-state feedback).
- [ ] Incoming CC for a control on a **non-visible page** updates the ROTO's stored value
      (if not: re-dump on page flip is impossible to detect — consider a periodic re-dump).
- [ ] HeartBeat TOG bindings + ROTO TOGGLE buttons stay in phase for mutes.
- [ ] Per-knob LCD value display updates on incoming MIDI in MIDI mode (fw 3.2).
- [ ] No CCs in 1–64 are ROTO-reserved in MIDI mode.
