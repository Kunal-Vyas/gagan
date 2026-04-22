# 2D Membrane Implementation

This document describes a concrete application of the Gagan framework (see [CONCEPTS.md](./CONCEPTS.md) for the full framework definition) to model a discretized elastic membrane — a two-dimensional surface composed of coupled lattice points, each capable of carrying displacement in two orthogonal directions. The membrane model illustrates fundamental wave mechanics concepts including propagation, interference, mode coupling, and boundary effects.

---

## Physical Background

A membrane is a thin, flexible surface that can vibrate and transmit waves. When discretized into a lattice of mass points connected by elastic couplings, the membrane becomes a computationally tractable system. Each lattice point can be displaced from equilibrium along two orthogonal axes, giving rise to two independent wave modes that can interact through the lattice coupling.

Key physical phenomena observable in this model:

- **Wave propagation** — Displacement energy spreads outward from excited regions through nearest-neighbor coupling.
- **Interference** — Overlapping wavefronts from multiple sources combine constructively or destructively.
- **Standing waves** — Waves reflecting off boundaries create stationary patterns at resonant frequencies.
- **Mode coupling** — Energy transfers between the two orthogonal displacement modes when lattice points are configured differently.
- **Boundary effects** — Fixed, free, or periodic boundaries shape wave reflection and pattern formation.

---

## Dimension Mapping

The two dimensions are ordered as follows within each constituent matrix:

| Index | Symbol | Type        | Description                          |
|-------|--------|-------------|--------------------------------------|
| 0     | u      | Longitudinal | Displacement along the first axis   |
| 1     | v      | Transverse   | Displacement along the second axis  |

Both dimensions represent wave propagation modes on the membrane surface. The choice of "longitudinal" and "transverse" is a naming convention; the two axes are interchangeable in an isotropic membrane.

---

## Value Semantics

Each element in a constituent matrix holds either `0` or `1`:

- **0** — The lattice point is at *equilibrium* along that displacement axis (no displacement energy).
- **1** — The lattice point is *displaced* along that axis (carries displacement energy).

The four possible states for a constituent:

| State          | Interpretation                              |
|----------------|---------------------------------------------|
| `[[0, 0]]`     | At rest — no displacement in either axis    |
| `[[1, 0]]`     | Displaced along u only                     |
| `[[0, 1]]`     | Displaced along v only                     |
| `[[1, 1]]`     | Bi-axially displaced — both modes active   |

This binary discretization approximates continuous displacement: a point is either significantly displaced or it is not. While coarse, this abstraction captures the qualitative behavior of wave propagation and is computationally efficient.

---

## Field Representation

The membrane is represented as a matrix of constituent matrices. Each constituent occupies a position in the outer matrix and represents one lattice point on the discretized membrane.

Example — a 3×3 membrane (nine lattice points):

```text
[
  [[1, 0], [0, 0], [0, 1]],
  [[0, 0], [1, 1], [0, 0]],
  [[0, 1], [0, 0], [1, 0]]
]
```

| Position | Constituent   | Interpretation                     |
|----------|---------------|------------------------------------|
| (0, 0)   | `[[1, 0]]`    | u-mode active                      |
| (0, 1)   | `[[0, 0]]`    | At rest                            |
| (0, 2)   | `[[0, 1]]`    | v-mode active                      |
| (1, 0)   | `[[0, 0]]`    | At rest                            |
| (1, 1)   | `[[1, 1]]`    | Bi-axially displaced (center)      |
| (1, 2)   | `[[0, 0]]`    | At rest                            |
| (2, 0)   | `[[0, 1]]`    | v-mode active                      |
| (2, 1)   | `[[0, 0]]`    | At rest                            |
| (2, 2)   | `[[1, 0]]`    | u-mode active                      |

In this example, the center point carries energy in both modes while four corner points each carry energy in a single mode — a pattern that could represent the initial displacement of a struck membrane.

---

## Configurations

Since the membrane has two displacement dimensions, each constituent can be in one of two configurations. A configuration determines which displacement mode the lattice point primarily uses to interact with its neighbors — i.e., which mode governs energy transfer.

