# TouchDesigner Project Prompt: Inside a Fusion Reactor – Audio-Reactive Visualization

## Project Goal

Create a hypnotic, sci-fi inspired 3D audio-reactive scene that feels like being inside a tokamak-style fusion reactor core.  
Plasma streams swirl and pulse to music, magnetic turbulence creates organic movement, heat blooms flare on high frequencies, and containment-field-like toroidal geometry holds everything together.

Target aesthetic: glowing plasma blues → hot reds/oranges on peaks, radiant bloom, subtle trails, contained toroidal energy field.

Keep implementation **simple**, reliable, and based on native TouchDesigner operators + common community patterns (instancing, audioAnalysis, feedback, bloom).

Estimated build time: 45–90 minutes for intermediate users.

## Core Concept & Reactivity Mapping

- **Low frequencies (bass)** → Plasma volume/pulsing scale (breathing containment field)
- **Mid frequencies** → Swirl/rotation speed + turbulence/noise strength (magnetic instability)
- **High frequencies** → Color temperature shift (blue → red/orange) + spark/heat intensity + bloom strength
- **Beat/Kick detection** → Sudden particle bursts or energy flares from center

## Required Components & Structure

### 1. Audio Pipeline

- Audio File In CHOP or Audio Device In CHOP (source)
- audioAnalysis COMP (from Palette → Tools)
  - Enable: Low, Mid, High, Kick/Snare (optional)
  - Adjust threshold/smoothing per band
- For each band (Low / Mid / High / Kick):
  - Select CHOP → Math CHOP (remap & scale) → Filter CHOP (smooth) → Null CHOP (export)

### 2. Geometry – Reactor Chamber

Base shape: Torus (tokamak ring)

- Torus SOP (inner plasma ring)
  - Major Radius: ~5–6
  - Minor Radius: ~0.8–1.2
- Optional outer containment ring:
  - Duplicate Torus → scale up slightly → Phong MAT (dark metallic, low reflectivity)

Plasma field (instanced):

- Line SOP or Circle SOP (circular point distribution, 80–200 points)
- Instance small geometry on points:
  - Options: Sphere SOP (small), Line SOP (streamers), or Grid SOP sliced thin
- Noise SOP or Transform SOP before instancing for organic deformation

### 3. Animation & Reactivity

- **Rotation / Swirl**
  - Rotate SOP or CHOP → connect Mid value to rotate speed (Y-axis dominant)
  - Range suggestion: speed = Mid \* 8–15

- **Pulsing Scale**
  - Uniform Scale on plasma Torus or instances
  - Expression: 1 + (Low \* 0.4–0.8)

- **Turbulence**
  - Noise CHOP or Noise SOP
  - Amplitude driven by Mid (0.1–1.0 range)
  - Apply to Translate or Position of instances

- **Color Temperature**
  - Constant MAT or PBR MAT with emissive
  - Use Ramp TOP + Lookup TOP driven by High
  - Suggested gradient: blue-purple (low) → white-yellow → red-orange (high peaks)

- **Beat Events**
  - Beat CHOP or Kick channel from audioAnalysis
  - Trigger Particle SOP burst (emit from center, short life, velocity = audio peak)

### 4. Rendering & Post-Processing

- Geometry COMP (with instances)
- Camera COMP (inside view, slight fisheye or wide FOV)
- Point Light or Area Light in center (warm color, high intensity)
- Render TOP
- Post chain:
  - Feedback TOP (low opacity, decay ~0.92–0.96, amount modulated by High)
  - Bloom TOP (threshold 0.6–0.9, intensity 2–6, size 5–15)
  - Optional: Blur TOP or Displace TOP (heat haze)
  - Level TOP or Grade TOP for final contrast/glow control

### 5. Optional Enhancements (Claude/Python integration)

- Script CHOP or DAT to:
  - Generate procedural helix paths from audio
  - Advanced color mapping using HSV → RGB conversion
  - Custom FFT processing for finer band control

### Performance Targets

- Instances: 150–500 max
- Keep CHOP chains short (use Nulls)
- Use Texture 3D or low-res if heavy post-processing

### Testing Music Suggestions

- Dark electronic / techno
- Drum & bass
- Cinematic sci-fi soundtracks with strong bass and high-frequency sparkle

## Quick Parameter Cheat Sheet

- Plasma scale mult: 0.4–1.0 × Low
- Swirl speed: 8–20 × Mid
- Bloom intensity: 2 + (High × 8)
- Feedback amount: 0.1–0.4 × High
- Color hue shift: map High 0–1 → 200° (blue) to 30° (red-orange)

## Final Mood Keywords

glowing plasma, tokamak core, magnetic confinement, fusion ignition, radiant heat, contained energy, sci-fi reactor interior, pulsing magnetic fields

Good luck building your fusion core – make it breathe with the music!
