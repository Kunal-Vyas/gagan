# 3D Acoustics Implementation

This document describes a concrete application of the Gagan framework (see [CONCEPTS.md](./CONCEPTS.md) for the full framework definition) to model acoustic wave propagation in an elastic solid medium — a discretized volume of lattice points, each capable of carrying energy in three orthogonal acoustic modes. The acoustics model illustrates fundamental wave phenomena including multi-mode propagation, mode conversion, scattering, interference, and resonance in a bounded three-dimensional medium.

---

## Physical Background

In an elastic solid, acoustic energy propagates through three distinct wave modes: one longitudinal (P-wave) that compresses and rarefies the material along the propagation direction, and two transverse shear waves (S-waves) that displace material perpendicular to the propagation direction. When discretized into a lattice of coupled points, the solid becomes a computationally tractable system where each lattice point can carry energy in all three modes simultaneously.

The key distinction from fluids (which support only longitudinal waves) is that solids support shear waves, creating a richer interaction landscape. At interfaces and boundaries between regions of different elastic properties — or between lattice points configured to prefer different coupling channels — energy can convert between P-wave and S-wave modes, producing mode coupling and scattering.

Key physical phenomena observable in this model:

- **P-wave propagation** — Compressional energy spreads outward from excited regions through nearest-neighbor coupling.
- **S-wave propagation** — Shear energy propagates in two orthogonal transverse directions.
- **Mode conversion** — Energy transfers between longitudinal and shear modes at configuration boundaries, analogous to P-to-S and S-to-P conversion at geological interfaces.
- **Scattering** — Propagating wavefronts disperse when encountering regions of different preferred coupling orientation.
- **Interference** — Overlapping wavefronts from multiple sources combine constructively or destructively.
- **Standing waves and resonance** — Bounded geometries support stationary patterns at characteristic frequencies.

---

## Dimension Mapping

The three dimensions are ordered as follows within each constituent matrix:

| Index | Symbol | Type        | Description                              |
|-------|--------|-------------|------------------------------------------|
| 0     | L      | Longitudinal| Compressional wave energy (P-wave mode) |
| 1     | S₁     | Shear-1     | Transverse wave energy along first shear axis |
| 2     | S₂     | Shear-2     | Transverse wave energy along second shear axis |

The longitudinal dimension L represents compressional energy — the lattice point is either compressed or at equilibrium. The two shear dimensions S₁ and S₂ represent transverse displacement energy along two orthogonal axes perpendicular to each other and to the propagation direction. In an isotropic solid, S₁ and S₂ are degenerate (interchangeable), but lattice anisotropy can break this symmetry.

---

## Value Semantics

Each element in a constituent matrix holds either `0` or `1`:

- **0** — The lattice point is at *equilibrium* along that acoustic mode (no wave energy).
- **1** — The lattice point is *active* along that mode (carries wave energy).

The eight possible states for a constituent:

| State          | Interpretation                              |
|----------------|---------------------------------------------|
| `[[0, 0, 0]]`  | At rest — no acoustic energy in any mode    |
| `[[1, 0, 0]]`  | P-wave active only                          |
| `[[0, 1, 0]]`  | S₁-wave active only                        |
| `[[0, 0, 1]]`  | S₂-wave active only                        |
| `[[1, 1, 0]]`  | P-wave and S₁-wave active                  |
| `[[1, 0, 1]]`  | P-wave and S₂-wave active                  |
| `[[0, 1, 1]]`  | Both shear modes active                    |
| `[[1, 1, 1]]`  | Fully active — all three modes carry energy |

This binary discretization approximates continuous wave amplitude: a point is either significantly excited or it is not along each mode. While coarse, this abstraction captures the qualitative behavior of multi-mode wave propagation and is computationally efficient.

---

## Field Representation

The acoustic medium is represented as a matrix of constituent matrices. Each constituent occupies a position in the outer matrix and represents one lattice point in the discretized volume. For visualization and analysis, the field is typically represented as a 2D slice through a 3D volume (or a 2D membrane-like approximation), with each point carrying three-mode acoustic state.

Example — a 3×3 field slice (nine lattice points):

```text
[
  [[1, 0, 0], [0, 0, 0], [0, 0, 1]],
  [[0, 0, 0], [1, 1, 1], [0, 0, 0]],
  [[0, 1, 0], [0, 0, 0], [0, 0, 0]]
]
```

