# Elektron features & workflows worth stealing for HeartBeat

Research synthesis, 2026-07-07. Sources: Digitakt II User Manual (OS 1.15A), Tonverk User
Manual (OS 1.0.2), Syntakt User Manual (OS 1.30), Sound On Sound / MusicTech / Gearnews /
Synth Anatomy / Perfect Circuit reviews, Elektronauts threads (DT2 feature requests,
polyphonic sequencing, Tonverk CC behavior, Rytm scene/performance tips), and the OODA iOS
sequencer as a touch-adaptation case study.

## Where HeartBeat already stands

HeartBeat already covers most of the Digitakt II sequencer core: p-locks, FILL/PRE/1ST/A:B
trig conditions, probability, micro-timing, per-track length/speed, 4-note chords (DT2's
MIDI ceiling is also 4 notes with shared velocity/gate), 128 steps / 8 pages (DT2's exact
spec), queued pattern changes, mutes, and live CC recording with performer override. What's
missing is mostly the **performance and arrangement layer** on top.

## Tier 1 — high value, natural fits

1. **Retrigs / ratchets per step.** Fire a step's note N times at a subdivided rate, with
   optional velocity ramp. Core Elektron feature — but famously *absent on Digitakt MIDI
   tracks* and heavily requested on Elektronauts. HeartBeat can beat the hardware: the
   compiler just emits N note events per trig. OODA proves ratchets work well on touch.

2. **Page loop.** DT2 and Tonverk OS 1.3 let you restrict playback to a chosen page (or
   subset of pages) as an instant live jump/loop tool. Touch adaptation: long-press a page
   marker in the pager to loop that page. Reviewers single this out as a favorite
   performance feature.

3. **Temp save / reload (FUNC+YES / FUNC+NO).** Snapshot the pattern to temporary memory,
   tweak destructively, revert in one tap. Tonverk generalizes this into **Perform mode**:
   while active, all edits live in temp memory and the pattern restores on exit. Cheap to
   implement (snapshot the `Pattern` struct); removes the biggest fear in live tweaking.

4. **Pattern chains + song mode.** Elektron's volatile chains (lost on power-off) are a
   documented complaint; DT2/Digitakt 1.50/Tonverk added Song mode in response: up to 99
   rows, each row = pattern + repeat count + row length + per-row mutes (+ tempo). A
   saveable chain list is the biggest arrangement upgrade to the existing pattern queuing;
   a row-table song editor is very touch-friendly.

5. **Euclidean generator per track.** DT2/Syntakt model: *two* pulse generators (pulses
   spread evenly over the track length), combined with a Boolean op (OR/XOR/AND/SUB),
   per-generator rotation + whole-track rotation, generated trigs still p-lockable, and a
   "commit to real trigs" gesture. Non-destructive — manual trigs return when switched off.
   Sliders make this better on touch than on encoders; users also wish the parameters could
   be randomized/modulated, which is easy in software.

## Tier 2 — the performance layer

6. **Scenes & performance macros (Rytm/Octatrack).** A scene = a named snapshot of CC
   values (and optionally mutes) recallable in one tap; a performance macro = a slider
   scaling several CC offsets at once (pad pressure on hardware → a touch slider or XY pad).
   Octatrack's crossfader morphing between scenes = an on-screen crossfader interpolating CC
   sets. Rytm players expose macros as MIDI-CC destinations so hardware faders can drive
   them — HeartBeat's MIDI-learn already fits that. The marquee "live" feature.

7. **Control All.** One knob gesture applied across all (or all-but-excluded) tracks.
   MIDI-only adaptation: apply the relative change to the same knob slot on every track.
   Elektron doesn't offer Control All on MIDI tracks — another chance to out-do the hardware.

8. **Kit lock across pattern changes.** DT2's Kit Perform mode (and Digitakt's persistent
   track volumes, added by demand in OS 1.30): switching patterns doesn't yank sound
   parameters. For HeartBeat: an option to carry current knob/CC state across a pattern
   switch instead of loading the new pattern's stored values.

