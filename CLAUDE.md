# AETHER — working notes

Browser-based theremin synth + 4-track ambient looper. The long-term aim is a
tool for producing sci-fi themes, stingers and SFX for a game the owner is
developing — favour sound-design capability and export workflow over
general-purpose DAW features.

## Shape of the project

Three files, no build step, no dependencies, no server. Open `index.html`
directly in a browser.

| File | Contents |
| --- | --- |
| `index.html` | Everything: CSS in `<style>` (lines ~7–1048), markup, then the app in one `<script>` (lines ~1510–3056). |
| `midi.js` | Vendored `MidiPlayer` library. Third-party — don't edit. |
| `README.md` | User-facing feature documentation and changelog. |

The script is sectioned with banner comments — search for `// 1.` … `// 5.`:

1. Sound preset library system
2. Audio synthesizer & effects graph
3. 4-track synchronized ambient looper engine
4. Stem & mixdown WAV export system
5. Main recorder & UI event listeners

Keep it that way: vanilla, single-file, zero-dependency. Don't introduce a
bundler, a framework, or npm packages.

## Audio graph

```
osc → oscGain → envGain → filterNode ─┬─ dry ────────────────→ masterGain → analyser → destination
                                      └─ delayNode → delayFilterNode → feedbackNode ─┘
vibratoOsc → vibratoGain → osc.frequency
looper tracks → gainNode → pannerNode → looperMasterGain → masterGain
```

Node roles, which are easy to get wrong:

- **`oscGain`** — *performance* level only. XY pad Y-axis, or 1.0 for keys/MIDI.
  Rests at 1, never used for gating.
- **`envGain`** — the ADSR. Does *all* note gating; rests at `MIN_GAIN`
  (0.0001, not 0, so exponential ramps stay legal).
- Because the envelope sits after the performance gain, it **multiplies** the
  played level. That's what lets the XY pad keep its continuous theremin volume
  while still having an envelope. Don't collapse these two stages.

Note lifecycle: `gateOn()` / `gateOff()` / `silenceVoice()`. The XY pad calls
`gateOn()` from `startPad` *after* `handlePadMove` has set pitch and level.
Keys and MIDI go through `triggerNoteOn` / `triggerNoteOff`.

In **One-Shot** mode `gateOff()` is a deliberate no-op — the sound decays to
silence on its own. `silenceVoice()` is the mode-independent hard stop, used by
tab switching and Stop MIDI.

## The layout is on a tight height budget

The app is locked to `100vh` with `overflow: hidden` and must not scroll at
100% zoom. The height budget is genuinely tight — at a 693px viewport it fits
with only a few pixels of slack.

**Adding a control to the full-width synth toolbar will break the fit.** A
second toolbar row costs ~43px, which comes out of the workspace, which shrinks
*both* columns, which pushes the track cards below their 86px minimum.

The right place for new synth controls is **inside `.play-section`**, alongside
the existing `.shaper-bar`. The play area is the elastic element with plenty of
slack, so a strip there costs the looper column nothing. This is exactly why
the envelope strip lives where it does — the same pattern should be used for
noise, ring mod, reverb and so on. If enough strips accumulate, make them a
tabbed page rather than stacking them.

Other load-bearing details:

- `.workspace` is `grid-template-columns: minmax(0, 1fr) 400px`.
- `.looper-tracks-grid` uses `grid-template-rows: repeat(4, minmax(86px, 1fr))`
  so the four cards divide the column height evenly. The track waveform is the
  only flexing element in a card, so it absorbs the slack.
- Everything in the chain from `body` down needs `min-height: 0` or the flex
  children refuse to shrink.
- Below 1100px wide the workspace stacks and normal page scrolling returns.

**Verify layout changes** by measuring, not by eye. Render with headless Chrome
at several viewport heights and compare `scrollHeight` against
`getBoundingClientRect().height` on `.looper-tracks-grid`, `#track-card-0` and
`document.documentElement`:

```
"/Applications/Google Chrome.app/Contents/MacOS/Google Chrome" \
  --headless=new --disable-gpu --window-size=1440,780 \
  --virtual-time-budget=4000 --dump-dom "file://<copy>.html"
```

Hide `#power-overlay` in the test copy. Check 620 / 700 / 780 / 900 — the app
should fit with no page scroll from a ~693px viewport upward, and the track
column should scroll internally (not clip) below that.

## Testing audio

`--virtual-time-budget` virtualises `setTimeout` but **not** `ctx.currentTime`,
so sampling AudioParam values over wall-clock in headless Chrome reads
everything at t≈0. It doesn't work.

Instead, swap the module-level globals onto an `OfflineAudioContext` and call
the real functions against a fake clock. `gateOn` / `gateOff` / `silenceVoice`
only touch `ctx.currentTime`, `envGain.gain`, `filterNode.frequency` and the
DOM, so a plain `{ currentTime: t }` object works as `ctx`:

```js
const off = new OfflineAudioContext(1, 44100 * 3, 44100);
const src = off.createConstantSource(); src.offset.value = 1;
const saved = [ctx, envGain, filterNode];
ctx = { currentTime: 0 };
envGain = off.createGain(); filterNode = off.createBiquadFilter();
src.connect(envGain); envGain.connect(off.destination);
// spy on envGain.gain / filterNode.frequency methods, then:
gateOn(); ctx.currentTime = 1.0; gateOff();
src.start(0); const buf = await off.startRendering();
```

Rendering a ConstantSource through `envGain` gives you the envelope curve as
raw samples. Spying on the AudioParam methods verifies the scheduling. Restore
the globals afterwards.

Add `--autoplay-policy=no-user-gesture-required` so `powerOn()` works headless,
and write results into a DOM node incrementally — if the page throws, you still
see how far it got. Build the output marker at runtime
(`String.fromCharCode(...)`), otherwise `--dump-dom` also returns the literal
from the script source and your grep matches the wrong thing.

## Presets

`BUILTIN_PRESETS` is a flat array of plain objects; user presets live in
`localStorage` and are import/exportable as JSON.

When adding a synth parameter, touch all three of:

- `loadSelectedPreset()` — read it with a `!== undefined` default that
  **reproduces the previous behaviour**, so presets saved before the parameter
  existed still sound the same.
- `saveCurrentPreset()` — write it.
- The `BUILTIN_PRESETS` entries, if the factory presets should use it.

The first five built-ins are also rendered as quick-access chips, so adding
presets at the end of the array won't disturb the chip row.

## Known rough edges

- Track waveforms are only drawn on record (`renderTrackWaveform`), so they
  stretch until the next take if the window is resized. Pre-existing.
- The filter envelope shares the amplitude envelope's A/D/S/R timing and only
  has its own depth. Worth splitting if it starts getting in the way.
- There's no resonance (Q) control — `filterNode.Q` sits at its default. Adding
  one is a couple of lines and makes the filter envelope far more useful.

## Conventions

- Match the surrounding style: 2-space indent, `camelCase`, `onclick=` handlers
  in the markup, CSS custom properties from `:root` for all colours.
- Don't add build tooling, and don't split the file up.
- README carries the user-facing changelog — update it when features land.
