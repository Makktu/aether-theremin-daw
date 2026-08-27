# AETHER | Theremin DAW & Ambient Looper 🛸🎶

A completely browser-based, zero-dependency Digital Audio Workstation (DAW) dedicated to the ethereal sounds of the Theremin and ambient layered soundscapes. Built with plain HTML, CSS, and vanilla Web Audio JavaScript, AETHER requires no installation, no dependencies, and no servers.

---

## ✨ Features & Architecture

### 🖥️ Single-Screen Workspace
AETHER is laid out as a **fixed two-column workspace that fits entirely within the browser viewport at 100% zoom** — no page scrolling at any point.

- **Full-width top chrome:** header/transport, preset bar, and the synth tone toolbar (waveform, portamento, filter, vibrato, delay, master volume).
- **Left column — the instrument:** interface tabs (XY Pad / Keys / MIDI), the oscilloscope, the play surface, and the envelope shaper strip beneath it.
- **Right column — the recorder:** the 4-track looper as a vertical stack of compact track cards.
- **Elastic play surface:** the XY pad / keyboard is the flexible element, absorbing whatever height is left over. Track waveform displays grow with the viewport (20px at ~690px tall, ~100px at ~1010px).
- **Graceful degradation:** below a 690px-tall viewport the track column scrolls internally rather than clipping; below 1100px wide the workspace stacks vertically and normal page scrolling returns.

### 🎛️ Theremin Sound Engine
- **Continuous Portamento Emulation:** True logarithmic pitch gliding replicating a physical theremin antenna.
- **Natural Vibrato LFO:** Dedicated LFO modulating oscillator pitch with rate (1–12 Hz) and depth (cents) controls.
- **Warm Tone Biquad Filter:** Adjustable low-pass filter (200 Hz – 12 kHz) for shaping resonance and warmth.
- **Damped Tape Space Delay:** Stereo feedback delay circuit with analog high-frequency dampening.
- **Live Pitch HUD & Tuner:** Real-time note name (e.g. `A4`, `C#5`) and exact frequency display.
- **Live Audio Oscilloscope:** Real-time visual representation of output waveforms.

### 📈 ADSR Envelope Engine
A dedicated envelope stage (`envGain`) sits **after** the performance gain and **before** the filter, so the envelope *multiplies* the played level rather than replacing it. The XY pad's Y-axis remains a continuous theremin volume control, and the envelope shapes on top of it.

**Signal chain:** `osc → oscGain (performance level) → envGain (ADSR) → filterNode → dry + delay → master → analyser → destination`

- **Attack** (1 ms – 2 s): linear ramp to full level.
- **Decay** (5 ms – 4 s): exponential fall toward the sustain level. Implemented as `setTargetAtTime` with a time constant of `decay / 3`, so the stated time is when the curve is ~95% of the way there.
- **Sustain** (0–100%): held level while the note is down.
- **Release** (5 ms – 6 s): exponential fall to silence after note-off.
- **Filter Env** (−4 to +4 octaves): sweeps the Tone Filter cutoff relative to its slider position, sharing the amplitude envelope's A/D/S/R shape. Negative values sweep *downward* — useful for power-downs and failing systems.

**Two envelope modes:**

| Mode | Behaviour | Use for |
| --- | --- | --- |
| **Sustained** | A → D → S while held → R on release | Theremin performance, pads, drones |
| **One-Shot** | A → D to silence; hold and note-off are **ignored entirely** | Stingers, laser zaps, UI blips, impacts |

One-Shot is what makes percussive sound design possible: a single touch fires a complete shaped hit regardless of how long you hold.

**Implementation notes:**
- Retriggering uses `cancelAndHoldAtTime` (with a `cancelScheduledValues` + `setValueAtTime` fallback for browsers lacking it), so a new note ramps from the current value instead of jumping.
- `updateSynthParams()` checks whether a filter envelope is mid-flight before writing the cutoff, so moving the Tone Filter slider during a note no longer overwrites the envelope.
- `silenceVoice()` provides a hard stop regardless of mode, used by tab switching and Stop MIDI.

### 🛸 Curated Sound Preset Library
Includes expertly configured factory presets plus user custom preset creation:
- **Clara Rockmore (Concert):** Classic, pure operatic theremin tone.
- **1950s Sci-Fi UFO:** Classic retro wobble with deep cosmic echo.
- **Blade Runner Vangelis:** Rich, filtered sawtooth with wide synth portamento.
- **Ghostly Opera Voice:** Eerie, vocal formant resonance with ethereal delay wash.
- **Cosmic Ambient Wash:** Super-smooth glide with long atmospheric decay.
- **Vintage Tube Radio:** Warm, saturated vintage radio tone.
- **8-Bit Chiptune:** Crisp, stepped retro square wave lead.
- **Deep Sub Drone:** Low-frequency foundation drone.
- **Laser Zap** *(SFX)*: 2 ms one-shot attack with a +3.5 octave filter sweep (900 Hz → ~10.2 kHz).
- **UI Blip** *(SFX)*: Ultra-short one-shot square wave for interface feedback.
- **Airlock Swell** *(SFX)*: 1.4 s attack sustained swell with a +2.6 octave filter opening.
- **Custom Presets:** Save, load, export, and import custom presets to/from JSON files (persisted in `localStorage`). Preset files carry all six envelope parameters; presets saved before the envelope engine existed load with a full-sustain shape, reproducing their original sound exactly.

