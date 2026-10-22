# Track Patterns

Version: `0.1.0-experimental` · author: [@pgstath](https://github.com/pgstath) · source: TrackPatterns `v0.1.0` (`a3a7e32`), the same code as [sambanks/octabam#846](https://github.com/sambanks/octabam/pull/846)

Each of the 16 tracks (T1–T8, M1–M8) can play its own pattern of the
playing bank and change it on its own, clip style. A change lands at the
end of that track's own loop, so it stays in time. The Part's machines,
samples and FX stay as they are. ColdFire-only (no DSP code, no effect
slot), for original OS 1.40C.

**Experimental.**
- Played on one MKII in octabam builds of this source, 21–22 Oct 2026.
- Not yet played as an image built by Modwerk's builder, and no USB (base)
  package yet.
- A silent hang seen on two builds is still open (below).

See [TESTING.md](TESTING.md).

## Overview

On a stock Octatrack every track plays the pattern the sequencer plays.
Track Patterns adds a pattern per track: hold a TRACK key, hold PATTERN
and press a trig key.
- **When it lands:** that track switches to the pattern at the end of its
  own loop. While the sequencer is stopped, the change applies at once and
  PLAY starts it from step 1.
- **Everything else carries on:** the global pattern, the Part and every
  other track.
- **Back to the global pattern:** the same gesture with the playing
  pattern's trig brings one track back. Any global pattern change, a chain
  step or a bank change brings every track back.

**What comes from where:**

| from the track's own pattern | stays the playing pattern's |
|---|---|
| trigs, trig conditions, micro-timing, parameter locks | tempo, the Part's machines, samples and FX, scenes |
| the track's length, under SCALE MODE PER TRACK | MASTER LENGTH, scale mode, chains, the track's speed (scale) |

**The trig LEDs**, for the current track when it plays its own pattern:
- Outside GRID REC (TRACKS trig mode), where stock shows only the running
  light, that pattern's trigs blink slowly, from the 16 steps its playhead
  is in.
- In GRID REC, the stock trigs (the pattern the keys edit) blink slowly, as
  a warning that this track does not play them.
- The running light stays in both.
- With TRACK n + PATTERN held, the pattern track n plays lights as the
  current one, and a queued change as a queued one.

**How it works.** Where the sequencer hands a track's step to the step
handler, the pattern argument is replaced by the track's own pattern.
Where the per-track advance compares the track's counter with its length,
the length comes from that pattern. A track with no change set gets
exactly what stock passes.

There is no clock of its own: the module follows the sequencer's own
steps, lengths, transport, tempo and swing.

## Controls

None: no effect slot, no knob, no menu row. The only input is the key
gesture:

| Action | Keys |
|---|---|
| Play pattern k on track n | Hold [TRACK n], hold [PATTERN], press [TRIG k] |
| Track n back to the global pattern | The same, with the playing pattern's trig |
| Every track back | Any global pattern change: PATTERN + TRIG without a TRACK key, a chain, a bank change |

In MIDI mode the TRACK keys address M1–M8. Press [TRACK n] first, then
[PATTERN]. The popup names the change: `T1 ▸ A03`, or `T1 FOLLOWS`.

## Usage

- **Clip-style playing:** keep a bank of variations (A01 the groove, A02 a
  fill, A03 a breakdown). While A01 plays, put the bass on A02 and the
  drums on A03, track by track. Each switches at its own loop end.
- **Lengths:** under SCALE MODE PER TRACK, a track on another pattern loops
  at its own length there, inside the playing pattern's MASTER LENGTH,
  which still restarts every track. Set MASTER LENGTH to INF to let longer
  patterns run in full. Under NORMAL every track wraps at the playing
  pattern's length.
- **Editing and recording** (GRID REC, LIVE REC, the pattern editor) still
  address the playing pattern. Look at the trig LEDs to see which pattern a
  track plays.
- **Overrides are not saved.** After a reload or a power cycle every track
  follows the global pattern again.

## Quick tutorial

1. Include Track Patterns in your build. Load a project whose bank has A01 and A03 with different trigs on track 1, select A01 and press PLAY.
2. Hold TRACK 1, hold PATTERN and press TRIG 3: the popup reads T1 ▸ A03, and at the end of its loop track 1 plays A03's trigs while every other track stays on A01. With track 1 current, its trig keys blink A03's trigs.
3. Hold TRACK 1, hold PATTERN and press TRIG 1: the popup reads T1 FOLLOWS and track 1 returns to A01 at the end of its loop. A global pattern change (PATTERN + TRIG without a TRACK key) brings every track back.

## Compatibility and limitations

**Base.**
- Original OS 1.40C.
- Played on an MKII. The gesture and LED runs under the ColdFire port
  behaved the same as an MKI and an MKII.

**Beside other modules.**
- Patches no byte another catalogue module or the base patches.
- Play Modes' step-load and rebuild detours (`0x400a2d6a`, `0x400a39b6`,
  `0x4009dc86`, `0x4009e3dc`) end where this module's begin. The two share
  no byte and compose by design.
