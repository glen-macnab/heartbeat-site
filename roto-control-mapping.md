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
| Button 5 | `SNAP` | 13 | `.knobSnapshot` | PUSH — **hold, tweak, release to restore**, LED teal |
| Button 6 | `CTRL ALL` | 14 | `.controlAll` | TOGGLE, LED violet |
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

## Page 4 — FX + SCENES

| Control | Label | CC | Target | ROTO config |
|---|---|---|---|---|
| Knob 1 | `DLY TIME` | 49 | `.globalFX(0)` | **stepped ×10**, step names `1/32 1/16 1/8T 1/16. 1/8 1/4T 1/8. 1/4 1/4. 1/2` |
| Knob 2 | `DLY FDBK` | 50 | `.globalFX(1)` | smooth |
| Knob 3 | `DLY TONE` | 51 | `.globalFX(3)` | smooth |
| Knob 4 | `DLY RETURN` | 52 | `.delayReturn` | smooth |
| Knob 5 | `REV SIZE` | 53 | `.globalFX(4)` | smooth |
| Knob 6 | `REV RETURN` | 54 | `.reverbReturn` | smooth |
| Knob 7 | `MORPH A>B` | 55 | `.sceneFader` | center detent |
| Knob 8 | `MASTER` | 56 | `.masterVolume` | indent at 100 (unity) |
| Buttons 1–8 | `SCENE 1–8` | 57–64 | `.scene(0–7)` | PUSH, one LED color per scene (LED lights when the slot holds a scene) |

Left to the touchscreen (set-and-forget): reverb damp/predelay/width,
compressor detail, sidechain routing. (Ping-pong lives on CREATE page 4.)

---

# Setup 2 — HB CREATE (music creation, channel 15)

Where the HEARTBEAT setup is built for jamming a finished kit (mix, mutes, scenes, global
FX), **HB CREATE** is for building the music: full sound design on the selected track,
track/pattern navigation, and the sequencer's editing tools. All controls on **MIDI
channel 15**, same CC rule. Files:

- [`docs/create-bindings/roto-heartbeat-create-bindings.json`](create-bindings/roto-heartbeat-create-bindings.json) —
  the 64 HeartBeat bindings (channel 15). Import alongside the performance bindings; no
  target overlaps, both stay live.
- [`docs/create-bindings/roto-heartbeat-create-setup.json`](create-bindings/roto-heartbeat-create-setup.json) — the
  ROTO-Setup side. Targets setup slot 63 (`index: 62`), name `HB CREATE`.

The knob pages mirror HeartBeat's own param pages one-to-one (`.samplerParam` ids, which
are machine-agnostic), and the app follows the hardware: **turning any mapped knob selects
the page that hosts it on screen** — turn a FLTR knob and the FLTR page comes forward,
turn a defaults knob and the TRG defaults panel opens. Machines relabel a few knobs
(sampler TUNE = synth DETUNE etc.); the printed labels use the sampler's names.

## Page 1 — SRC (selected track)

| Control | Label | CC | Target | ROTO config |
|---|---|---|---|---|
| Knob 1 | `TUNE` | 1 | `.samplerParam(0)` | center detent |
| Knob 2 | `FINE` | 2 | `.samplerParam(1)` | center detent |
| Knob 3 | `MODE` | 3 | `.samplerParam(2)` | stepped ×4 `FWD REV FWD-LOOP REV-LOOP` |
| Knob 4 | `SLOT` | 4 | `.samplerParam(3)` | smooth |
| Knob 5 | `START` | 5 | `.samplerParam(4)` | smooth |
| Knob 6 | `LENGTH` | 6 | `.samplerParam(5)` | smooth |
| Knob 7 | `LOOP` | 7 | `.samplerParam(6)` | smooth |
| Knob 8 | `LEVEL` | 8 | `.samplerParam(7)` | smooth |
| Buttons 1–8 | `TRACK 1`…`TRACK 8` | 9–16 | `.trackSelect(0–7)` | PUSH, track colors, LED = selected |

## Page 2 — FLTR (selected track)