| Position | Constituent   | Interpretation                     |
|----------|---------------|------------------------------------|
| (0, 0)   | `[[1, 0, 0]]` | P-wave active                      |
| (0, 1)   | `[[0, 0, 0]]` | At rest                            |
| (0, 2)   | `[[0, 0, 1]]` | S₂-wave active                     |
| (1, 0)   | `[[0, 0, 0]]` | At rest                            |
| (1, 1)   | `[[1, 1, 1]]` | Fully active (center)              |
| (1, 2)   | `[[0, 0, 0]]` | At rest                            |
| (2, 0)   | `[[0, 1, 0]]` | S₁-wave active                     |
| (2, 1)   | `[[0, 0, 0]]` | At rest                            |
| (2, 2)   | `[[0, 0, 0]]` | At rest                            |

In this example, the center point carries energy in all three modes (a fully excited source), while three corner points each carry energy in a single mode — a pattern representing a point source with multi-mode radiation into an initially quiet medium.

---

## Configurations

Since the acoustic field has three wave modes, each constituent can be in one of three configurations. A configuration determines which acoustic mode the lattice point primarily uses to interact with its neighbors — i.e., which mode governs energy transfer.

The three configurations are complementary: a constituent in L-conf cannot simultaneously be in S₁-conf or S₂-conf.

Using superscript notation:

| Configuration | Label    | Oriented mode | Notation example    |
|---------------|----------|---------------|---------------------|
| C₁            | L-conf   | L (index 0)   | `[[1, 0, 0]]ᴸ`      |
| C₂            | S₁-conf  | S₁ (index 1)  | `[[1, 0, 0]]ˢ¹`     |
| C₃            | S₂-conf  | S₂ (index 2)  | `[[1, 0, 0]]ˢ²`     |

Two constituents can share the same values but differ in configuration. For instance, `[[1, 1, 0]]ᴸ` and `[[1, 1, 0]]ˢ¹` are bi-modally active but oriented toward different coupling channels. Their behavior during neighbor interactions will differ according to the defined rules.

### Physical Interpretation of Configurations

Configurations specify the *preferred coupling channel* for a lattice point, modeling local anisotropy in the elastic medium:

- A **L-conf** point transfers energy to neighbors primarily through the longitudinal mode, even if it also carries shear energy. This models a region where the medium's elastic properties favor compressional coupling — e.g., a region with high bulk modulus relative to shear modulus.

- A **S₁-conf** point transfers energy primarily through the first shear mode, modeling a region favoring shear coupling along one axis.

- A **S₂-conf** point transfers energy primarily through the second shear mode, modeling a region favoring shear coupling along the orthogonal axis.

A uniform L-conf field represents an isotropic medium where only compressional waves propagate coherently. A checkerboard or random configuration pattern represents a heterogeneous medium with varying local elastic properties — the realistic case in geological and engineered materials.

---

## Interaction Rules

When two adjacent constituents interact, the outcome depends on both their values and their configurations. These rules are customizable; the following set is motivated by elastic wave mechanics in heterogeneous solids.

### Rule Set

- **Same configuration, same values** — *Equilibrium.* No energy transfer. The two points are in identical states with identical preferred coupling channels — no driving force for interaction. This models a uniform region of the medium at rest or in coherent motion.

- **Same values, config mismatch** — *Mode coupling.* The two points carry identical acoustic energy but prefer different coupling channels. Energy is exchanged between the mismatched modes: energy may flow from the longitudinal mode of one point to the shear mode of the other, or between the two shear modes. This models P-to-S and S-to-S mode conversion at interfaces between regions of different elastic anisotropy — a well-known phenomenon in seismology.

  - **Longitudinal–shear mismatch** (e.g., L-conf vs. S₁-conf): Energy converts between P-wave and S-wave modes. Analogous to P-to-S conversion at a geological boundary.
  - **Shear–shear mismatch** (S₁-conf vs. S₂-conf): Energy redistributes between the two shear modes. Analogous to shear wave splitting in anisotropic media.

- **Same configuration, different values** — *Coherent propagation.* Energy flows from the more active point to the less active point along the shared configured mode. For example, `[[1, 0, 0]]ᴸ` adjacent to `[[0, 0, 0]]ᴸ` causes P-wave energy to propagate from the first to the second — analogous to a compressional wavefront advancing through the lattice.

- **Different values, config mismatch** — *Scattering.* The interaction is complex: energy may partially propagate along each point's preferred mode while also scattering into the other modes. This models wave scattering at boundaries between regions of different anisotropy — the propagating wavefront disperses rather than advancing coherently. Standard stochastic evolution applies unless overridden by a specific scattering rule.

### Interaction Outcome Table

