# 4D Spacetime Implementation

This document describes a concrete application of the Gagan framework (see [CONCEPTS.md](./CONCEPTS.md) for the full framework definition) to model four-dimensional spacetime — a discrete field of points, each carrying presence information along three spatial axes and one temporal axis. The spacetime model illustrates fundamental field phenomena including propagation, reflection, interference, mode coupling, scattering, and vacuum collapse.

---

## Physical Background

A spacetime field models a region of four-dimensional spacetime discretized into a lattice of points. Each point carries binary presence information along three spatial dimensions (x, y, z) and one temporal dimension (t). The field evolves stochastically, with interaction rules governing how neighboring points influence each other and how spatial and temporal configurations interact.

Key physical phenomena observable in this model:

- **Spatial propagation** — Presence spreads outward from active regions through nearest-neighbor coupling.
- **Causal interaction** — Spatially and temporally configured points interact asymmetrically, simulating causal relationships.
- **Mode coupling** — Energy transfers between spatial dimensions at configuration boundaries.
- **Scattering** — Propagation disperses when encountering regions of different configuration.
- **Vacuum collapse** — Dormant regions with no active neighbors shrink under absorbing boundaries.
- **Boundary effects** — Fixed, free, periodic, or absorbing boundaries shape field evolution differently.

---

## Dimension Mapping

The four dimensions are ordered as follows within each constituent matrix:

| Index | Symbol | Type     | Description                     |
|-------|--------|----------|---------------------------------|
| 0     | x      | Spatial  | Presence along the x-axis       |
| 1     | y      | Spatial  | Presence along the y-axis       |
| 2     | z      | Spatial  | Presence along the z-axis       |
| 3     | t      | Temporal | Presence along the temporal axis |

The three spatial dimensions are interchangeable in an isotropic field. The temporal dimension is distinct — it governs causal interaction with spatial configurations.

---

## Value Semantics

Each element in a constituent matrix holds either `0` or `1`:

- **0** — The point is *absent* (inactive) along that dimension.
- **1** — The point is *present* (active) along that dimension.

The 16 possible states for a constituent:

| State                | Interpretation                            |
|----------------------|-------------------------------------------|
| `[[0, 0, 0, 0]]`     | Fully absent — vacuum                     |
| `[[1, 0, 0, 0]]`     | Present along x only                      |
| `[[0, 1, 0, 0]]`     | Present along y only                      |
| `[[0, 0, 1, 0]]`     | Present along z only                      |
| `[[0, 0, 0, 1]]`     | Present along t only (temporally active)  |
| `[[1, 1, 0, 0]]`     | Present along x and y                     |
| `[[1, 0, 0, 1]]`     | Present along x and t                     |
| `[[0, 0, 1, 1]]`     | Present along z and t                     |
| `[[1, 1, 1, 1]]`     | Fully present — all dimensions active     |
| ... (and 8 more)     | Various spatial–temporal combinations      |

This binary discretization approximates continuous presence: a point is either significantly present or absent along each dimension. A fully absent point `[[0, 0, 0, 0]]` represents spacetime vacuum.

---

## Field Representation

The spacetime field is represented as a matrix of constituent matrices. Each constituent occupies a position in the outer matrix and represents one discrete point in spacetime.

Example — a 3×2 field (six spacetime points):

```text
[
  [[0, 1, 0, 0], [1, 0, 0, 1], [0, 0, 1, 0]],
  [[1, 1, 0, 0], [0, 0, 0, 1], [1, 0, 1, 0]]
]
```

| Position     | Constituent      | Interpretation                        |
|--------------|------------------|---------------------------------------|
| (0, 0)       | `[[0, 1, 0, 0]]` | Present along y only                  |
| (0, 1)       | `[[1, 0, 0, 1]]` | Present along x and t                 |
| (0, 2)       | `[[0, 0, 1, 0]]` | Present along z only                  |
| (1, 0)       | `[[1, 1, 0, 0]]` | Present along x and y                 |
| (1, 1)       | `[[0, 0, 0, 1]]` | Present along t only                  |
| (1, 2)       | `[[1, 0, 1, 0]]` | Present along x and z                 |

---

## Configurations

Since the spacetime field has four dimensions, each constituent can be in one of four configurations. A configuration determines which dimension the point primarily uses to interact with its neighbors.

