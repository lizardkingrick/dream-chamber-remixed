# Roadmap

[← Back to the README](../README.md)

## Audio engine

What's left from the audio audit, roughly in order of what users would notice. Effort: S is under
an hour, M a few hours, L a day or more. What's done is at the bottom, and in the
[changelog](../CHANGELOG.md).

**Test first**

- [ ] **Safari (Mac, iPhone, iPad) and Firefox.** Everything so far was measured in Chromium only.
  Check that layers stay in sync on Safari (it now runs at the device's sample rate with a wider
  scheduling margin and loads the FLAC stems), the limiter (`js/limiter-worklet.js`) and its
  compressor fallback, WAV export through the limiter, Stutter / Tape stop / Wah / Phaser,
  recordings saving as `.m4a` on Safari, and on an iPhone: sound with the silent switch on and
  coming back after a call.
- [ ] **A real Android phone (a Galaxy S23 Ultra reported stutter when switching).** Phones now
  get two stretch workers, no background hq renders, a cheaper limiter and `latencyHint:
  'playback'`, all only tried in Chromium's phone emulation. Time a switch, and listen for
  dropouts while a pick renders.

**Loading and downloads**

- [ ] **First pick on Safari and phones (M).** At the device's 48 kHz the worker also converts the
  sample rate, so a first pick takes about 0.45 s against 0.3 s on a desktop. Splitting the
  conversion across both workers, or a 16-tap filter, would win back some of it.
- [ ] **Less data on iPhones and Macs (M).** WebKit loads the FLAC stems (about 4.5x the Ogg) to be
  exact to the sample. Once Safari's Ogg decoding is checked on real devices, the length and
  mark check in `exactDecode` could let it use the Ogg when it matches.

**Needs a UI change (talk to whoever owns the UI)**

- [ ] **Lossless recording (M).** A WAV option for Rec, taken from the mix with an AudioWorklet.
- [ ] **Look at the visuals with the steadier meters (S).** `meters()` no longer flickers and its
  bass follows the kick, so the kick flash is stronger and sustained layers sit 5-45% higher
  after the stage's own peak normalising (`normalise` in `js/stage-pyramid.js`). Only measured
  with the stage's maths, not watched on a real GPU: retune there if anything looks too hot.
- [ ] **Export extras (S-M).** A 24-bit option, and exporting each layer as its own file.

**Done**

- [x] Hosted site on older Safari: FLAC copies built by `tools/build-stems.mjs`, used when Ogg
  won't decode, and a plain message when nothing plays.
- [x] Safari sync: native sample rate, a wider scheduling margin where `baseLatency` is missing,
  late starts moved to the next beat, and decodes checked against `frames` / `marks` in
  `stems.json`.
- [x] Phones: fewer stretch workers, no background hq, a cheaper limiter, no long tasks during a
  switch.
- [x] Sound sooner on the first pick: a near-unity path in the worker (20-30 ms) instead of
  playing the original first; one worker starts on the first tap.
- [x] Prefetch that pays off: the hovered song, and the layers it would re-key or re-tempo, are
  stretched ahead.
- [x] The current mix's renders kept across reloads (IndexedDB, `KEEP.version`).
- [x] Less memory per stem: 16-bit originals, worker input with room at the end.
- [x] Loudness, frame counts and alignment marks in `stems.json`.
- [x] Phase vocoder for held chords and pads, swapped in on a bar (desktops only).
- [x] Natural voices when the key shifts (formant correction on sung leads).
- [x] Stutter that follows the tempo (Auto length) and auto tempo from the stems' average.
- [x] A tempo lock (auto tempo off) in the tempo box, remembered on the device.
- [x] Volume sliders with a taper (gain = position squared); share links keep storing the gain.
- [x] Steadier meters: layer levels over 46 ms, bass from the output's 40-120 Hz band.
- [x] When a pick will be heard: the card and the hub say "Downloading 40%", "Stretching", "Joins on
  the next bar" and so on until it plays (`pickStatus` / `tempoStatus` in `js/engine.js`), and the
  tempo box shows the tempo still heard while the mix moves to a new one.
- [x] Steadier sung leads: fast-path splices placed in steady vowels and silences with every
  syllable start anchored; the hq render keeps a lead's voiced parts in the vocoder.