| Values       | Configs                 | Outcome            | Physical Analog                    |
|--------------|-------------------------|--------------------|------------------------------------|
| Same         | Same                    | Equilibrium        | Uniform region at rest or in phase |
| Same         | Longitudinal–shear      | Mode coupling      | P-to-S / S-to-P conversion         |
| Same         | Shear–shear             | Mode coupling      | Shear wave splitting               |
| Different    | Same                    | Coherent propagation | Wavefront advance               |
| Different    | Mismatch                | Scattering         | Anisotropic scattering             |

---

## Boundary Conditions

The edges of the outer matrix define the field boundary. Boundary behavior significantly affects wave patterns and is configurable:

| Boundary Type   | Acoustic Analog                                   | Description                                                              |
|-----------------|---------------------------------------------------|--------------------------------------------------------------------------|
| Fixed           | Rigid wall (acoustic impedance → ∞)               | Edge points cannot be created. Wave energy at edges reflects with phase inversion (pressure-release-like) or without (rigid wall). |
| Free            | Free surface (acoustic impedance → 0)             | Edge points can expand outward via creation rules. Energy reflects without phase inversion. |
| Periodic        | Periodic medium / infinite crystal lattice        | The field wraps around — opposite edges are treated as adjacent. No edge effects. |
| Absorbing       | Absorbing boundary (anechoic treatment)           | Edge points in the `[[0, 0, 0]]` state for consecutive steps are removed. Energy leaves the medium. |

### Acoustic Interpretation of Boundary Types

- **Fixed (rigid wall)** — The boundary has infinite acoustic impedance. Wave energy cannot escape; all energy is reflected back into the medium. This models a solid embedded in a much stiffer material, or a cavity with rigid walls. Reflection may invert the wave phase (pressure-release) or preserve it (rigid), depending on the implementation choice.

- **Free (free surface)** — The boundary has zero acoustic impedance. The surface can displace freely, and energy reflects without phase inversion. This models the surface of a solid exposed to vacuum or air. The free boundary also allows the medium to physically expand — new lattice points can be created at the edge.

- **Periodic** — The medium has no physical boundary; it repeats indefinitely. This models a crystal lattice with periodic boundary conditions, commonly used in computational solid-state physics to approximate an infinite medium. Wavefronts pass through the boundary and emerge at the opposite edge.

- **Absorbing (anechoic)** — The boundary is treated to absorb wave energy. Energy reaching the edge does not reflect but instead dissipates. This models an anechoic chamber or a perfectly matched layer (PML) in computational acoustics. Dormant edge points are removed over time.

---

## Initial Conditions

An acoustic analysis begins with predefined initial conditions:

1. **Field dimensions** — The size of the outer matrix (e.g., 5×5, 10×10, 20×20 for a 2D slice; or a 3D grid for full volume modeling).
2. **Boundary type** — Fixed, free, periodic, or absorbing.
3. **Initial constituent values** — The acoustic energy pattern at step 0. Common seed patterns:
   - *P-wave point source* — A single `[[1, 0, 0]]` at the center, all others `[[0, 0, 0]]`. Models a compressional pulse that radiates outward.
   - *Multi-mode source* — A single `[[1, 1, 1]]` at the center. Models an explosive or impact source exciting all modes simultaneously.
   - *S-wave line source* — A row or column of `[[0, 1, 0]]`. Models a shear source for directional S-wave propagation.
   - *Plane wave front* — An entire row or column set to `[[1, 0, 0]]`. Models an incident P-wave plane front.
   - *Random seed* — Random binary values across the field. Models thermal excitation or background noise.
4. **Initial configurations** — How constituents are assigned L-conf, S₁-conf, or S₂-conf:
   - *Uniform* — All set to one configuration. Isotropic propagation in one mode only.
   - *Layered* — Horizontal bands of alternating configurations. Models a stratified geological medium with horizontal layering.
   - *Checkerboard* — Regular 2D pattern of all three configurations (e.g., L, S₁, S₂ repeating). Creates a rich mode-coupling lattice.
   - *Random* — Each point assigned randomly. Models a disordered heterogeneous medium with random local anisotropy.
5. **Evolution parameters** — Number of steps, whether configurations are reassigned between steps, etc.

### Example Initial Conditions

**Multi-mode point source, fixed boundary:**

```text
Field: 7×7
Boundary: fixed
Seed: center (3,3) set to [[1, 1, 1]], all others [[0, 0, 0]]
Configurations: layered (rows 0-2: L-conf, rows 3-4: S₁-conf, rows 5-6: S₂-conf)
Steps: 100
```

**P-wave plane front, periodic boundary:**

```text
Field: 10×10
Boundary: periodic
Seed: column 0 all set to [[1, 0, 0]], all others [[0, 0, 0]]
Configurations: all L-conf
Steps: 50
```