| Configuration | Label   | Oriented dimension | Notation example            |
|---------------|---------|--------------------|-----------------------------|
| C₁            | x-conf  | x (index 0)        | `[[0, 1, 0, 0]]ˣ`          |
| C₂            | y-conf  | y (index 1)        | `[[0, 1, 0, 0]]ʸ`          |
| C₃            | z-conf  | z (index 2)        | `[[0, 1, 0, 0]]ᶻ`          |
| C₄            | t-conf  | t (index 3)        | `[[0, 1, 0, 0]]ᵗ`          |

Two constituents can share the same values but differ in configuration. For instance, `[[1, 0, 0, 1]]ˣ` and `[[1, 0, 0, 1]]ᵗ` hold identical values but are oriented along different dimensions. Their behavior during neighbor interactions will differ according to the defined rules.

### Physical Interpretation of Configurations

Configurations can be understood as specifying the *preferred coupling channel* for a spacetime point:

- A spatially configured point (x-conf, y-conf, or z-conf) interacts with neighbors primarily through its oriented spatial dimension, simulating spatial locality and directional influence.
- A temporally configured point (t-conf) interacts through the temporal dimension, simulating causal advancement — the point "moves forward in time" rather than spreading spatially.

This distinction between spatial and temporal configurations is unique to the spacetime domain and enables modeling of causal structure within the field.

---

## Interaction Rules

When two adjacent constituents interact, the outcome depends on both their values and their configurations. These rules are customizable; the following set is motivated by relativistic field behavior.

### Rule Set

- **Same configuration, same values** — *Equilibrium.* No interaction; the constituents are in identical states with no driving force for change. This models a uniform region of spacetime.

- **Same values, spatial–spatial config mismatch** (e.g., x-conf vs. z-conf) — *Mode coupling.* The two points carry identical presence but prefer different spatial coupling channels. Energy is exchanged between the two configured spatial dimensions. This models spatial displacement transfer between orthogonal axes.

- **Same values, spatial–temporal config mismatch** (e.g., x-conf vs. t-conf) — *Causal interaction.* The spatially configured constituent is "frozen" for one evolution step while the temporally configured constituent advances, simulating a causal relationship where temporal evolution governs spatial change.

- **Different values, same configuration** — *Coherent propagation.* Presence flows from the more active point to the less active point along the shared configured dimension. This represents spatial or temporal presence spreading through the field.

- **Different values, config mismatch** — *No special rule.* Standard stochastic evolution applies. The mismatch prevents coherent propagation, causing propagating activity to dissipate into random noise at configuration boundaries.

