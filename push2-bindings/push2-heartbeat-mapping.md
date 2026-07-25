# Ableton Push 2 → HeartBeat mapping

Research source: Ableton's public Push 2 MIDI & Display Interface manual
(github.com/Ableton/push-interface). Everything below is MIDI-mode only — the 960×160
display speaks a proprietary USB bulk protocol that iOS can't drive (no raw USB access),
so it stays on the Ableton splash. All the pads, buttons, and encoders work.

## Getting connected

1. Push 2 is USB class-compliant MIDI: camera adapter / USB-C hub into the iPhone
   (Push wants its power supply for full LED brightness).
2. Put Push in **User mode**: press the **User** button (top right). In User mode the
   hardware talks plain MIDI on the **User Port** ("Ableton Push 2 User Port") and
   stops reserving controls for Live.
3. Import `push2-heartbeat-bindings.json` in HeartBeat: Settings → MIDI Learn
   Bindings → ••• → Import (merge semantics, incoming wins).
4. For button/knob LED feedback, point Settings → Control → Control Surface at the
   Push User Port.

Everything is on **channel 1** (0-based 0), which is what the User Port emits.

## Encoders (relative)

Push encoders are endless and send two's-complement *deltas* — these bindings use the
`"relative": true` mapping flag (added for this surface): HeartBeat accumulates the
delta onto the target's current value.

| Control | CC | Target |
|---|---|---|
| Encoders 1–8 (above display) | 71–78 | `pageSlot(0–7)` — the visible param page, positionally (SRC grid, FLTR/AMP/FX, TRG cells, LFO page — whatever is showing) |
| Master encoder | 79 | `masterVolume` |
| Tempo / Swing encoders | 14 / 15 | unmapped (no BPM/swing sweep targets yet) |

With the standardized SRC grid, encoder 8 is always LEVEL, encoder 4 always SLOT, on
every machine and sample mode.

## Buttons

| Button | CC | Target |
|---|---|---|
| Play | 85 | `playStop` |
| Record | 86 | `record` |
| Tap Tempo | 3 | `tapTempo` |
| Undo | 119 | `undo` |
| Quantize | 116 | `quantizeRecord` (Q) |
| Double Loop | 117 | `doubleLength` (2×→) |
| Duplicate | 88 | `copyStep` |
| New | 87 | `pasteStep` |
| Repeat | 56 | `fill` — **hold to fill** |
| Mute | 60 | `muteMode` (track strip → mute board) |
| Note | 50 | `keysMode` (track strip → keyboard) |
| Session | 51 | `sceneMode` (track strip → scenes) |
| Page ◀ / Page ▶ | 62 / 63 | `paramPagePrev` / `paramPageNext` (TRG + knob pages) |
| Arrow ▶ | 45 | `pageNext` (next 16-step grid page) |
| Upper display row (8) | 102–109 | `patternSlot(0–7)` |
| Lower display row (8) | 20–27 | `scene(0–7)` |
| Right column top | 43 | `djFX(5)` — **hold to beat-repeat** |
| Right column 2nd | 42 | `djFX(6)` — **hold to tape-brake** |
| Right column 3rd | 41 | `muteScopeGlobal` — mute board in GLOBAL scope |
| Right column 4th | 40 | `muteScopePattern` — mute board in PTN scope |

Unmapped and free for later: Setup, Layout, Scale, Accent, Select, Shift, Delete,
Convert, Device/Browse/Mix/Clip, Add Track/Device, Master, Stop Clip, Metronome
(no learn target), arrows ◀▲▼, Octave ±, right-column buttons 3–8, Fixed Length,
Automate, Solo.

## Pads (8×8, notes 36–99)

Bottom-left pad = note 36, rising left→right then row by row.

Within each pair of rows the *upper* row is tracks 1–8 — matching HeartBeat's track
strip (top row 1–8, bottom row 9–16).

| Rows (bottom-up) | Notes | Function |
|---|---|---|
| Row 1 (bottom) | 36–43 | `trackSelect(9–16)` |
| Row 2 | 44–51 | `trackSelect(1–8)` |
| Row 3 | 52–59 | `trackMute(9–16)` |
| Row 4 | 60–67 | `trackMute(1–8)` |
| Rows 5–8 | 68–99 | **unmapped — playable** |

The top 32 pads deliberately carry no bindings: unmapped notes flow through
HeartBeat's performance-input path to the selected track, with velocity (and poly
aftertouch pass-through), so the upper half of the grid is a live keyboard. That also
means they **write onto a latched step** (chords included) via step-capture, and
record while REC rolls.

## Ignored outputs

Push emits **touch notes** the moment a finger lands on an encoder (notes 0–7 for the
display encoders, 8 master, 9 swing, 10 tempo) and on the touch strip (note 12).
Unmapped, those would fall through to the performance-thru path and play sub-bass
notes on the selected track — so the bindings map them all to the `ignore` target,
which consumes a control and does nothing. Use the same trick for any other noisy
surface output.

## Touch strip

Sends pitch bend — HeartBeat's thru path already re-channels bend to the selected
track while record is armed (internal machines bend TUNE ±2 semitones render-side).
No binding needed.

## Feedback / LEDs

HeartBeat's feedback emitter sends CC values, so **CC-bound buttons light**: Play,
Record, Q, the mode latches, patterns/scenes (value 127 = lit / 0 = dark; on RGB
buttons 127 lands in the palette, good enough to read state). Note-bound pads (the
select/mute rows) get no feedback yet — the emitter is CC-only. Lighting the pad grid
(selected track color, mute states, even a step sequencer view) would need note-based
feedback plus the Push RGB palette; doable later, as is driving LED animations
(channels 1–15 select transition curves).

## Known gaps / future

- Tempo/Swing encoders idle (no continuous BPM/swing targets).
- Pad LEDs dark (note feedback not implemented).
- The display can't be driven from iOS at all (USB bulk protocol, no DriverKit on
  iPhone) — same bridge-process caveat as the ROTO serial API.
