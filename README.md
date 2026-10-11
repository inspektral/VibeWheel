# VibeWheel

# Hammond B3 Electromechanical Modeling Project

## Project objective

Build a computational reconstruction of the Hammond B3 tone-generation system from its underlying electromechanical mechanisms rather than creating a conventional “Hammond-like” synthesizer.

The central idea is to model the physical generator—motor, gearing, rotating shafts, tonewheels, magnetic pickups, electrical routing and keyboard contacts—and allow the resulting sound to emerge from that model.

The project should begin as an offline Python research implementation. Real-time deployment can come later through Web Audio, SuperCollider, a native DSP implementation, or embedded hardware.

Access to a real Hammond B3 provides an important advantage: individual stages of the model can be measured and validated against the physical instrument, preferably before the Leslie.

---

# 1. Guiding principle

Do not begin with:

> “How can we synthesize a convincing Hammond sound?”

Begin with:

> “What does the physical Hammond B3 do, and how accurately can we reproduce those processes computationally?”

Avoid arbitrary “vintage” effects such as random oscillator drift unless they correspond to an identified physical mechanism.

The implementation should distinguish between:

1. **Ideal model** — correct mechanical frequencies but idealized components.
2. **Physical model** — sound derived from wheel geometry, gearing, pickups and other physical mechanisms.
3. **Empirical model** — measured waveforms/parameters from the reference B3.

These provide useful baselines against one another.

---

# 2. Physical architecture

A classic B3 uses a continuously operating electromechanical tone generator.

The generator contains approximately:

- 91 sounding tonewheels
- 48 rotating two-disc assemblies
- 12 rotational-speed groups
- multiple tonewheel tooth counts
- individual electromagnetic pickups
- a shared generator serving the keyboards

The generator should therefore not be represented as independent oscillators instantiated for each note.

Instead:

```text
Synchronous motor
        │
        ▼
Mechanical drive/gearing
        │
        ▼
12 rotational speed groups
        │
        ▼
48 rotating assemblies
        │
        ▼
91 sounding tonewheels
        │
        ▼
91 magnetic pickup signals
        │
        ▼
Keyboard/contact routing
        │
        ▼
Drawbar summation
```

The tonewheels operate continuously regardless of which keys are pressed.

Keys are principally routing/switching mechanisms.

---

# 3. Shared tonewheel architecture

Tonewheels are reused extensively.

A single physical tonewheel may contribute to multiple keys and drawbar footages. Consequently, two simultaneously sounding components requesting the same physical tonewheel must share:

- frequency
- waveform
- phase
- mechanical irregularities
- pickup characteristics

They must not instantiate independent oscillators.

This is a fundamental architectural requirement.

A conceptual mapping should eventually exist of the form:

```python
tonewheel_id = mapping[
    manual,
    key,
    drawbar
]
```

including the actual Hammond foldback behavior at the extremes of the keyboard.

---

# 4. Mechanical frequency generation

Musical frequencies should initially be derived from the physical generator rather than assigned using MIDI equal temperament.

For a wheel with \(N\) teeth rotating at a given speed:

\[
f = N \frac{\mathrm{RPM}}{60}
\]

The rotational speeds themselves should be derived from the motor and gear train where documentation permits.

The first mechanical model should therefore contain representations of:

- synchronous motor speed
- main drive
- gear ratios
- shaft/assembly rotational speeds
- tonewheel tooth counts
- assembly pairings
- tonewheel numbers
- resulting electrical frequencies

A primary early deliverable is a table such as:

```text
wheel_id
assembly_id
speed_group
gear_ratio
RPM
tooth_count
frequency
nearest musical pitch
deviation from 12-TET
pickup configuration
```

### First major success criterion

The model should generate the B3 tonewheel frequencies from mechanical parameters without entering the desired musical frequencies directly.

---

# 5. Tonewheel waveform model

A normal tonewheel is approximately sinusoidal, but a mathematically perfect sine oscillator should not be assumed to be the final model.

The waveform should ideally emerge from:

```text
tonewheel geometry
        ↓
distance/geometry relative to pickup
        ↓
magnetic flux
        ↓
change in magnetic flux
        ↓
induced pickup voltage
```

A simplified initial formulation can describe the wheel perimeter as:

\[
r(\theta)
\]

and the magnetic flux as a function of instantaneous geometry:

\[
\Phi(\theta)=F(d(\theta))
\]

The induced voltage then follows Faraday's law:

\[
V(t)=-N\frac{d\Phi}{dt}
\]

A full finite-element electromagnetic simulation is not required initially. Begin with simple physically motivated approximations and increase complexity only when measurements justify it.

---

# 6. Individual tonewheel character

Different wheels should be capable of producing slightly different waveforms.

Potential causes include:

- tonewheel geometry
- tooth count
- pickup geometry
- pickup gap
- coil characteristics
- wheel eccentricity
- magnetic differences
- frequency-dependent filtering
- manufacturing tolerances

These differences should preferably be measured or derived rather than randomly generated.

The model should eventually support a representation such as:

```python
ToneWheel(
    id=...,
    assembly=...,
    teeth=...,
    geometry=...,
    pickup=...,
    eccentricity=...,
)
```

---

# 7. Reference B3 measurement program

A real Hammond B3 is available for comparison.

This should be treated as the project's reference measurement system.

Record as close as practical to the pre-Leslie electrical signal so that generator behavior can be studied independently of Leslie acoustics.

Measurements should be reproducible and metadata-rich.

For every recording retain:

- organ model and serial if appropriate
- signal extraction point
- sample rate/bit depth
- recording interface and gain
- manual
- key
- drawbar registration
- percussion state
- vibrato/chorus state
- expression position
- organ warm-up state/time
- date/session

---

# 8. Single-tonewheel identification

An important experiment is to isolate nominally individual tonewheel components using one drawbar at a time.

For a measured wheel, estimate a periodic representation:

\[
x_i(t)=
\sum_{k=1}^{K}
A_{i,k}
\sin(2\pi k f_i t+\phi_{i,k})
\]

Measure:

- fundamental frequency
- harmonic amplitudes
- harmonic phases where meaningful
- amplitude modulation
- frequency/phase modulation
- noise
- leakage components

Then compare:

```text
ideal sinusoid
      vs
physical simulation
      vs
real B3 measurement
```

This establishes whether a detailed wheel/pickup model provides meaningful improvements over simple sinusoidal generation.

---

# 9. Tonewheel reuse as an experimental tool

Because one physical wheel can be accessed through multiple key/drawbar combinations, the architecture provides a useful identification experiment.

Measure the same wheel through several different keyboard routes.

If the resulting signals differ, those differences cannot originate solely from the wheel itself.

This can help separate:

```text
tonewheel properties
        from
pickup properties
        from
wiring/contact/network properties
```

Comparisons between the upper and lower manuals may be particularly useful.

---

# 10. Empirical wavetable baseline

As a baseline, measured periodic waveforms may be stored directly.

For example:

```python
measured_wheels[91][2048]
```

Even 91 2048-sample float32 waveforms require less than 1 MB.

This provides a highly practical reference model.

However, measured wavetables should not replace the physical modeling effort. Instead maintain three comparable generator modes:

### Ideal

Mechanically correct frequency + ideal sinusoidal waveform.

### Empirical

Measured waveform for each physical wheel.

### Physical

Waveform produced from the computational wheel/pickup model.

The difference between these models is itself an important research result.

---

# 11. Physically motivated imperfections

Do not introduce generic analog-synth imperfections without justification.

Candidate physical mechanisms include:

### Gear-related pitch deviations

The generator's mechanical gearing determines its tuning.

### Backlash

Mechanical gear behavior may create small correlated phase/frequency variations.

### Eccentricity

A slightly off-center wheel changes its distance from the pickup periodically:

\[
d(t)=d_0+\epsilon\cos(\omega t+\phi)
\]

This can introduce deterministic amplitude modulation.

### Electromagnetic leakage

A pickup may contain weak components originating from nearby wheels.