The two configurations are complementary: a constituent in u-config cannot simultaneously be in v-config.

Using superscript notation:

| Configuration | Label    | Oriented mode | Notation example    |
|---------------|----------|---------------|---------------------|
| C₁            | u-conf   | u (index 0)   | `[[1, 0]]ᵘ`        |
| C₂            | v-conf   | v (index 1)   | `[[1, 0]]ᵛ`        |

Two constituents can share the same values but differ in configuration. For instance, `[[1, 1]]ᵘ` and `[[1, 1]]ᵛ` are bi-axially displaced but oriented toward different interaction modes. Their behavior during neighbor interactions will differ according to the defined rules.

### Physical Interpretation of Configurations

Configurations can be understood as specifying the *preferred coupling channel* for a lattice point:

- A u-config point transfers energy to neighbors primarily through the u-mode, even if it also carries v-mode displacement.
- A v-config point transfers energy to neighbors primarily through the v-mode.

This models anisotropic coupling — a realistic feature of membranes with directional stiffness or pre-stress.

---

## Interaction Rules

When two adjacent constituents interact, the outcome depends on both their values and their configurations. These rules are customizable; the following set is motivated by wave mechanics on coupled lattices.

### Rule Set

- **Same configuration, same values** — *Equilibrium.* No energy transfer. The two points are in identical states and have no driving force for interaction. This models a uniform region of the membrane at rest or in coherent motion.

- **Same values, config mismatch** — *Mode coupling.* The two points carry identical displacement but prefer different coupling channels. Energy is exchanged between modes: the u-config point may gain v-mode energy while the v-config point gains u-mode energy. This models the conversion between longitudinal and transverse wave energy at anisotropic coupling sites.

- **Same configuration, different values** — *Coherent propagation.* Energy flows from the more displaced point to the less displaced point along the shared mode. For example, `[[1, 0]]ᵘ` adjacent to `[[0, 0]]ᵘ` causes u-mode energy to propagate from the first to the second — analogous to a wavefront advancing through the lattice.

- **Different values, config mismatch** — *Scattering.* The interaction is complex: energy may partially propagate along each point's preferred mode while also scattering into the other mode. This models wave scattering at boundaries between regions of different anisotropy. Standard stochastic evolution applies unless overridden by a specific scattering rule.

### Interaction Outcome Table

| Values       | Configs      | Outcome            | Physical Analog              |
|--------------|--------------|--------------------|------------------------------|
| Same         | Same         | Equilibrium        | Uniform region               |
| Same         | Mismatch     | Mode coupling      | Polarization conversion      |
| Different    | Same         | Coherent propagation | Wavefront advance         |
| Different    | Mismatch     | Scattering         | Anisotropic scattering       |

---

## Boundary Conditions

The edges of the outer matrix define the membrane boundary. Boundary behavior significantly affects wave patterns and is configurable:

| Boundary Type   | Description                                                              |
|-----------------|--------------------------------------------------------------------------|
| Fixed           | Edge points cannot be created. Displacement energy at edges is absorbed (reflected as inversion). |
| Free            | Edge points can expand outward via creation rules. Energy reflects without inversion. |
| Periodic        | The field wraps around — opposite edges are treated as adjacent. No edge effects. |
| Absorbing       | Edge points in the `[[0, 0]]` state for consecutive steps are removed. Energy leaves the membrane. |

Boundary type is specified as part of the initial conditions.

---

## Initial Conditions

A membrane analysis begins with predefined initial conditions:

1. **Field dimensions** — The size of the outer matrix (e.g., 5×5, 10×10, 20×20).
2. **Boundary type** — Fixed, free, periodic, or absorbing.
3. **Initial constituent values** — The displacement pattern at step 0. Common seed patterns:
   - *Center strike* — A single `[[1, 1]]` at the center, all others `[[0, 0]]`. Models a point impact.
   - *Line excitation* — A row or column of `[[1, 0]]` or `[[0, 1]]`. Models a line source for directional wave propagation.
   - *Corner excitation* — Displacement at one corner. Models asymmetric wave launch.
   - *Random seed* — Random binary values across the membrane. Models thermal excitation.
