# AETHER | Theremin DAW & Ambient Looper 🛸🎶

A completely browser-based, zero-dependency Digital Audio Workstation (DAW) dedicated to the ethereal sounds of the Theremin and ambient layered soundscapes. Built with plain HTML, CSS, and vanilla Web Audio JavaScript, AETHER requires no installation, no dependencies, and no servers.

---

## ✨ Features & Architecture

### 🎛️ Theremin Sound Engine
- **Continuous Portamento Emulation:** True logarithmic pitch gliding replicating a physical theremin antenna.
- **Natural Vibrato LFO:** Dedicated LFO modulating oscillator pitch with rate (1–12 Hz) and depth (cents) controls.
- **Warm Tone Biquad Filter:** Adjustable low-pass filter (200 Hz – 12 kHz) for shaping resonance and warmth.
- **Damped Tape Space Delay:** Stereo feedback delay circuit with analog high-frequency dampening.
- **Live Pitch HUD & Tuner:** Real-time note name (e.g. `A4`, `C#5`) and exact frequency display.
- **Live Audio Oscilloscope:** Real-time visual representation of output waveforms.

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
- **Custom Presets:** Save, load, export, and import custom presets to/from JSON files (persisted in `localStorage`).

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
4. Play using either the **Ethereal Pad (XY Mode)** or the **Synthesizer Keys**.
5. Build multi-layered soundscapes using the **4-Track Ambient Looper** below the playing surface.
6. Export your final master mixdown or individual stems as `.WAV` files!