**Random source in heterogeneous medium:**

```text
Field: 15×15
Boundary: absorbing
Seed: random binary values (50% density)
Configurations: random assignment (1/3 each for L, S₁, S₂)
Steps: 200
```

---

## Stochastic Evolution

At each evolution step, every element of every constituent matrix is randomly assigned `0` or `1`. This stochastic process models thermal noise — random molecular vibrations that exist in any real solid at finite temperature.

The evolution loop:

```text
1. For each step S from 1 to N:
   a. For each constituent C in the field:
      i.   For each dimension D (L, S₁, S₂):
             - Generate random value V ∈ {0, 1}
             - Assign C[D] = V
             - Publish event VALUE_ASSIGNED(C, D, V, S)
   b. Evaluate interaction rules between adjacent constituents
      (respecting boundary conditions for neighbor identification)
   c. Evaluate creation/removal conditions
      (respecting boundary type for eligibility)
   d. Apply any event-driven actions
```

For a 7×7 field, each step generates 7 × 7 × 3 = 147 random binary values.

### Note on Stochastic vs. Deterministic Evolution

Pure stochastic evolution (uniform 50/50 random assignment) produces a thermally noisy medium with no coherent wave structure — analogous to a solid at high temperature where thermal phonons overwhelm any organized wave pattern. To observe wave-like behavior, the model should be extended with **biased stochastic generation** (see Extending the Model) where neighbor values influence the probability of assignment, simulating elastic coupling between lattice points. The interaction rules then shape the emergent patterns on top of this coupled random process.

With biased generation, a quiet point adjacent to an active P-wave point has a higher probability of becoming P-wave active on the next step — modeling compressional wave propagation through the lattice. Without bias, active and quiet states appear randomly regardless of neighbors.

---

## Event System

The event system is identical to the framework standard:

| Event                 | Payload                              | Description                                  |
|-----------------------|--------------------------------------|----------------------------------------------|
| `VALUE_ASSIGNED`      | `(constituent, dimension, value, step)` | An acoustic mode was assigned a value          |
| `CONSTITUENT_CREATED` | `(position, values, config, step)`   | A new lattice point was added (free boundary)  |
| `CONSTITUENT_REMOVED` | `(position, step)`                   | A lattice point was removed (absorbing boundary) |
| `INTERACTION`         | `(c1, c2, rule, step)`               | Two adjacent points interacted via a rule     |

### Acoustics-Specific Subscribers

- **P-wave front tracker** — Subscribes to `VALUE_ASSIGNED` for dimension L, identifies the leading edge of P-wave activity at each step, and plots front position over time to measure compressional wave speed.

- **S-wave front tracker** — Subscribes to `VALUE_ASSIGNED` for dimensions S₁ and S₂, tracks shear wavefronts separately. Comparing P-wave and S-wave front speeds reveals the velocity ratio Vp/Vs — a key diagnostic in seismology.

- **Mode energy monitor** — Counts constituents with `L = 1`, `S₁ = 1`, and `S₂ = 1` separately at each step, plotting the energy partition between modes over time. Mode coupling events should cause transfers between these three energy pools.

- **Mode conversion detector** — Subscribes to `INTERACTION` events with the "mode coupling" rule, logging the configuration pair involved (L–S₁, L–S₂, or S₁–S₂) and the values exchanged. This tracks where and how often P-to-S and S-to-S conversions occur.

- **Scattering mapper** — Subscribes to `INTERACTION` events with the "scattering" rule, recording the locations of scattering events. Clusters of scattering events indicate configuration boundaries where wavefronts are disrupted.

- **Interference mapper** — After each step, scans for regions where adjacent points have the same active mode and same configuration, logging coherent clusters. Overlapping wavefronts from multiple sources produce interference patterns detectable as spatial correlations in activity.

- **Standing wave detector** — Subscribes to `VALUE_ASSIGNED`, tracks positions that oscillate between active and rest with regular periodicity, flagging potential standing wave nodes and antinodes. Separate detectors for L, S₁, and S₂ modes can identify mode-specific standing wave patterns.

- **Resonance analyzer** — Compares standing wave pattern persistence across different field sizes and boundary conditions, identifying resonant geometries where patterns persist significantly longer than average.

---

## Creation and Removal of Constituents

Boundary conditions govern whether the acoustic medium can grow or shrink.

### Creation (Free Boundary)

New lattice points are created at the field edge when:

- **Acoustic expansion** — A boundary constituent with any active acoustic mode (`[[1, *, *]]`, `[[*, 1, *]]`, or `[[*, *, 1]]`) has no neighbor in at least one outward direction. A new `[[0, 0, 0]]` constituent is created in that direction with a randomly assigned configuration. This models a free surface that can displace outward under acoustic pressure.

- **Sustained pressure expansion** — A boundary constituent in the `[[1, 1, 1]]` state (all modes active) for more than K consecutive steps causes new constituents to be created in all empty outward positions. This models sustained multi-mode excitation at the surface driving material expansion — analogous to ablation or erosion.

### Removal (Absorbing Boundary)

Lattice points are removed when:

- **Edge absorption** — A boundary constituent has been `[[0, 0, 0]]` for more than M consecutive steps. It is removed, shrinking the field. This models acoustic energy leaving through an absorbing boundary, with dormant lattice points representing material that has been "consumed" by the absorbing treatment.

- **Interior collapse** — A non-boundary constituent has been `[[0, 0, 0]]` for more than M steps *and* all its neighbors are also `[[0, 0, 0]]`. It is removed, creating a void in the medium. This models structural failure under sustained absence of acoustic energy — a region that has been completely isolated from wave activity may represent a cavity or crack nucleation site.

Fixed and periodic boundaries do not trigger creation or removal — the field size remains constant.

---

## Worked Example

### Setup

```text
Field: 3×3
Boundary: absorbing
Initial state:
  (0,0): [[0, 0, 0]]ᴸ    (0,1): [[0, 0, 0]]ˢ¹    (0,2): [[0, 0, 0]]ᴸ
  (1,0): [[0, 0, 0]]ˢ¹    (1,1): [[1, 1, 1]]ˢ²    (1,2): [[0, 0, 0]]ˢ¹
  (2,0): [[0, 0, 0]]ᴸ    (2,1): [[0, 0, 0]]ˢ²    (2,2): [[0, 0, 0]]ᴸ
Subscribed actions: edge absorption (M = 2), interior collapse (M = 3)
```

The center point (1,1) carries energy in all three acoustic modes and is in S₂-conf. All edge points are at rest. Configurations follow a pattern where edges alternate between L-conf and S₁-conf, with the center in S₂-conf — creating configuration boundaries between the center and all its neighbors.

### Step 1 — Stochastic Assignment

```text
  (0,0): [[0, 1, 0]]ᴸ    (0,1): [[1, 0, 0]]ˢ¹    (0,2): [[0, 0, 0]]ᴸ
  (1,0): [[0, 0, 1]]ˢ¹    (1,1): [[1, 0, 1]]ˢ²    (1,2): [[0, 1, 0]]ˢ¹
  (2,0): [[0, 0, 0]]ᴸ    (2,1): [[1, 1, 0]]ˢ²    (2,2): [[0, 0, 1]]ᴸ
```

Events published: 27 `VALUE_ASSIGNED` events.

**Interaction check (selected pairs):**
- (1,1) `[[1, 0, 1]]ˢ²` and (0,1) `[[1, 0, 0]]ˢ¹`: L-values same (both 1), S₁-values same (both 0), S₂-values different (1 vs 0), config mismatch (S₂ vs S₁) → *scattering*. The wave energy at the center encounters a configuration boundary and disperses.
- (1,1) `[[1, 0, 1]]ˢ²` and (1,0) `[[0, 0, 1]]ˢ¹`: S₂-values same (both 1), config mismatch (S₂ vs S₁) → *mode coupling* (shear–shear mismatch). Energy may transfer between S₁ and S₂ modes at these points.
- (1,1) `[[1, 0, 1]]ˢ²` and (1,2) `[[0, 1, 0]]ˢ¹`: all values different, config mismatch → *scattering*.
- (1,1) `[[1, 0, 1]]ˢ²` and (2,1) `[[1, 1, 0]]ˢ²`: L-values same (both 1), different S₁ and S₂ values, same config → *coherent propagation* for L-mode (no energy transfer needed since values match), but overall values differ so *coherent propagation* for the mismatched modes.
- (0,0) `[[0, 1, 0]]ᴸ` and (0,1) `[[1, 0, 0]]ˢ¹`: different values, config mismatch (L vs S₁) → *scattering*.

**Removal check:** No boundary point has been `[[0, 0, 0]]` for consecutive steps yet (this is step 1). No removal.

### Step 2 — Stochastic Assignment

```text
  (0,0): [[0, 0, 0]]ᴸ    (0,1): [[0, 0, 0]]ˢ¹    (0,2): [[0, 0, 0]]ᴸ
  (1,0): [[0, 0, 0]]ˢ¹    (1,1): [[1, 1, 0]]ˢ²    (1,2): [[0, 0, 0]]ˢ¹
  (2,0): [[0, 0, 0]]ᴸ    (2,1): [[0, 0, 0]]ˢ²    (2,2): [[0, 0, 0]]ᴸ
```