| Control | Label | CC | Target | ROTO config |
|---|---|---|---|---|
| Knob 1 | `TYPE` | 17 | `.samplerParam(8)` | stepped ×6 `OFF LP2 LP4 BP HP NOTCH` |
| Knob 2 | `CUTOFF` | 18 | `.samplerParam(9)` | smooth |
| Knob 3 | `RESO` | 19 | `.samplerParam(10)` | smooth |
| Knob 4 | `ENV` | 20 | `.samplerParam(11)` | center detent (bipolar depth) |
| Knobs 5–8 | `ATK DEC SUS REL` | 21–24 | `.samplerParam(12–15)` | smooth |
| Buttons 1–8 | `TRACK 9`…`TRACK 16` | 25–32 | `.trackSelect(8–15)` | PUSH, track colors, LED = selected |

## Page 3 — AMP (selected track)

| Control | Label | CC | Target | ROTO config |
|---|---|---|---|---|
| Knob 1 | `VOL` | 33 | `.samplerParam(22)` | indent at 100 (unity) |
| Knob 2 | `VELO` | 34 | `.samplerParam(23)` | smooth |
| Knob 3 | `PAN` | 35 | `.samplerParam(21)` | center detent |
| Knob 4 | `DRIVE` | 36 | `.samplerParam(26)` | smooth |
| Knobs 5–8 | `ATK DEC SUS REL` | 37–40 | `.samplerParam(16, 18, 19, 20)` | smooth |
| Buttons 1–8 | `PTN 1`…`PTN 8` | 41–48 | `.patternSlot(0–7)` | PUSH, LED = active pattern |

## Page 4 — SEQ + FX (selected track)

| Control | Label | CC | Target | ROTO config |
|---|---|---|---|---|
| Knob 1 | `NOTE` | 49 | `.trackDefault(0)` | smooth — track default note |
| Knob 2 | `VEL` | 50 | `.trackDefault(1)` | smooth — track default velocity |
| Knob 3 | `GATE` | 51 | `.trackDefault(2)` | smooth — exponential, ⅛ step → 128 steps |
| Knob 4 | `LOOP LEN` | 52 | `.trackLength` | smooth — sweeps the length choices (1…128) |
| Knob 5 | `DLY SND` | 53 | `.samplerParam(27)` | smooth |
| Knob 6 | `REV SND` | 54 | `.samplerParam(28)` | smooth |
| Knob 7 | `CRUSH` | 55 | `.samplerParam(24)` | smooth |
| Knob 8 | `CHORUS` | 56 | `.samplerParam(30)` | smooth |
| Button 1 | `TRG` | 57 | `.stepPage` | PUSH — toggle the step editor / defaults panel |
| Button 2 | `2X LEN` | 58 | `.doubleLength` | PUSH — double the loop, duplicating content |
| Button 3 | `COPY` | 59 | `.copyStep` | PUSH — copy the held/latched step |
| Button 4 | `PASTE` | 60 | `.pasteStep` | PUSH — paste onto the held/latched step |
| Button 5 | `MUTES` | 61 | `.muteMode` | PUSH — track strip becomes the mute board |
| Button 6 | `KEYS` | 62 | `.keysMode` | PUSH — track strip becomes the keyboard |
| Button 7 | `SCENES` | 63 | `.sceneMode` | PUSH — track strip becomes the scene pads |
| Button 8 | `PPONG` | 64 | `.globalFX(2)` | PUSH — delay ping-pong toggle |

Notes:

- **Track select is the CREATE setup's steering wheel**: pick a track on page 1/2, and
  every sound knob (pages 1–4) and the on-screen param area retarget to it. Feedback
  re-seats all motors to the new track's values.
- Defaults + `LOOP LEN` recompile the pattern on a trailing edge (150 ms after the sweep
  settles), so motorized sweeps don't flood the compiler.
- `TRG`, `MUTES`, `KEYS`, `SCENES` flip Perform-screen state the engine doesn't mirror, so
  their LEDs won't track app-side taps — treat them as one-shot pushes.
- MIDI-machine tracks: pages 1–3 knobs are internal-machine params and do nothing; page 4
  defaults, `LOOP LEN`, and all buttons work on every track type.

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