9. **Mute scopes + transition modes.** DT2 distinguishes **global mutes** (project-wide,
   survive pattern changes — what HeartBeat has) from **pattern mutes** (saved per pattern).
   Tonverk adds pattern *transition modes*: sequential / direct jump / direct start /
   temp-jump-then-return.

## Tier 3 — note entry & musicality

10. **Scale-aware keyboard + per-pattern scale/root.** Tonverk ships 36 scales; DT2 users
    request per-pattern scale storage and a "quantize all trigs to scale" action. The piano
    roll could dim/hide out-of-scale keys — a better fit on touch than on 16 trig keys.

11. **Arpeggiator per track.** Tonverk has one on every MIDI track (rate/octave/direction).
    Pairs well with 4-note chord steps — an arp toggle that plays the chord as a run.

12. **LFOs on CC.** DT2 gives MIDI tracks 2 LFOs transmitted as CC — generative motion
    without recording automation. One LFO assignable per knob (shape/rate/depth,
    tempo-synced) would complement recorded lanes; rate-limit the CC output.

13. **Instrument definitions / CC learn.** Top Elektronauts wishlist items: named CCs
    (HeartBeat has this), *reusable device templates* so CC maps aren't re-entered per
    track, and learning a CC number by wiggling the knob on the external synth.

## Tier 4 — smaller gaps

14. **Missing conditions:** NEI/¬NEI (depends on the neighbor track's last condition
    result) and LST/¬LST (fires on the final pass before a queued pattern change —
    HeartBeat already knows a switch is queued, so this is cheap and great for transitions).
15. **Faster per-track speeds.** DT2 tops out at 2× and users complain; add 4×.
16. **Program change per pattern.** Send patch/bank changes to external gear on pattern
    switch — standard Elektron MIDI behavior, very relevant to a MIDI-only app.
17. **Ghost trigs.** Rytm technique: steps at zero velocity that a macro's velocity offset
    "un-hides" for builds.

## Design caution

MusicTech's main criticism of DT2 is that the feature pile-up created menu-diving
convolution. The consistent praise for touch sequencers (OODA, Loopy Pro forum) is that
they win precisely when features stay one gesture away — so Tier 1–2 items should land as
visible performance surfaces (page-loop on the pager, scenes as a strip, temp-reload as one
button) rather than settings screens.

## Suggested next build

**Retrigs, page loop, and temp save/reload** — highest performance payoff for the least
architecture, and all three slot into existing UI.

## Reference notes (spec details worth keeping)

- DT2: 16 tracks (each audio *or* MIDI), 128 steps / 8 pages, per-track length + speed
  (1/8×–2×), chance + fill + condition stack independently per step, 16 assignable CCs +
  pitch bend/aftertouch + 2 LFOs per MIDI track, CC numbers learnable from incoming MIDI,
  Euclidean per track, Song mode, Control All with per-track exclusion, Kit Perform mode.
- Digitakt (orig) Song mode: 16 songs/project, ≤99 rows; per-row pattern, repeat 1–32, row
  length 2–1024 steps, tempo, swing, mutes; songs can be generated from a chain.
- Tonverk: 256 steps / 16 pages, tracks 1–12 can run a MIDI machine, 16-note polyphony per
  step, 16 assignable CCs, chord mode + keyboard mode (36 scales) + per-track arp, Perform
  mode (temp-memory edits, restore on exit), 4 transition modes, pattern mute (OS 1.2),
  page looping (OS 1.3). Encoder CC-out not implemented at launch ("MIDI implementation
  isn't finished" per Elektronauts).
- Syntakt MIDI tracks: chords as 3 offsets-from-root (transpose with root), CCs default OFF
  until enabled, no retrig/Control All on MIDI tracks.
- FILL activation grammar (all boxes): armed-for-one-cycle, momentary hold, latched.
- DT2 chord limit: 4 notes/step, shared velocity + gate; Digitone II raises this to 16 with
  per-note velocity and splay. Workarounds: second track on same channel; micro-timed
  neighbor steps.