4. **Initial configurations** — How constituents are assigned u-conf or v-conf:
   - *Uniform* — All set to u-conf (or v-conf). Isotropic propagation in one mode.
   - *Alternating* — Checkerboard pattern of u-conf and v-conf. Creates a regular mode-coupling lattice.
   - *Random* — Each point assigned randomly. Models a disordered anisotropic membrane.
5. **Evolution parameters** — Number of steps, whether configurations are reassigned between steps, etc.

### Example Initial Conditions

**Center strike, fixed boundary:**

```text
Field: 5×5
Boundary: fixed
Seed: center (2,2) set to [[1, 1]], all others [[0, 0]]
Configurations: alternating checkerboard
Steps: 50
```

**Line source, periodic boundary:**

```text
Field: 8×8
Boundary: periodic
Seed: row 3 all set to [[0, 1]], all others [[0, 0]]
Configurations: all set to v-conf
Steps: 100
```

---

## Stochastic Evolution

At each evolution step, every element of every constituent matrix is randomly assigned `0` or `1`. This stochastic process models thermal noise — random perturbations that exist in any real physical membrane.

The evolution loop:

```text
1. For each step S from 1 to N:
   a. For each constituent C in the field:
      i.   For each dimension D (u, v):
             - Generate random value V ∈ {0, 1}
             - Assign C[D] = V
             - Publish event VALUE_ASSIGNED(C, D, V, S)
   b. Evaluate interaction rules between adjacent constituents
      (respecting boundary conditions for neighbor identification)
   c. Evaluate creation/removal conditions
      (respecting boundary type for eligibility)
   d. Apply any event-driven actions
```

For a 5×5 membrane, each step generates 5 × 5 × 2 = 50 random binary values.

### Note on Stochastic vs. Deterministic Evolution

Pure stochastic evolution (uniform 50/50 random assignment) produces a thermally noisy membrane with no coherent wave structure. To observe wave-like behavior, the model should be extended with **biased stochastic generation** (see Extending the Model) where neighbor values influence the probability of assignment, simulating elastic coupling. The interaction rules then shape the emergent patterns on top of this coupled random process.

---

## Event System

The event system is identical to the framework standard:

| Event                 | Payload                              | Description                                  |
|-----------------------|--------------------------------------|----------------------------------------------|
| `VALUE_ASSIGNED`      | `(constituent, dimension, value, step)` | A displacement mode was assigned a value       |
| `CONSTITUENT_CREATED` | `(position, values, config, step)`   | A new lattice point was added (free boundary)  |
| `CONSTITUENT_REMOVED` | `(position, step)`                   | A lattice point was removed (absorbing boundary) |
| `INTERACTION`         | `(c1, c2, rule, step)`               | Two adjacent points interacted via a rule     |

### Membrane-Specific Subscribers

- **Wavefront tracker** — Subscribes to `VALUE_ASSIGNED`, identifies the leading edge of active displacement at each step, and plots wavefront position over time to measure propagation speed.
- **Mode energy monitor** — Counts constituents with `u = 1` and `v = 1` separately at each step, plotting the energy partition between modes over time. Mode coupling events should cause fluctuations in this ratio.
- **Interference mapper** — After each step, scans for regions where adjacent points have the same active mode, logging constructive interference clusters via `INTERACTION` events with the "coherent propagation" rule.
- **Standing wave detector** — Subscribes to `VALUE_ASSIGNED`, tracks positions that oscillate between active and rest with regular periodicity, flagging potential standing wave nodes and antinodes.

---

## Creation and Removal of Constituents

Boundary conditions govern whether the membrane can grow or shrink.

### Creation (Free Boundary)

New lattice points are created at the membrane edge when:

- **Edge excitation** — A boundary constituent with any active displacement (`[[1, *]]` or `[[*, 1]]`) has no neighbor in at least one outward direction. A new `[[0, 0]]` constituent is created in that direction with a randomly assigned configuration. This models a membrane that can physically expand under displacement.