- Under the ColdFire port, an image with Play Modes, Euclid, Tape Echo,
  Mini Verb, E-Verb and USB audio gave the same gesture events and trig LEDs
  as this module alone.
- On a unit it ran beside octabam's KITS and DIRECT JUMP (Kyoti), which are
  not in this catalogue.

**Deliberate change to a stock flow: the gesture.**
- What changes: with a TRACK key held, PATTERN + TRIG changes that track's
  pattern instead of the global pattern. Who sees it: anyone who holds a
  TRACK key while changing patterns. PATTERN + TRIG without a TRACK key
  stays stock.
- Why this gesture: it is the closest one to stock's pattern change. In
  stock 1.40C the held TRACK key is ignored, so this combination does
  nothing more than PATTERN + TRIG.
- Considered instead: an own page, and other key combinations. Each would
  add a step, or take a combination stock or another module uses.
- Turning it off: leave the module out of the build.
- Checked beside it: stock PATTERN + TRIG, a tapped (latched) PATTERN,
  TRACK pressed after PATTERN, chains, GRID REC editing, and save and
  reload.

**Deliberate change to a stock flow: the trig LEDs.**
- What changes: for the current track on its own pattern, the trig keys
  blink, as described in Overview. With every track following the global
  pattern, the LEDs are stock.
- Other trig modes: CHROMATIC (checked on a unit and under the port) and
  the other modes (on a unit) show stock's display only.

**Limits of v1.**
- Editing and recording address the playing pattern.
- Overrides are runtime state, not saved with the project.
- A track's speed (scale) stays the playing pattern's.

**Open.**
- A silent hang after 5–20 minutes on the main playback screen with an
  override set was seen on two octabam builds of this source (`OCTABAM2`,
  `OCTABAM3`). It is not yet explained.
- Track speeds other than 1x, swing, a long run, and MIDI note-offs across
  a switch with a long note are untested.
- No image built by Modwerk's builder has been played on a unit, and the
  USB (base) package is not built yet.

**Memory.** Like every DRAM module, the build gives up about 10 MB of
sample memory to the platform reserve.

## Tests and measurements

**The gate.** `python3 verify.py` compiles the engine and the firmware glue
for the host and runs their suites:
- a change landing at the track's own loop start, for lengths 1–64;
- MASTER LENGTH's repeated step, and PLAY restarts;
- the way back to following, and global and bank changes clearing;
- the look-ahead, MIDI tracks and the popup text;
- the gesture and the trig LEDs, including the page that follows the
  playhead.

Planted defects fail it. With `m68k-elf-gcc` on the PATH, it also checks
that `trackpatterns.s` matches the C.

**Cost**, under the emulator (`ot_emu --mkii`): instructions per call,
median / max, counting the stock instructions each detour replays.

| detour | median / max |
|---|---|
| audio tick | 86 / 95 |
| MIDI tick | 89 / 98 |
| audio advance | 60 / 186 |
| MIDI advance | 60 / 84 |
| the LED pass's own part | 52 / 658 |

- A 16-track step is about 2,360 instructions, 3,700 at most.
- Memory: 3,457 B of code and data, 84 B of it state. No `.bss`, heap,
  battery RAM or DSP words. The stack is about 72 B.
- Chip cycles per detour are not measured.

**On an MKII with CF METER** (300 BPM, a trig on every step of 15 tracks):
- With every track following, idle time and the frame interrupt match the
  build without the module, within the noise.
- With eight tracks on their own patterns, the LED pass and gestures, idle
  is 1.5 points lower (about 5.4 µs of each 362.8 µs frame), and the
  interrupt's mean is unchanged.

Every run and its result: [TESTING.md](TESTING.md). How every firmware
address was found: [INVESTIGATION.md](INVESTIGATION.md).

## Authorship and licences

- **Code:** original code by pgstath, MIT ([LICENSE](LICENSE)).
- **Sequencer addresses and findings:** they build on devilfish707's Play
  Modes investigation (Octaplay, MIT; addresses and findings only, no
  code). That investigation cites octabam (Sam Banks), SCALE QUANTIZER and
  DIRECT JUMP (Tim Hastie), the KYOTI notes (Zac-Kyoti) and EUCLID.
- **LED model:** builds on octabam's `docs/firmware/PANEL.md`.
- **Everything else:** read from the author's own 1.40C. No Elektron code,
  tables or images are included.

## Screens and audio

The real black-and-white LCD captures are still to be made, in the
headless emulator from this module's Modwerk build: the main screen with
the `T1 ▸ A03` and `T1 FOLLOWS` popups. The module has no sound of its own
and no audio preview.