This can be modeled as:

\[
y_i=x_i+\sum_j L_{ij}x_j
\]

where \(L\) is a sparse leakage matrix informed by physical proximity and/or measurements.

### Shared mechanical behavior

Mechanical disturbances should be correlated where components share a physical mechanism. Independent random detuning of 91 oscillators is not an appropriate substitute.

---

# 12. Keyboard and drawbar model

After validating the generator itself, implement the keyboard/contact network.

The organ should consist of continuously operating wheel signals that are selected and summed by key contacts.

Conceptually:

```text
key press
   │
   ├── drawbar 16'
   ├── drawbar 5⅓'
   ├── drawbar 8'
   ├── drawbar 4'
   ├── drawbar 2⅔'
   ├── drawbar 2'
   ├── drawbar 1⅗'
   ├── drawbar 1⅓'
   └── drawbar 1'
           │
           ▼
corresponding physical tonewheels
```

The model must eventually reproduce:

- upper manual
- lower manual
- pedal system
- drawbar routing
- tonewheel reuse
- foldback
- contact behavior

---

# 13. Drawbar levels

Do not initially assume that drawbar position maps linearly to amplitude.

For example, avoid assuming:

```python
amplitude = drawbar_position / 8
```

Measure or document the actual attenuation relationships.

Useful reference measurements include isolated and combined registrations such as:

```text
800000000
080000000
880000000
```

Comparison of isolated signals with their combination can also reveal nonlinearities, loading or downstream saturation.

---

# 14. Later organ subsystems

Once the steady-state generator and keyboard routing are validated, add additional subsystems individually.

Suggested order:

1. Contact/keying behavior
2. Key click
3. Percussion
4. Leakage
5. Chorus/vibrato scanner
6. Expression system
7. Preamp/electronics
8. Leslie

Each subsystem should have an isolated validation experiment where possible.

The Leslie should deliberately remain a separate late-stage project because it introduces substantial electromechanical and acoustic complexity.

---

# 15. Leslie validation strategy

Access to a real Leslie creates a useful eventual A/B experiment.

Compare:

```text
Real B3 generator ───────┐
                         ├──► same real Leslie
Virtual B3 generator ────┘
```

This controls for the Leslie and room while evaluating the generator/electronics model.

Only after the virtual generator is satisfactory should a computational Leslie model become a priority.

---

# 16. Initial software architecture

Python is the preferred research environment.

Suggested structure:

```text
tonewheel/
│
├── data/
│   ├── generator.yaml
│   ├── wheel_geometry/
│   ├── measurements/
│   └── analysis/
│
├── tonewheel/
│   ├── mechanics.py
│   ├── wheel.py
│   ├── pickup.py
│   ├── generator.py
│   ├── leakage.py
│   ├── contacts.py
│   ├── drawbars.py
│   └── organ.py
│
├── experiments/
│   ├── 01_gearing.ipynb
│   ├── 02_single_wheel.ipynb
│   ├── 03_pickup.ipynb
│   ├── 04_real_b3_analysis.ipynb
│   └── 05_model_vs_real.ipynb
│
└── tests/
```

Initial dependencies can remain small:

```text
numpy
scipy
matplotlib
soundfile
pyyaml
```

Additional dependencies should be introduced only when they provide clear value.

---

# 17. Separation of physical model and real-time renderer

The research implementation should not become tied to SuperCollider, Web Audio or any other runtime.

The physical model should ultimately produce compact representations such as periodic waveforms or Fourier coefficients.

For example:

```text
physical B3 model
       │
       ▼
91 wheel descriptions
       │
       ├──► Python offline renderer
       ├──► Web Audio / AudioWorklet
       ├──► SuperCollider
       ├──► native C++/JUCE
       └──► embedded/headless instrument
```

This permits the same validated generator to drive multiple implementations.

---

# 18. Web implementation opportunity

A web version could be particularly valuable because the model is both an instrument and an educational visualization.

A future interface could display the physical generator and allow users to:

- play the manuals
- move drawbars
- inspect individual tonewheels
- see which wheels a pressed key connects to
- visualize shared wheels between notes
- inspect mechanical RPM and tooth count
- display resulting waveforms
- display spectra
- compare ideal/physical/measured models
- inspect leakage
- switch between generator stages

The web version should therefore be considered a potential primary frontend rather than merely a demo.

---

# 19. Standalone hardware opportunity

A later implementation could run headlessly on a small computer with MIDI controllers.

Conceptually:

```text
Upper MIDI manual ─┐
                   │
Lower MIDI manual ─┼──► embedded computer
                   │          │
Pedals ────────────┘          ▼
                        B3 generator
                             │
                       audio output
```

Physical drawbars and organ controls could be added through MIDI or a custom controller.

The underlying generator is computationally lightweight enough that the challenge is expected to lie primarily in accurate modeling, routing and peripheral integration rather than raw compute requirements.

---

# 20. Development roadmap

## Phase 0 — Historical/technical research

Establish authoritative mechanical and electrical documentation.

**Deliverable:** documented generator specification with sources.

## Phase 1 — Mechanical generator

Represent the complete assembly/gear/tonewheel system in Python.

**Deliverable:** all tonewheel frequencies generated mechanically.

## Phase 2 — Single-wheel physical model

Model wheel geometry, magnetic interaction and pickup voltage.

**Deliverable:** simulated waveform and spectrum for representative wheels.

## Phase 3 — B3 measurement dataset

Record isolated components from the reference instrument.

**Deliverable:** reproducible measurement dataset with metadata.

## Phase 4 — System identification

Compare physical simulations against measurements and estimate uncertain parameters.

**Deliverable:** validated wheel/pickup models.

## Phase 5 — Complete generator

Run all physical tonewheel sources with correct shared behavior.

**Deliverable:** offline 91-wheel generator.

## Phase 6 — Keyboard/contact network

Implement manuals, drawbars, tonewheel reuse and foldback.

**Deliverable:** polyphonic playable offline B3 core.

## Phase 7 — Physical imperfections

Implement measured leakage, eccentricity, backlash and other justified mechanisms.

**Deliverable:** calibrated generator model.

## Phase 8 — Organ-specific articulation

Implement key click and percussion.

## Phase 9 — Electronics

Implement scanner chorus/vibrato, expression and relevant amplification/electronics.

## Phase 10 — Real-time deployment

Port the validated model to Web Audio, SuperCollider, native DSP and/or embedded hardware.

## Phase 11 — Leslie

Model and validate the rotary speaker independently.

---

# 21. Immediate next task

Do not begin with audio synthesis.

First reconstruct the complete B3 tone generator in machine-readable form.

Create `generator.yaml` containing, as far as reliable documentation permits:

- tonewheel IDs
- assembly IDs
- disc pairings
- speed groups
- drive gears
- gear ratios
- rotational speeds
- tooth counts
- pickup types/configurations
- resulting frequencies

Then create a Python notebook that:

1. Loads this physical specification.
2. Reconstructs the generator topology.
3. Calculates all rotational speeds.
4. Calculates all generated frequencies.
5. Maps frequencies to their nearest musical pitches.
6. Reports tuning deviations.
7. Visualizes the 48 assemblies and 91 sounding wheels.

### Initial acceptance criterion

**The program produces the Hammond B3's tonewheel frequency set from its mechanical construction without using the desired musical frequencies as input.**

Once that works, proceed to modeling one physical tonewheel and pickup.

---

# 22. Research philosophy

Prefer explanations over approximations and measurements over folklore.

If a measured Hammond behavior can be explained by a specific physical mechanism, model that mechanism.

If a simpler model is perceptually and quantitatively indistinguishable from a more complicated one, retain the simpler model.

Keep physical parameters, measured parameters and fitted parameters clearly separated.

Preserve raw measurements so models can be reevaluated later.

The long-term goal is not merely a convincing organ preset. It is an interpretable, validated computational reconstruction of the Hammond B3 whose signal can be followed from the synchronous motor and rotating steel wheels all the way to the final audio output.
