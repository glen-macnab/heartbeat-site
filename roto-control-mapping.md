# HeartBeat × ROTO-Control — live-jam mapping

A mapping of HeartBeat onto the Melbourne Instruments ROTO-Control (8 motorized,
touch-sensitive knobs + 8 RGB buttons per page, 4 pages per MIDI setup). One ROTO setup,
all controls on **MIDI channel 14**, with one rule for every CC:

```
CC = 16 × (page − 1) + position        knobs = positions 1–8, buttons = 9–16
```

Page 1 is CC 1–16, page 2 is 17–32, page 3 is 33–48, page 4 is 49–64. CC 65–127 stay free
for a future second setup. HeartBeat's learn table keys on channel + number, so overlaps
with conventional CC meanings (1 = mod wheel, 7 = volume) are harmless on a dedicated channel.

A ready-to-import ROTO-Setup file for this mapping lives at
[`docs/roto-heartbeat-setup.json`](roto-heartbeat-setup.json) (import via ROTO-Setup →
File → Import). The full design rationale and research notes are in the project artifact;
this file is the working reference.

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

Left to the touchscreen (set-and-forget): reverb damp/predelay/width, ping-pong,
compressor detail, sidechain routing.

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
  only editable in the ROTO-Setup desktop app. Import `docs/roto-heartbeat-setup.json`, or
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
