# AETHER | Theremin DAW 🛸🎶

A completely browser-based, single-file Digital Audio Workstation (DAW) dedicated exclusively to the ethereal sounds of the Theremin. Built with plain HTML, CSS, and vanilla JavaScript, AETHER requires no installation, no dependencies, and no servers.

## ✨ Features

- **True Theremin Emulation:** Replicates the continuous pitch and volume gliding (portamento) characteristic of a physical Theremin.
- **Dual Play Interfaces:** \* **Ethereal Pad:** A 2D XY pad that utilizes mouse or touch tracking to control pitch and volume simultaneously.
  - **Synthesizer Keys:** A traditional piano-roll interface for discrete note triggering.
- **Built-in Effects:** Real-time control over waveform types, glide speed (portamento), and spatial delay (feedback loop).
- **Native `.WAV` Recording:** Captures raw, uncompressed 16-bit PCM audio data directly from the audio graph and exports a pristine `.wav` file locally.
- **Modern Dark UI:** A sleek, distraction-free interface designed for low-light studio environments.

---

## 🚀 How to Use

Because AETHER is a zero-dependency web app, getting started takes seconds:

1. Create a new text file and name it `index.html`.
2. Paste the AETHER source code into the file and save it.
3. Double-click `index.html` to open it in any modern web browser (Chrome, Firefox, Safari, Edge).
4. Click the **"Power On"** button to initialize the browser's audio engine.

---

## 🎛️ Controls & Interface

### The Playing Surfaces

- **Ethereal Pad (XY Mode):** Click and drag (or touch and drag) inside the grid.
  - **X-Axis (Left/Right):** Controls the pitch (frequency). Moving right increases the pitch logarithmically.
  - **Y-Axis (Up/Down):** Controls the volume (amplitude). Moving up increases the volume.
- **Synthesizer Keys:** Click or tap individual keys to play specific notes. The glide setting will still affect how the notes transition into one another.

### The Toolbar

- **Waveform:** Choose the core sound of the oscillator.
  - _Sine:_ The classic, pure Theremin tone.
  - _Triangle:_ Slightly brighter and warmer.
  - _Sawtooth:_ Harsh, buzzing, and highly synthetic.
- **Portamento (Glide):** Controls how long it takes for the audio engine to slide from one pitch/volume to the next. Higher values create a "lazier," more authentic Theremin swoop.
- **Space (Delay):** Controls the feedback loop of the built-in delay node. Turning this up creates an echoing, atmospheric wash of sound.
- **Master Volume:** Overall output gain.

### Recording Studio

1. Click **Record** to begin capturing your performance. The timer will start running.
2. Play your instrument using either the XY pad or the keys.
3. Click **Stop Rec** when finished.
4. Click **↓ Save .WAV** to instantly download your uncompressed, high-fidelity audio file.

---

## 🧠 How It Works (Under the Hood)

AETHER relies on the browser's native **Web Audio API** to generate and process sound.

**The Signal Chain:**

1. **OscillatorNode:** Generates the raw waveform (`sine`, `triangle`, `sawtooth`).
2. **GainNode (Envelope):** Controls the volume of the oscillator based on your mouse's Y-position or key releases.
3. **DelayNode & Feedback Loop:** A portion of the audio is routed into a delay node, which feeds back into itself based on your "Space" setting, creating a custom reverb effect.
4. **Master Gain:** Final volume staging.
5. **AudioDestination:** Your speakers.

**Custom WAV Encoder:**
Browsers typically only record compressed audio natively (like `.webm`). To bypass this, AETHER uses a `ScriptProcessorNode` to intercept the raw floating-point audio data before it hits your speakers. When you save your recording, the app mathematically interleaves the left and right stereo channels, converts the floating-point data into 16-bit PCM data, writes a standard 44-byte RIFF/WAVE header, and generates a downloadable Blob entirely client-side.