### 🔁 4-Track Synchronized Ambient Looper
- **4 Independent Audio Channels:** Record, overdub, layer, and arrange ambient soundscapes.
- **Master Cycle Synchronization:** Track 1 establishes master loop duration; subsequent tracks synchronize seamlessly.
- **Independent Channel Controls:**
  - Record / Overdub / Replace triggers
  - Play / Pause / Stop states
  - Mute (M) and Solo (S) buttons
  - Volume slider (0% to 150%)
  - Stereo Pan slider (L50 to R50)
  - ↺ Reverse playback toggle (reversed theremin textures)
  - Live waveform visualizer with synchronized playhead
- **BPM Clock & Metronome:** Integrated click track with Tap Tempo and tempo adjustments.
- **Stem & Mixdown Export:** Download individual track stems or an interleaved master mixdown as uncompressed 16-bit PCM `.WAV` files.

---

## 🚀 How to Use

1. Double-click `index.html` in any modern web browser.
2. Click **"Power On"** to start the Web Audio API context.
3. Select a preset from the top bar or sculpt your sound using the synthesizer parameters.
4. Play using the **Ethereal Pad (XY Mode)**, the **Synthesizer Keys**, or load a file in the **MIDI Player**.
5. Shape the sound with the **Envelope** strip beneath the play surface — switch to **One-Shot** mode for stingers and SFX.
6. Build multi-layered soundscapes using the **4-Track Ambient Looper** in the right-hand column.
7. Export your final master mixdown or individual stems as `.WAV` files!

---

## 📝 Changelog

### Layout reorganisation — single-screen two-column workspace
The previous vertical stack (toolbar → play area → looper → status) has been replaced with a two-column workspace, and the entire app is now locked to the viewport.

- **New `.workspace` grid:** `minmax(0, 1fr) 400px` — play section left, looper panel right. Header, preset bar and synth toolbar remain full width above.
- **Viewport lock:** `body` is `height: 100vh; overflow: hidden`; the container and workspace use flex/grid with `min-height: 0` so the play area and looper column absorb leftover height.
- **Top chrome compacted** to make room: reduced body padding and container gaps, smaller title/HUD/timer, tighter preset bar and synth toolbar padding, reduced slider margins and thumb size.
- **Track cards rebuilt** from seven stacked rows (~270px each) into four compact rows (~86px minimum): title + state pill + export/clear icons on one line; waveform; a single button row (Rec / Play / Mute / Solo / ↺); and Volume + Pan side by side with inline labels.
- **Even track distribution:** `grid-template-rows: repeat(4, minmax(86px, 1fr))` divides the column height evenly between the four cards, with the waveform display as the only flexing element. `overflow-y: auto` acts as a safety valve on very short viewports.
- **Looper header** stacks vertically in the narrow column; the four transport buttons became a 4-up grid, and "Export Mixdown .WAV" was shortened to "↓ Mixdown" (full text retained as the tooltip).
- **Responsive fallback:** below 1100px wide the workspace stacks and the track grid returns to its 4 / 2 / 1 column behaviour with normal page scrolling.

Verified in headless Chrome across viewport heights of 620–1100px: no page scroll from 693px upward.

### ADSR envelope engine
Added amplitude and filter envelopes — see the [ADSR Envelope Engine](#-adsr-envelope-engine) section above for full detail.

- New `envGain` node inserted between the performance gain and the filter; `oscGain` is now purely performance level while `envGain` handles all gating.
- New `gateOn()` / `gateOff()` / `silenceVoice()` functions replace the previous fixed `setTargetAtTime` fades in `triggerNoteOn` / `triggerNoteOff`. The XY pad fires `gateOn()` on touch-down, after pitch and level are set.
- New **Envelope** control strip in the left column (Mode, Attack, Decay, Sustain, Release, Filter Env), placed inside `.play-section` so it draws its height from the elastic play area and leaves the looper column untouched.
- Envelope parameters added to preset save / load / JSON export, with backwards-compatible defaults.
- Three new **SFX** built-in presets: Laser Zap, UI Blip, Airlock Swell.

Envelope scheduling was verified by swapping the audio globals onto an `OfflineAudioContext` and spying on the AudioParam automation calls, confirming: linear attack timing, correct sustain hold and release decay, that One-Shot mode issues zero amplitude automation on note-off, and correct filter sweep targets.