**Interaction check:** Only (1,1) is active with `[[1, 1, 0]]ˢ²`. All its neighbors are `[[0, 0, 0]]` — different values from (1,1). All neighbor config mismatches (S₂ vs L or S₂ vs S₁) yield *scattering*. The multi-mode energy at the center cannot propagate coherently because every neighbor has a different configuration.

**Removal check:** All eight edge points are `[[0, 0, 0]]` for the second consecutive step. The threshold M = 2 is met. All edge points are removed via edge absorption.

Field after step 2:

```text
  (1,1): [[1, 1, 0]]ˢ²
```

The medium has collapsed to a single point — the acoustic energy at the center was surrounded by configuration boundaries that scattered all propagation attempts, and the absorbing boundary steadily consumed all inactive edge points before the energy could reach them.

### Step 3 — Stochastic Assignment

```text
  (1,1): [[0, 1, 0]]ˢ²
```

**Interaction check:** (1,1) has no neighbors — no interactions to evaluate.

**Removal check:** (1,1) is now `[[0, 1, 0]]` — not fully zero (S₁ is active), so it is not eligible for removal. The S₁-mode energy persists even though L and S₂ have gone to zero.

Field after step 3:

```text
  (1,1): [[0, 1, 0]]ˢ²
```

The single remaining point retains S₁-mode energy while the other modes are at rest. This illustrates an important property of the 3D model: different modes can decay independently under stochastic generation. If the S₁ energy also goes to zero on the next step and remains there for M consecutive steps, the final point will be removed and the medium will cease to exist.

This worked example demonstrates how **configuration boundaries** (mismatches between neighboring configurations) can effectively trap energy by converting coherent propagation into scattering. In a uniform configuration field, the same center energy would have propagated outward more coherently, potentially sustaining edge points against absorption. The three-configuration system creates richer scattering dynamics than the two-configuration 2D membrane, where alternating checkerboard configurations already produce frequent scattering.

---

## Physical Observations