- **Pressure expansion** — A boundary constituent in `[[1, 1]]` state for more than K consecutive steps causes new constituents to be created in all empty outward positions. This models sustained pressure at the edge driving membrane growth.

### Removal (Absorbing Boundary)

Lattice points are removed when:

- **Edge dissipation** — A boundary constituent has been `[[0, 0]]` for more than M consecutive steps. It is removed, shrinking the membrane. This models energy leaving through the boundary.

- **Interior collapse** — A non-boundary constituent has been `[[0, 0]]` for more than M steps *and* all its neighbors are also `[[0, 0]]`. It is removed, creating a hole in the membrane. This models structural failure under sustained absence of tension.

Fixed and periodic boundaries do not trigger creation or removal — the field size remains constant.

---

## Worked Example

### Setup

```text
Field: 3×3
Boundary: absorbing
Initial state:
  (0,0): [[0, 0]]ᵘ    (0,1): [[0, 0]]ᵛ    (0,2): [[0, 0]]ᵘ
  (1,0): [[0, 0]]ᵛ    (1,1): [[1, 1]]ᵘ    (1,2): [[0, 0]]ᵛ
  (2,0): [[0, 0]]ᵘ    (2,1): [[0, 0]]ᵛ    (2,2): [[0, 0]]ᵘ
Subscribed actions: edge dissipation (M = 2), interior collapse (M = 3)
```

The center point (1,1) carries bi-axial displacement in u-config. All edge points are at rest. Configurations alternate in a checkerboard pattern.

### Step 1 — Stochastic Assignment

```text
  (0,0): [[0, 1]]ᵘ    (0,1): [[1, 0]]ᵛ    (0,2): [[0, 0]]ᵘ
  (1,0): [[1, 0]]ᵛ    (1,1): [[1, 0]]ᵘ    (1,2): [[0, 1]]ᵛ
  (2,0): [[0, 0]]ᵘ    (2,1): [[1, 1]]ᵛ    (2,2): [[0, 1]]ᵘ
```

Events published: 18 `VALUE_ASSIGNED` events.

**Interaction check:**
- (1,1) `[[1, 0]]ᵘ` and (0,1) `[[1, 0]]ᵛ`: same values, config mismatch → *mode coupling*. Energy may transfer between u and v modes at these points.
- (1,1) `[[1, 0]]ᵘ` and (1,0) `[[1, 0]]ᵛ`: same values, config mismatch → *mode coupling*.
- (1,1) `[[1, 0]]ᵘ` and (1,2) `[[0, 1]]ᵛ`: different values, config mismatch → *scattering*.
- (1,1) `[[1, 0]]ᵘ` and (2,1) `[[1, 1]]ᵛ`: different values, config mismatch → *scattering*.

**Removal check:** No boundary point has been `[[0, 0]]` for consecutive steps yet (this is step 1). No removal.

### Step 2 — Stochastic Assignment

```text
  (0,0): [[0, 0]]ᵘ    (0,1): [[0, 0]]ᵛ    (0,2): [[0, 0]]ᵘ
  (1,0): [[0, 0]]ᵛ    (1,1): [[1, 1]]ᵘ    (1,2): [[0, 0]]ᵛ
  (2,0): [[0, 0]]ᵘ    (2,1): [[0, 0]]ᵛ    (2,2): [[0, 0]]ᵘ
```

**Interaction check:** Only (1,1) is active. Its neighbors are all `[[0, 0]]` — different values from (1,1)'s `[[1, 1]]`. Config mismatches with (0,1), (1,0), (1,2), (2,1) all yield *scattering*.

**Removal check:** All eight edge points are `[[0, 0]]` for the second consecutive step. The threshold M = 2 is met. All edge points are removed via edge dissipation.

Field after step 2:

```text
  (1,1): [[1, 1]]ᵘ
```

The membrane has collapsed to a single point — the energy at the center could not propagate fast enough to sustain the lattice under absorbing boundary conditions. This illustrates how boundary dissipation can dominate over stochastic re-excitation in small membranes.