**Note on relationship to the canonical framework:** The canonical interaction rules in [CONCEPTS.md](./CONCEPTS.md#interaction-rules) define four cases. This spacetime implementation refines the "same values, config mismatch" case into two sub-cases based on whether the mismatch is between two spatial configurations (mode coupling) or between a spatial and temporal configuration (causal interaction). The "different values, config mismatch" case maps to the canonical scattering outcome — here it is expressed as "no special rule" since no spacetime-specific scattering behavior is defined beyond default stochastic evolution.

### Interaction Outcome Table

| Values       | Configs                          | Outcome            | Spacetime Analog              |
|--------------|----------------------------------|--------------------|-------------------------------|
| Same         | Same                             | Equilibrium        | Uniform spacetime region      |
| Same         | Spatial–spatial mismatch         | Mode coupling      | Spatial displacement exchange |
| Same         | Spatial–temporal mismatch        | Causal interaction | Causal freeze/advance        |
| Different    | Same                             | Coherent propagation  | Wavefront propagation      |
| Different    | Mismatch                         | No special rule    | Stochastic evolution          |

---

## Boundary Conditions

The edges of the outer matrix define the field boundary. Boundary behavior significantly affects field dynamics and is configurable:

| Boundary Type | Spacetime Analog                            | Behavior                                                                 |
|---------------|---------------------------------------------|--------------------------------------------------------------------------|
| Fixed         | Bounded region with reflecting walls        | Edge points cannot expand; activity reflects inward                      |
| Free          | Expanding universe boundary                 | New spacetime points can be created at the edge via spatial expansion    |
| Periodic      | Closed universe topology (e.g., toroidal)   | Opposite edges are identified; no boundary effects                      |
| Absorbing     | Open universe boundary (causal horizon)     | Dormant edge points are removed, simulating activity crossing a horizon  |

### Boundary Interaction Example

In a 3×3 field with periodic boundaries, constituent (0, 0) has neighbors at (0, 1), (1, 0), (0, 2) [wrapped], and (2, 0) [wrapped]. All four interaction directions are valid, and no edge-specific rules apply. In the same field with fixed boundaries, (0, 0) would have only two neighbors — (0, 1) and (1, 0) — and interaction rules would only be evaluated for those pairs.

---

## Initial Conditions

A spacetime analysis begins with predefined initial conditions:

1. **Field dimensions** — The size of the outer matrix (e.g., 3×2, 5×5, 10×10).
2. **Boundary type** — Fixed, free, periodic, or absorbing.
3. **Initial constituent values** — The presence pattern at step 0. Common seed patterns:
   - *Center seed* — A single `[[1, 1, 1, 1]]` at the center, all others `[[0, 0, 0, 0]]`. Models a localized spacetime excitation.
   - *Temporal line* — A row or column of points with `t = 1`, all spatial dimensions zero. Models a causal chain initialization.
   - *Spatial plane* — All points with one spatial dimension active (e.g., `x = 1`), others zero. Models a planar wavefront.
   - *Random seed* — Random binary values across the field. Models quantum vacuum fluctuations.
4. **Initial configurations** — How constituents are assigned x-conf, y-conf, z-conf, or t-conf:
   - *Uniform* — All set to one configuration. Isotropic propagation in one mode.
   - *Alternating* — Regular pattern of spatial and temporal configurations. Creates a structured coupling lattice.
   - *Random* — Each point assigned randomly. Models a disordered spacetime with varying local anisotropy.
5. **Evolution parameters** — Number of steps, whether configurations are reassigned between steps, etc.

### Example Initial Conditions

**Center seed, fixed boundary:**

```text
Field: 3×3
Boundary: fixed
Seed: center constituent set to [[1, 1, 1, 1]], all others [[0, 0, 0, 0]]
Configurations: all set to t-conf
Steps: 100
```

**Temporal line, periodic boundary:**

```text
Field: 5×5
Boundary: periodic
Seed: center column all set to [[0, 0, 0, 1]], all others [[0, 0, 0, 0]]
Configurations: alternating spatial and temporal
Steps: 200
```

---

## Stochastic Evolution

At each evolution step, every element of every constituent matrix is randomly assigned `0` or `1`. This stochastic process models quantum fluctuations — random perturbations inherent in any discretized spacetime.

The evolution loop:

```text
1. For each step S from 1 to N:
   a. For each constituent C in the field:
      i.   For each dimension D (x, y, z, t):
             - Generate random value V ∈ {0, 1}
             - Assign C[D] = V
             - Publish event VALUE_ASSIGNED(C, D, V, S)
   b. Evaluate interaction rules between adjacent constituents
      (respecting boundary conditions for neighbor identification)
   c. Evaluate creation/removal conditions
      (respecting boundary type for eligibility)
   d. Apply any event-driven actions
```

For a 3×3 field, each step generates 3 × 3 × 4 = 36 random binary values.

### Note on Stochastic vs. Deterministic Evolution

Pure stochastic evolution (uniform 50/50 random assignment) produces a quantum-noisy spacetime with no coherent structure. To observe structured behavior (e.g., wave propagation, causal chains), the model should be extended with **biased stochastic generation** where neighbor values influence the probability of assignment. The interaction rules then shape the emergent patterns on top of this coupled random process.

---

## Event System

The event system is identical to the framework standard:

| Event                 | Payload                              | Description                                  |
|-----------------------|--------------------------------------|----------------------------------------------|
| `VALUE_ASSIGNED`      | `(constituent, dimension, value, step)` | A dimension was assigned a presence value       |
| `CONSTITUENT_CREATED` | `(position, values, config, step)`   | A new spacetime point was added (free boundary)  |
| `CONSTITUENT_REMOVED` | `(position, step)`                   | A spacetime point was removed (absorbing boundary) |
| `INTERACTION`         | `(c1, c2, rule, step)`               | Two points interacted via a rule             |

### Spacetime-Specific Subscribers

- **Spatial density tracker** — Counts constituents with any spatial dimension active (`x = 1` or `y = 1` or `z = 1`) at each step, plotting spatial presence over time. Compare against temporal density for spatial–temporal balance analysis.

- **Temporal density tracker** — Counts constituents with `t = 1` at each step. A rising temporal density may indicate increasing causal activity; a declining density may suggest the field is "cooling."

- **Spatial cluster detector** — Scans for contiguous groups of constituents where all spatial dimensions are active (`[1, 1, 1, *]`). Large spatial clusters may represent localized regions of high spatial presence — analogous to matter concentration in a cosmological context.

- **Causal chain observer** — Subscribes to `INTERACTION` events involving spatial–temporal config mismatches, recording the sequence of causal links. Long chains may represent extended causal relationships across the field.

- **Interaction frequency monitor** — Tracks how often each interaction rule fires separately for spatial–spatial, spatial–temporal, and temporal–temporal pairings, revealing the dominant dynamics at each step.

- **Vacuum collapse tracker** — Monitors removal events under absorbing boundaries, tracking the rate and location of spacetime point disappearance.

---

## Creation and Removal of Constituents

Boundary conditions govern whether the spacetime field can grow or shrink.

### Creation (Free Boundary)

New spacetime points are created when:

- **Spatial expansion** — A constituent with all spatial dimensions active (`[1, 1, 1, *]`) and in a spatial configuration has no neighbor in at least one adjacent position. A new `[[0, 0, 0, 0]]` constituent is created in the empty position with a randomly assigned configuration. This models a "big bang"–like spatial expansion at the field boundary.

- **Temporal branching** — A constituent in t-conf with `t = 1` that has persisted for more than K consecutive steps causes a new constituent to be created at its position with the same values but a randomly chosen spatial configuration (simulating a temporal "fork" into spatial evolution).

### Removal (Absorbing Boundary)

Spacetime points are removed when:

- **Vacuum collapse** — A constituent has been `[[0, 0, 0, 0]]` for more than M consecutive steps and has no active neighbors. It is removed, shrinking the field. This models spacetime vacuum disappearing when isolated from all active regions.

- **Causal isolation** — A constituent in a spatial configuration has been spatially inactive (`x = 0, y = 0, z = 0`) for more than L steps while all its neighbors are in temporal configurations. It is removed, simulating the disappearance of a spatially disconnected point.

Fixed and periodic boundaries do not trigger creation or removal — the field size remains constant.

---

## Worked Example

### Setup

```text
Field: 2×2
Boundary: absorbing
Initial state:
  (0,0): [[0, 0, 0, 0]]ᵗ
  (0,1): [[0, 0, 0, 0]]ᵗ
  (1,0): [[1, 1, 1, 1]]ᵗ
  (1,1): [[0, 0, 0, 0]]ᵗ
Subscribed actions: spatial expansion, vacuum collapse (M = 2)
```

All four positions are edge positions. Under absorbing boundaries, dormant edge points are eligible for removal after M consecutive steps.

### Step 1 — Stochastic Assignment

Random values are generated:

```text
  (0,0): [[1, 0, 0, 1]]ᵗ
  (0,1): [[0, 1, 0, 0]]ᵗ
  (1,0): [[1, 1, 0, 1]]ᵗ
  (1,1): [[0, 0, 1, 0]]ᵗ
```

Events published: 16 `VALUE_ASSIGNED` events.

**Interaction check:** (1,0) and (1,1) are adjacent. (1,0) has values `[1, 1, 0, 1]`, (1,1) has `[0, 0, 1, 0]` — different values, so no special interaction.

**Creation check:** (1,0) has spatial values `[1, 1, 0]` — not all spatially active, so no expansion triggered.

**Removal check:** No constituent is `[[0, 0, 0, 0]]`, so no removal.

### Step 2 — Stochastic Assignment

```text
  (0,0): [[0, 0, 0, 0]]ᵗ
  (0,1): [[0, 0, 0, 0]]ᵗ
  (1,0): [[1, 1, 1, 0]]ᵗ
  (1,1): [[0, 0, 0, 0]]ᵗ
```

**Creation check:** (1,0) has spatial values `[1, 1, 1]` but is in t-conf (not spatial). No expansion — the rule requires a spatial configuration.

**Removal check:** (0,0), (0,1), and (1,1) are all `[[0, 0, 0, 0]]`. This is their first consecutive step at zero, so the vacuum collapse threshold (M = 2) is not yet met.

### Step 3 — Stochastic Assignment

```text
  (0,0): [[0, 0, 0, 0]]ᵗ
  (0,1): [[0, 0, 0, 0]]ᵗ
  (1,0): [[1, 1, 1, 1]]ˣ   ← configuration changed to x-conf
  (1,1): [[0, 0, 0, 0]]ᵗ
```

**Creation check:** (1,0) has spatial values `[1, 1, 1]` and is now in x-conf (spatial). It has no neighbor at position (1,2) — the field boundary. However, the absorbing boundary type does not permit creation; only free boundaries allow edge expansion. No creation occurs.

**Removal check:** (0,0) and (0,1) have been `[[0, 0, 0, 0]]` for 2 consecutive steps. (0,0)'s only neighbor is (0,1), which is also all-zero — no active neighbor. (0,0) is removed via vacuum collapse. Similarly for (0,1) — its neighbor (0,0) is being removed, and its other neighbor (1,1) is all-zero — removed. (1,1) has also been `[[0, 0, 0, 0]]` for 2 steps; its only remaining neighbor is (1,0), which is active — so (1,1) is not removed (it has an active neighbor).

Field after step 3:

```text
  (1,0): [[1, 1, 1, 1]]ˣ
  (1,1): [[0, 0, 0, 0]]ᵗ
```

The field contracted from 2×2 to an effective 1×2 as vacant points collapsed under the absorbing boundary — illustrating how boundary conditions, stochastic generation, complementary configurations, and event-driven rules combine to produce dynamic field evolution.

---

## Physical Observations

The [Field Phenomena](./CONCEPTS.md#field-phenomena) section in CONCEPTS.md defines the physical concepts that emerge from the framework's mechanisms. This section describes how each phenomenon manifests specifically in the spacetime model.

### Coupling

The spacetime model exhibits coupling through all four interaction rule outcomes. The "mode coupling" rule (same values, spatial–spatial config mismatch) is a form of isotropic spatial coupling — all spatial dimensions are treated symmetrically. The "causal interaction" rule (same values, spatial–temporal config mismatch) is a form of anisotropic coupling — the effect is asymmetric (one constituent is frozen while the other advances).

Currently, coupling is binary (an interaction either fires or it does not). Introducing a coupling strength parameter λ would allow continuous tuning between no coupling (λ = 0, pure stochastic) and full coupling (λ = 1, deterministic neighbor copying).

### Propagation

Propagation in the spatial dimensions occurs when the "coherent propagation" rule fires: a spatially active constituent (e.g., `[[1, 0, 0, *]]` in x-conf) causes a neighbor with the same configuration to adopt similar spatial values. This represents spatial presence spreading across the field.

Propagation in the temporal dimension occurs when a temporally configured constituent with `t = 1` influences neighboring temporal states through interaction rules. This represents causal influence spreading through the field.

In the absence of biased generation, spatial propagation is weak — active spatial values appear and disappear stochastically rather than spreading coherently. The "coherent propagation" rule provides a mechanism for propagation, but stochastic reassignment competes with it at every step.

### Reflection

Under fixed boundaries, spatial activity reaching the edge has nowhere to propagate outward. Depending on the model, this activity may reflect inward with inversion (an active spatial value at the boundary causes the interior neighbor to become inactive on the next step) or without inversion (the interior neighbor retains the active state).

Under periodic boundaries, activity does not reflect in the traditional sense — it passes through the boundary and emerges at the opposite edge, effectively "wrapping around" the field.

Under free boundaries, spatial activity at the edge does not reflect but instead may trigger spatial expansion creation rules, growing the field outward.

Under absorbing boundaries, activity at the edge diminishes. Rather than reflecting, active values dissipate and edge points are eventually removed.

### Interference

Constructive interference in the spatial dimensions occurs when two spatially active regions propagate toward each other and overlap. The overlapping region may sustain spatial activity from both sources, producing a zone of higher spatial density than either source alone.

Destructive interference in the spatial dimensions occurs when a propagation front meets a region where interaction rules suppress further activation (e.g., encountering a configuration boundary that triggers scattering instead of coherent propagation). The propagation front may dissipate at the overlap.

In the temporal dimension, constructive interference manifests as regions where multiple causal chains converge, producing sustained temporal activity that persists longer than activity from a single chain.

### Scattering

Scattering is realized in the spacetime model through the "different values, config mismatch" interaction outcome, which currently applies no special rule — standard stochastic evolution proceeds. This means propagating activity that encounters a configuration boundary is not systematically transferred but instead dissolves into random noise.

A more refined scattering model could define specific scattering outcomes (e.g., partial propagation along one dimension, partial conversion to another dimension, partial dissipation). This would require extending the interaction rule set.

In a field with random configurations, scattering is frequent and causes any propagating patterns to diffuse rather than maintain coherent structure. In a field with uniform configurations, scattering is absent and propagation can be more coherent.

### Damping

The spacetime model currently has no explicit damping mechanism. Activity decays only through the absorbing boundary (edge removal) and vacuum collapse rules, which are spatially localized rather than field-wide.

Damping can be introduced as a per-step probability that any active constituent is reset to `[[0, 0, 0, 0]]`, independent of its neighbors or configuration. This models energy dissipation — a physical reality in any real system.

With damping, the temporal density observation becomes more informative: a declining temporal density indicates that damping is overcoming stochastic generation, and the field is "cooling." A stable temporal density indicates a dynamic equilibrium between generation and damping.

### Standing Waves

In the spacetime model, standing wave behavior would manifest as fixed positions where spatial activity oscillates between active and inactive over multiple steps while neighboring positions remain relatively stable.

Standing waves require both coherent propagation (to carry activity to boundaries and back) and boundary reflection (to return it). Under pure stochastic evolution, standing waves are unlikely because propagation is not coherent.

In the temporal dimension, standing wave behavior would manifest as periodic activation and deactivation of the t-dimension at specific field positions, driven by causal interaction rules reflecting off boundaries.

Detecting standing waves requires observing the temporal autocorrelation of individual positions — a task suited to the standing wave detector subscriber (see [Analysis Patterns](./CONCEPTS.md#analysis-patterns)).

### Resonance

In the spacetime model, resonance may occur when field dimensions are such that propagation from a seed point reflects off boundaries and returns to the origin in phase with the next evolution step, reinforcing the active state. For example, in a field with side length L, a propagation pattern that takes L/2 steps to reach a boundary and L/2 steps to return would resonate if L is such that the return timing aligns with the stochastic generation cycle.

Resonance is more likely under periodic boundaries, where propagation can circulate indefinitely without edge losses. Under absorbing boundaries, resonance is suppressed because energy leaves the field.

Detecting resonance requires comparing the persistence of activity patterns across different field sizes or boundary conditions — if certain configurations produce significantly longer-lived patterns, resonance is likely occurring.

---

## Extending the Model

The spacetime implementation serves as a foundation. Directions for extension:

- **Biased stochastic generation** — Instead of uniform 50/50, bias random assignment based on neighboring values (e.g., a constituent is more likely to adopt a neighbor's spatial value, simulating locality).
- **Coupling strength parameter** — Introduce a parameter λ ∈ [0, 1] that controls the neighbor influence bias. λ = 0 is pure stochastic; λ = 1 is fully deterministic (a neighbor's value is always copied). Intermediate values produce varying degrees of coherence.
- **Configuration reassignment** — Allow configurations to change between steps based on interaction outcomes, not just at initialization.
- **Damping** — Introduce a probability that an active constituent becomes `[[0, 0, ...]]` independent of its neighbors, simulating energy dissipation.
- **Larger fields and batching** — Scale to 100×100 or larger fields and batch-process evolution steps to study macro-scale pattern emergence.
- **Visualization** — Map each step's field state to a color-coded grid where spatial dimensions map to RGB channels and temporal presence maps to brightness, producing a frame sequence playable as video.
- **Continuous-valued extension** — Relax the binary constraint to allow values in [0, 1] or a wider integer range, enabling smoother representation at the cost of computational complexity.

---