The [Field Phenomena](./CONCEPTS.md#field-phenomena) section in CONCEPTS.md defines the physical concepts that emerge from the framework's mechanisms. This section describes how each phenomenon manifests specifically in the 3D acoustics model.

### Coupling

The acoustics model exhibits coupling through all four interaction rule outcomes:

- **Equilibrium coupling** (same config, same values) maintains coherent regions where wave energy is uniform.
- **Mode coupling** (same values, config mismatch) transfers energy between L, S₁, and S₂ modes. The three possible mismatch pairs (L–S₁, L–S₂, S₁–S₂) create a richer coupling landscape than the 2D membrane, which has only one mismatch pair.
- **Coherent propagation coupling** (same config, different values) drives wavefront advance within uniform configuration regions.
- **Scattering coupling** (different values, config mismatch) disperses wavefronts at configuration boundaries.

In a field with all three configurations present, coupling occurs across three channels simultaneously, producing complex energy redistribution patterns. The configuration distribution subscriber (see [Analysis Patterns](./CONCEPTS.md#configuration-distribution)) tracks how the ratio of L-conf, S₁-conf, and S₂-conf points evolves over time.

### Propagation

Propagation occurs when the "coherent propagation" rule fires: an active constituent causes a neighbor with the same configuration to adopt similar values. In the acoustics model, propagation can occur independently along each of the three modes:

- **P-wave propagation** — L-mode energy spreads through L-config regions.
- **S₁-wave propagation** — First shear mode energy spreads through S₁-config regions.
- **S₂-wave propagation** — Second shear mode energy spreads through S₂-config regions.

In a uniform L-conf field, only P-wave propagation is coherent — shear energy appears and disappears stochastically without organized spreading. In a heterogeneous field with all three configurations present, each mode propagates coherently only within its configured regions and scatters at configuration boundaries.

In the absence of biased generation, propagation is weak — active values appear and disappear stochastically. With biased generation, wavefronts advance coherently at a speed determined by the coupling strength parameter λ.

### Reflection

Under fixed (rigid wall) boundaries, acoustic energy reaching the edge reflects inward. Depending on the implementation, reflection may be with or without phase inversion:

- **Without inversion** (rigid wall): An active P-wave point at the boundary causes its interior neighbor to become active — energy reflects preserving the compressional phase.
- **With inversion** (pressure-release surface): An active P-wave point at the boundary causes its interior neighbor to become inactive — energy reflects with a phase flip, analogous to a sound wave reflecting off an open end of a pipe.

Under periodic boundaries, energy does not reflect but instead wraps around, emerging at the opposite edge.

Under free boundaries, energy reflects without inversion and may also trigger edge expansion creation rules.

Under absorbing boundaries, energy diminishes at the edge without reflection — this is the anechoic condition used to simulate open or infinite domains.

### Interference

Constructive interference occurs when wavefronts from multiple sources overlap in phase. In the acoustics model, this manifests as regions where multiple active neighbors reinforce a point's activity through biased stochastic generation. For example, two P-wave sources on opposite sides of the field will produce a region of sustained P-wave activity where their wavefronts meet.

Destructive interference occurs when overlapping wavefronts are out of phase. In the binary model, this manifests as a point where competing neighbor influences cancel — neither value is strongly favored, and the point remains at rest (or switches randomly). This is less pronounced in the binary model than in continuous-valued models but can still be detected as zones of low activity between active wavefronts.

Multi-mode interference — where P-wave and S-wave fronts overlap — produces complex patterns. At the overlap region, mode coupling may transfer energy between modes, producing secondary wavefronts that differ in character from either parent wave.

### Scattering

Scattering is the defining feature of the 3D acoustics model compared to lower-dimensional implementations. With three configurations, scattering occurs at three types of configuration boundaries:

- **L–S₁ boundaries** — P-waves scattering into shear energy
- **L–S₂ boundaries** — P-waves scattering into the orthogonal shear mode
- **S₁–S₂ boundaries** — One shear mode scattering into the other

In a uniform configuration field, scattering is absent and wavefronts propagate coherently. In a field with all three configurations (e.g., random or layered assignment), scattering is pervasive — any propagating wavefront will encounter configuration mismatches and disperse.

The **scattering mapper** subscriber identifies scattering event locations, enabling visualization of the "scattering landscape" — which regions of the field are opaque (frequent scattering) and which are transparent (uniform configuration, allowing coherent propagation). This maps directly to the concept of acoustic impedance contrast in real materials.

### Damping

Without explicit damping, acoustic energy is sustained only by stochastic generation, which is equally likely to create or destroy active states. When damping is introduced (see Extending the Model), active points have a probability of returning to `[[0, 0, 0]]` each step, independent of their neighbors.

With damping, the mode energy monitor shows declining energy in all three modes over time. The decay rate depends on the damping parameter δ and the coupling strength λ — if coupling can propagate energy faster than damping removes it, the wavefronts advance; otherwise, the field decays to silence.

Damping affects different modes equally in the basic model. A more refined extension could introduce mode-dependent damping — e.g., shear waves damping faster than P-waves, which is physically realistic in many materials where shear attenuation exceeds compressional attenuation.

### Standing Waves

Standing waves emerge under fixed or periodic boundaries when coherent propagation reflects off edges and interferes with itself. In the 3D acoustics model, standing waves can form independently in each mode:

- **P-wave standing waves** — Stationary patterns of longitudinal activity.
- **S₁-wave standing waves** — Stationary patterns of first shear mode activity.
- **S₂-wave standing waves** — Stationary patterns of second shear mode activity.

Each mode has its own characteristic standing wave frequencies determined by the field geometry. In a rectangular field of dimensions L₁ × L₂, the P-wave standing wave frequencies are:

```text
fₘₙ = (Vp / 2) × sqrt((m/L₁)² + (n/L₂)²)
```

where Vp is the P-wave velocity and m, n are integers. Similar expressions hold for S₁ and S₂ modes with their respective velocities.

In the binary stochastic model, standing waves manifest as positions that oscillate between active and rest with regular periodicity while neighboring positions remain relatively stable. The **standing wave detector** subscriber identifies these by tracking the temporal autocorrelation of individual lattice points.

### Resonance

Resonance in the acoustic model occurs when the field dimensions support standing wave patterns that self-reinforce. At resonant frequencies, the standing wave pattern persists much longer than non-resonant patterns because reflected wavefronts arrive back at the source in phase with the next evolution cycle.

In the 3D model, resonance can occur independently for each mode:
- **P-wave resonance** — Field dimensions that support integer half-wavelengths of the P-wave mode.
- **S₁-wave resonance** — Field dimensions that support integer half-wavelengths of the first shear mode.
- **S₂-wave resonance** — Field dimensions that support integer half-wavelengths of the second shear mode.

If the coupling strength λ and damping δ are such that all three modes can sustain activity, a multi-modal resonance may occur where all three standing wave patterns coexist — a complex but physically realistic scenario in bounded solids.

The **resonance analyzer** subscriber identifies resonant geometries by comparing pattern persistence across different field sizes. Fields that produce significantly longer-lived patterns are flagged as resonant.

---

## Comparison to Other Implementations

The 3D acoustics model fills the conceptual gap between the 2D membrane and the 4D spacetime implementations:

| Feature                    | 2D Membrane    | 3D Acoustics                   | 4D Spacetime               |
|----------------------------|----------------|--------------------------------|----------------------------|
| Configurations             | 2 (u, v)       | 3 (L, S₁, S₂)                 | 4 (x, y, z, t)            |
| Mode coupling pairs        | 1 (u–v)        | 3 (L–S₁, L–S₂, S₁–S₂)        | 6 (all spatial pairs + spatial–temporal) |
| Scattering complexity      | Moderate       | Rich                           | Very rich                  |
| Physical domain            | Surface waves  | Bulk elastic waves             | Spacetime fields           |
| Unique phenomena           | 2-mode coupling | P/S mode conversion, shear splitting | Causal interaction |

The 3D model introduces **shear–shear mode coupling** (S₁–S₂), which has no analog in the 2D membrane. This models shear wave splitting — a phenomenon where a single S-wave entering an anisotropic region splits into two orthogonal S-wave components with different velocities. This is a key diagnostic in seismology for detecting anisotropy in the Earth's mantle.

See [Dimensionality Effects](./CONCEPTS.md#dimensionality-effects) in CONCEPTS.md for the general comparison table across dimensionalities.

---

## Extending the Model

The basic 3D acoustics implementation can be extended in several directions:

- **Biased stochastic generation** — Instead of uniform 50/50, set the probability of `V = 1` based on neighbor values in the same dimension and same configuration. For example, if a neighbor has `L = 1` and the same configuration, set `P(L = 1) = 0.7` for the current point. This simulates elastic coupling and produces coherent wave propagation instead of pure stochastic noise.

- **Coupling strength parameter** — Introduce a parameter λ ∈ [0, 1] that controls the neighbor influence bias. λ = 0 is pure stochastic; λ = 1 is fully deterministic (a neighbor's value is always copied). Intermediate values produce varying degrees of wave coherence. Mode-dependent coupling strengths (λL, λS₁, λS₂) can model anisotropic elastic properties.

- **Velocity ratio modeling** — By setting different coupling strengths for longitudinal vs. shear modes, the model naturally produces different propagation speeds for P-waves and S-waves. The ratio Vp/Vs is a critical parameter in seismology (typically ~1.7 for crustal rocks).

- **Configuration reassignment** — Allow configurations to change based on interaction outcomes. For example, after a mode coupling event between L-conf and S₁-conf points, swap their configurations. This models dynamic anisotropy — regions where the preferred coupling channel evolves with the wave field.

- **Damping** — Introduce a probability δ that any active point becomes `[[0, 0, 0]]` independent of its neighbors, simulating viscoelastic attenuation. Mode-dependent damping (δL, δS₁, δS₂) models materials where shear waves attenuate faster than P-waves.

- **Mode-dependent propagation rules** — Extend the interaction rules to treat P-wave propagation differently from S-wave propagation. For example, P-waves could propagate to all four neighbors (including diagonals) while S-waves propagate only to orthogonal neighbors, modeling the different radiation patterns of compressional and shear sources.

- **Continuous-valued extension** — Relax the binary constraint to allow values in [0, 1] or a wider integer range, enabling smoother amplitude representation. This is particularly valuable for interference modeling, where binary values cannot represent partial cancellation.

- **Visualization** — Map L-mode to red, S₁-mode to green, S₂-mode to blue in an RGB image per step. Stacking frames yields a video showing multi-mode wave propagation, mode conversion at configuration boundaries (color mixing), and standing wave patterns. Separate single-channel visualizations for each mode isolate individual wave behaviors.

- **3D field representation** — Extend the outer matrix to three dimensions (e.g., 10×10×10) for true volumetric modeling. This enables study of 3D wave phenomena such as head waves, diffraction around obstacles, and 3D standing wave patterns in cavities.

- **Source mechanisms** — Define specific source patterns beyond simple point sources: dipole sources (opposite values at adjacent points), double-couple sources (specific shear patterns modeling fault slip), and explosive sources (uniform L-mode activation). These map directly to seismological source types.

- **Layered media** — Assign configurations in horizontal bands to model geological layering. Each layer has a dominant configuration (e.g., L-conf for sedimentary basins, S₁-conf for fractured zones), creating mode conversion at every layer boundary — analogous to the real Earth's layered structure.