---

## Physical Observations

The [Field Phenomena](./CONCEPTS.md#field-phenomena) section in CONCEPTS.md defines the physical concepts that emerge from the framework's mechanisms. This section describes how each phenomenon manifests specifically in the membrane model.

### Wave Propagation Speed

In a biased stochastic model (see Extensions), the rate at which activity spreads from a seed point depends on the coupling bias strength. Higher bias → faster apparent wave speed. The wavefront tracker subscriber can quantify this.

### Interference Patterns

When two or more seed points are active, their "wavefronts" (regions of high activity density) overlap. In regions where both u-mode and v-mode are active simultaneously, constructive interference occurs. Where one mode is suppressed, destructive interference is observed.

### Scattering

Scattering occurs when a propagating wavefront encounters a configuration boundary — a point where neighboring constituents differ in configuration. The mismatch in preferred coupling channels disperses the propagating activity rather than allowing it to continue coherently.

In a membrane with alternating (checkerboard) configurations, scattering is frequent and causes wavefronts to diffuse rapidly, producing blurred, short-lived patterns. In a membrane with uniform configurations, scattering is absent and wavefronts propagate more coherently with sharper edges.

### Mode Coupling Dynamics

With alternating configurations, mode coupling events should cause periodic oscillation in the u-energy/v-energy ratio. With uniform configurations, mode coupling is absent and each mode evolves independently.

### Damping

Without explicit damping, membrane activity is sustained only by stochastic generation, which is equally likely to create or destroy displacement. When damping is introduced (see Extending the Model), active points have a probability of returning to equilibrium each step, independent of their neighbors.

With damping active, the wavefront tracker will observe propagation fronts that diminish in amplitude with distance. The mode energy monitor will show declining total energy over time unless the propagation rate exceeds the damping rate.

### Boundary Reflection

Under fixed boundaries, activity reaching the edge should increase in the interior on subsequent steps (inversion reflection). Under free boundaries, activity should persist at the edge. Under absorbing boundaries, activity diminishes at the edge.

### Standing Waves

Standing waves emerge under fixed boundaries when propagation reflects off edges and interferes with itself. In the membrane, this manifests as positions that oscillate between displaced and equilibrium states with regular periodicity while neighboring positions remain relatively static.

Standing waves are more observable with biased stochastic generation (which produces coherent propagation) and in fields whose dimensions support integer half-wavelength patterns. The standing wave detector subscriber identifies these by tracking the temporal autocorrelation of individual positions.

### Resonance

In periodic boundary conditions with biased stochastic evolution, certain field sizes may produce standing wave patterns that persist longer than others — analogous to resonant frequencies of a physical membrane. The standing wave detector subscriber can identify these.

---

## Extending the Model

The basic membrane implementation can be extended in several directions:

- **Biased stochastic generation** — Instead of uniform 50/50, set the probability of `V = 1` based on the neighbor's value in the same dimension. For example, if a neighbor has `u = 1`, set `P(u = 1) = 0.7` for the current point. This simulates elastic coupling and produces coherent wave propagation instead of pure thermal noise.

- **Coupling strength parameter** — Introduce a parameter λ ∈ [0, 1] that controls the neighbor influence bias. λ = 0 is pure stochastic; λ = 1 is fully deterministic (a neighbor's value is always copied). Intermediate values produce varying degrees of wave coherence.

- **Configuration reassignment** — Allow configurations to change based on interaction outcomes. For example, after a mode coupling event, swap the configurations of the two interacting points. This models dynamic anisotropy.

- **Damping** — Introduce a probability that an active point becomes `[[0, 0]]` independent of its neighbors, simulating energy dissipation. Higher damping → shorter wave persistence.

- **Continuous-valued extension** — Relax the binary constraint to allow values in [0, 1] or a wider integer range, enabling smoother displacement representation at the cost of computational complexity.

- **Visualization** — Map u-mode to one color channel and v-mode to another, producing a two-color image per step. Stacking frames yields a video of membrane evolution, making wave patterns directly observable.