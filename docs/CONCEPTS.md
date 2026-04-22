# Gagan Framework

Gagan represents any field as a matrix of matrices. Each inner matrix — called a **constituent** — represents a single point in the field. This document describes the framework's core concepts, illustrated with concrete implementations for four-dimensional spacetime and a [two-dimensional elastic membrane](./2D-MEMBRANE.md).

---

## Field Representation

### Constituent Matrices

A field is an outer matrix whose elements are constituent matrices. Each constituent represents one discrete point in the modeled space (e.g., a point in spacetime, a point on a grid, etc.).

Every constituent matrix has the same number of elements as the number of dimensions in the field:

- A one-dimensional field with a single constituent: `[[1]]`
- A point in four-dimensional spacetime: `[[0, 1, 0, 0]]`

### Value Semantics

Each element in a constituent matrix holds a binary value — either `0` or `1`. The meaning of these values is predefined by the model. This binary constraint keeps the analysis simple and is well-suited to the underlying computational approach.

**Spacetime Implementation:**

The four dimensions are ordered as follows within each constituent matrix:

| Index | Symbol | Type     |
|-------|--------|----------|
| 0     | x      | Spatial  |
| 1     | y      | Spatial  |
| 2     | z      | Spatial  |
| 3     | t      | Temporal |

Value semantics for spacetime:

- **0** — The point is *absent* (inactive) along that dimension.
- **1** — The point is *present* (active) along that dimension.

For example, `[[0, 1, 0, 0]]` describes a point present along the y-axis but absent along x, z, and t. A point present along all dimensions is `[[1, 1, 1, 1]]`; a fully absent point is `[[0, 0, 0, 0]]`.

### Concrete Field Example

A spacetime field is a matrix of constituent matrices. Each constituent occupies a position in the outer matrix and represents one discrete point in spacetime.

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

### Complementary Orientations

A field with *n* dimensions has exactly *n* possible configurations. A configuration determines which dimension a constituent is *oriented* toward — i.e., which dimension primarily governs its interaction and evolution within the field.

The configurations are complementary: no two configurations activate the same dimension, and a constituent in one configuration cannot simultaneously be in another.

Using superscript notation to mark the active configuration, a two-dimensional field has:

- `[[0, 1]]¹` — oriented toward dimension 1
- `[[0, 1]]²` — oriented toward dimension 2

The values remain the same; only the configuration label changes.

**Spacetime Implementation:**

| Configuration | Label   | Oriented dimension | Notation example            |
|---------------|---------|--------------------|-----------------------------|
| C₁            | x-conf  | x (index 0)        | `[[0, 1, 0, 0]]ˣ`          |
| C₂            | y-conf  | y (index 1)        | `[[0, 1, 0, 0]]ʸ`          |
| C₃            | z-conf  | z (index 2)        | `[[0, 1, 0, 0]]ᶻ`          |
| C₄            | t-conf  | t (index 3)        | `[[0, 1, 0, 0]]ᵗ`          |

Two constituents can share the same values but differ in configuration. For instance, `[[1, 0, 0, 1]]ˣ` and `[[1, 0, 0, 1]]ᵗ` hold identical values but are oriented along different dimensions. Their interactions with neighboring constituents differ according to predefined rules.

### Interaction Rules

When two adjacent constituents interact, the outcome depends on both their values and their configurations. These rules are customizable.

**Spacetime Implementation — Example Rule Set:**

- **Same configuration, same values** — No interaction; the constituents are in equilibrium.
- **Same values, spatial–spatial config mismatch** (e.g., x-conf vs. z-conf) — The constituents exert a mutual spatial displacement influence; the field may propagate activity between the two configured dimensions.
- **Same values, spatial–temporal config mismatch** (e.g., x-conf vs. t-conf) — The spatially configured constituent is "frozen" for one evolution step while the temporally configured constituent advances, simulating a causal relationship.
- **Different values, any config mismatch** — No special interaction; standard stochastic evolution applies.

| Values       | Configs      | Outcome            | Spacetime Analog              |
|--------------|--------------|--------------------|-------------------------------|
| Same         | Same         | Equilibrium        | Identical spacetime points    |
| Same         | Mismatch     | Mode interaction   | Spatial displacement exchange |
| Different    | Same         | Coherent transfer  | Wavefront propagation         |
| Different    | Mismatch     | No special rule    | Standard stochastic evolution |

---

## Initial Conditions

Analysis begins with predefined initial conditions:

1. **Field dimensions** — The size of the outer matrix (e.g., 3×2, 5×5, 10×10).
2. **Boundary type** — How the field behaves at its edges (see [Boundary Conditions](#boundary-conditions)).
3. **Initial constituent values** — Either all zeros (empty field) or a specific seed pattern.
4. **Initial configurations** — How constituents are assigned configurations (e.g., uniformly at random, or all set to one configuration).
5. **Evolution parameters** — Number of steps, whether configurations are reassigned between steps, etc.

**Spacetime Implementation — Example Initial Condition:**

```text
Field: 3×3
Boundary: fixed
Seed: center constituent set to [[1, 1, 1, 1]], all others [[0, 0, 0, 0]]
Configurations: all set to t-conf
Steps: 100
```

---

## Boundary Conditions

The edges of the outer matrix define the field boundary. Boundary conditions govern how constituents behave at and beyond these edges, directly influencing interaction evaluation, creation and removal, and overall field dynamics. Boundary type is specified as part of the initial conditions.

### Boundary Types

| Type      | Description                                                                 |
|-----------|-----------------------------------------------------------------------------|
| Fixed     | The field size is constant. Edge constituents cannot be created. Activity at edges is reflected inward (with or without inversion, depending on the model). |
| Free      | The field can expand. Edge constituents with no outward neighbor may trigger creation rules. Activity reflects without inversion. |
| Periodic  | The field wraps around — opposite edges are treated as adjacent. No edge effects; the field has no true boundary. |
| Absorbing | The field can shrink. Dormant edge constituents are removed over time, simulating energy or activity leaving the field. |

### Effect on Other Framework Concepts

- **Interactions** — Under periodic boundaries, constituents at opposite edges are treated as neighbors. Under fixed or absorbing boundaries, edge constituents have fewer neighbors (or none in certain directions).
- **Creation** — Only free boundaries permit constituent creation at the edge. Fixed, periodic, and absorbing boundaries do not trigger expansion.
- **Removal** — Only absorbing boundaries permit constituent removal at the edge. Fixed, periodic, and free boundaries maintain edge integrity.

**Spacetime Implementation:**

| Boundary Type | Spacetime Analog                            | Behavior                                                                 |
|---------------|---------------------------------------------|--------------------------------------------------------------------------|
| Fixed         | Bounded region with reflecting walls        | Spacetime points at the edge cannot expand; activity reflects inward     |
| Free          | Expanding universe boundary                 | New spacetime points can be created at the edge via spatial expansion    |
| Periodic      | Closed universe topology (e.g., toroidal)   | Opposite edges are identified; no boundary effects                      |
| Absorbing     | Open universe boundary (causal horizon)     | Dormant edge points are removed, simulating activity crossing a horizon  |

**Spacetime — Boundary Interaction Example:**

In a 3×3 field with periodic boundaries, constituent (0, 0) has neighbors at (0, 1), (1, 0), (0, 2) [wrapped], and (2, 0) [wrapped]. All four interaction directions are valid, and no edge-specific rules apply. In the same field with fixed boundaries, (0, 0) would have only two neighbors — (0, 1) and (1, 0) — and interaction rules would only be evaluated for those pairs.

---

## Stochastic Evolution

At each evolution step, every element of every constituent matrix is randomly assigned `0` or `1`. This is the core stochastic process that drives the field forward.

The evolution loop:

```text
1. For each step S from 1 to N:
   a. For each constituent C in the field:
      i.   For each dimension D:
             - Generate random value V ∈ {0, 1}
             - Assign C[D] = V
             - Publish event VALUE_ASSIGNED(C, D, V, S)
   b. Evaluate interaction rules between adjacent constituents
      (respecting boundary conditions for neighbor identification)
   c. Evaluate creation/removal conditions
      (respecting boundary type for eligibility)
   d. Apply any event-driven actions
```

**Spacetime Implementation:**

For a 3×3 field, each step generates 3 × 3 × 4 = 36 random binary values — one per dimension per constituent.

### Note on Stochastic vs. Deterministic Evolution

Pure stochastic evolution (uniform 50/50 random assignment) produces a thermally noisy field with no coherent structure. To observe structured behavior (e.g., wave propagation, clustering), the model should be extended with **biased stochastic generation** where neighbor values influence the probability of assignment. The interaction rules then shape the emergent patterns on top of this coupled random process.

---

## Event System

Every value assignment produces an event that can be subscribed to. This event-driven architecture enables flexible, composable analysis without modifying the core evolution logic.

| Event                 | Payload                              | Description                                  |
|-----------------------|--------------------------------------|----------------------------------------------|
| `VALUE_ASSIGNED`      | `(constituent, dimension, value, step)` | A single dimension element was assigned a value |
| `CONSTITUENT_CREATED` | `(position, values, config, step)`   | A new constituent was added to the field      |
| `CONSTITUENT_REMOVED` | `(position, step)`                   | A constituent was removed from the field      |
| `INTERACTION`         | `(c1, c2, rule, step)`               | Two constituents interacted via a defined rule |

Subscribers attach handlers to these events to perform analysis. Examples:

- **Density tracker** — Subscribes to `VALUE_ASSIGNED`, counts how many constituents have `1` in a target dimension at each step, and plots density over time.
- **Cluster detector** — After each step, scans the field for contiguous groups of constituents where certain dimension values are all `1`, logging cluster sizes via `INTERACTION` events.
- **Causal chain observer** — Subscribes to `INTERACTION` events involving spatial–temporal config mismatches, recording the sequence of causal links.

---

## Creation and Removal of Constituents

Under predefined conditions, the field can grow or shrink. These operations are event-driven — triggered by subscribing to `VALUE_ASSIGNED` or `INTERACTION` events and evaluating the defined conditions. Eligibility for creation or removal is constrained by the [boundary type](#boundary-conditions).

### Creation

New constituents are inserted into the field when creation conditions are met. Only **free boundaries** permit edge creation.

**Spacetime Implementation:**

- **Spatial expansion** — A constituent with all spatial dimensions active (`[1, 1, 1, *]`) and in a spatial configuration has no neighbor in at least one adjacent position. A new `[[0, 0, 0, 0]]` constituent is created in the empty position with a randomly assigned configuration.
- **Temporal branching** — A constituent in t-conf with `t = 1` that has persisted for more than K consecutive steps causes a new constituent to be created at its position with the same values but a randomly chosen spatial configuration (simulating a temporal "fork" into spatial evolution).

### Removal

Constituents are removed from the field when removal conditions are met. Only **absorbing boundaries** permit edge removal.

**Spacetime Implementation:**

- **Vacuum collapse** — A constituent has been `[[0, 0, 0, 0]]` for more than M consecutive steps and has no active neighbors. It is removed, shrinking the field.
- **Causal isolation** — A constituent in a spatial configuration has been spatially inactive (`x = 0, y = 0, z = 0`) for more than L steps while all its neighbors are in temporal configurations. It is removed, simulating the disappearance of a spatially disconnected point.

Fixed and periodic boundaries do not trigger creation or removal — the field size remains constant throughout evolution.

---

## Field Phenomena

The preceding sections define the framework's *mechanisms* — how constituents are structured, how they interact, how boundaries behave, and how the field evolves stochastically. This section describes the *phenomena* that emerge from those mechanisms. These are physical concepts observable in any Gagan field, regardless of the specific domain being modeled.

### Coupling

Coupling is the mechanism by which adjacent constituents influence each other's evolution. It is the property that makes a field a connected system rather than a collection of independent points. Coupling can be characterized by its strength (how strongly neighbors influence each other), its directionality (isotropic vs. anisotropic), and its range (nearest-neighbor vs. longer-range).

In the base framework, coupling is realized through interaction rules — discrete events that fire when specific value and configuration conditions are met. Coupling can be extended with a strength parameter (see Extending the Model) that controls the probability of neighbor influence.

**Spacetime Implementation:**

- The spacetime model exhibits coupling through all four interaction rule outcomes. The "mode interaction" rule (same values, config mismatch between spatial configurations) is a form of isotropic spatial coupling — all spatial dimensions are treated symmetrically.
- The "causal interaction" rule (same values, spatial–temporal config mismatch) is a form of anisotropic coupling — the effect is asymmetric (one constituent is frozen while the other advances).
- Currently, coupling is binary (an interaction either fires or it does not). Introducing a coupling strength parameter λ would allow continuous tuning between no coupling (λ = 0, pure stochastic) and full coupling (λ = 1, deterministic neighbor copying).

### Propagation

Propagation is the spread of activity through the field from one constituent to its neighbors. It occurs when an interaction causes a neighboring constituent to adopt a state similar to the source. Propagation has a direction (outward from the source), a rate (how quickly activity spreads), and a mode (which dimension the activity spreads along).

Pure stochastic evolution produces no coherent propagation — activity appears randomly everywhere. Coherent propagation requires biased stochastic generation where neighbor values influence assignment probability.

**Spacetime Implementation:**

- Propagation in the spatial dimensions occurs when the "coherent transfer" rule fires: a spatially active constituent (e.g., `[[1, 0, 0, *]]` in x-conf) causes a neighbor with the same configuration to adopt similar spatial values. This represents spatial presence spreading across the field.
- Propagation in the temporal dimension occurs when a temporally configured constituent with `t = 1` influences neighboring temporal states through interaction rules. This represents causal influence spreading through the field.
- In the absence of biased generation, spatial propagation is weak — active spatial values appear and disappear stochastically rather than spreading coherently. The "coherent transfer" rule provides a mechanism for propagation, but stochastic reassignment competes with it at every step.

### Reflection

Reflection is how activity behaves when it reaches a field boundary. Reflection is governed by the boundary type and can occur with inversion (activity reverses upon reflection) or without inversion (activity preserves its state).

Reflection is a consequence of boundary conditions combined with propagation. It is only observable under fixed or periodic boundaries; free boundaries trigger expansion instead of reflection, and absorbing boundaries dissipate activity rather than reflecting it.

**Spacetime Implementation:**

- Under fixed boundaries, spatial activity reaching the edge has nowhere to propagate outward. Depending on the model, this activity may reflect inward with inversion (an active spatial value at the boundary causes the interior neighbor to become inactive on the next step) or without inversion (the interior neighbor retains the active state).
- Under periodic boundaries, activity does not reflect in the traditional sense — it passes through the boundary and emerges at the opposite edge, effectively "wrapping around" the field.
- Under free boundaries, spatial activity at the edge does not reflect but instead may trigger spatial expansion creation rules, growing the field outward.
- Under absorbing boundaries, activity at the edge diminishes. Rather than reflecting, active values dissipate and edge points are eventually removed.

### Interference

Interference occurs when two or more propagation fronts overlap in the field. It can be constructive (activity from multiple sources reinforces, producing higher activity density in the overlap region) or destructive (activity from multiple sources cancels or diminishes in the overlap region).

Interference is a universal field phenomenon that emerges from propagation + overlapping sources. It is distinct from clustering (which describes spatial grouping) — interference specifically describes the combination of propagating wavefronts.

**Spacetime Implementation:**

- Constructive interference in the spatial dimensions: When two spatially active regions propagate toward each other and overlap, the overlapping region may sustain spatial activity from both sources, producing a zone of higher spatial density than either source alone.
- Destructive interference in the spatial dimensions: When a propagation front meets a region where interaction rules suppress further activation (e.g., encountering a configuration boundary that triggers scattering instead of coherent transfer), the propagation front may dissipate at the overlap.
- In the temporal dimension, constructive interference manifests as regions where multiple causal chains converge, producing sustained temporal activity that persists longer than activity from a single chain.

### Scattering

Scattering occurs when propagation encounters a region where the interaction rules change — typically at a configuration boundary between adjacent constituents with different configurations. Scattering redirects, disperses, or partially converts propagating activity rather than allowing it to continue coherently.

Scattering is the counterpart to coherent propagation. When propagation encounters a region of uniform configuration, coherent transfer can occur; when it encounters a configuration mismatch, scattering occurs instead.

**Spacetime Implementation:**

- Scattering is realized in the spacetime model through the "different values, config mismatch" interaction outcome, which currently applies no special rule — standard stochastic evolution proceeds. This means propagating activity that encounters a configuration boundary is not systematically transferred but instead dissolves into random noise.
- A more refined scattering model could define specific scattering outcomes (e.g., partial propagation along one dimension, partial conversion to another dimension, partial dissipation). This would require extending the interaction rule set.
- In a field with random configurations, scattering is frequent and causes any propagating patterns to diffuse rather than maintain coherent structure. In a field with uniform configurations, scattering is absent and propagation can be more coherent.

### Damping

Damping is the tendency for activity in the field to decay over time, independent of stochastic generation. Without damping, the only way activity decreases is through stochastic reassignment (which is equally likely to increase activity) or through boundary removal. With damping, activity has a systematic bias toward decreasing.

Damping interacts with propagation: if propagation rate exceeds damping rate, activity can sustain and spread; if damping exceeds propagation, activity contracts and the field cools toward quiescence.

**Spacetime Implementation:**

- The spacetime model currently has no explicit damping mechanism. Activity decays only through the absorbing boundary (edge removal) and vacuum collapse rules, which are spatially localized rather than field-wide.
- Damping can be introduced as a per-step probability that any active constituent is reset to `[[0, 0, 0, 0]]`, independent of its neighbors or configuration. This models energy dissipation — a physical reality in any real system.
- With damping, the temporal density observation becomes more informative: a declining temporal density indicates that damping is overcoming stochastic generation, and the field is "cooling." A stable temporal density indicates a dynamic equilibrium between generation and damping.

### Standing Waves

Standing waves are stationary patterns that form when propagation fronts reflect off boundaries and interfere with themselves or with each other. Unlike traveling waves (which move through the field), standing waves persist at fixed positions — oscillating in place rather than propagating.

Standing waves are an emergent phenomenon that requires three conditions: propagation (to carry activity), reflection (to return it), and interference (to create stationary patterns). They are most observable under fixed or periodic boundaries.

**Spacetime Implementation:**

- In the spacetime model, standing wave behavior would manifest as fixed positions where spatial activity oscillates between active and inactive over multiple steps while neighboring positions remain relatively stable.
- Standing waves require both coherent propagation (to carry activity to boundaries and back) and boundary reflection (to return it). Under pure stochastic evolution, standing waves are unlikely because propagation is not coherent.
- In the temporal dimension, standing wave behavior would manifest as periodic activation and deactivation of the t-dimension at specific field positions, driven by causal interaction rules reflecting off boundaries.
- Detecting standing waves requires observing the temporal autocorrelation of individual positions — a task suited to the standing wave detector subscriber (see Analysis Patterns).

### Resonance

Resonance occurs when field dynamics self-reinforce at certain field sizes, geometries, or parameter values. Resonant configurations produce patterns that persist longer, are more stable, or have higher amplitude than non-resonant configurations.

Resonance is a higher-order emergent phenomenon — it arises from the interaction of propagation, reflection, interference, and standing waves. It is most observable when the field geometry supports patterns that reflect back to their origin in phase with themselves.

**Spacetime Implementation:**

- In the spacetime model, resonance may occur when field dimensions are such that propagation from a seed point reflects off boundaries and returns to the origin in phase with the next evolution step, reinforcing the active state. For example, in a field with side length L, a propagation pattern that takes L/2 steps to reach a boundary and L/2 steps to return would resonate if L is such that the return timing aligns with the stochastic generation cycle.
- Resonance is more likely under periodic boundaries, where propagation can circulate indefinitely without edge losses. Under absorbing boundaries, resonance is suppressed because energy leaves the field.
- Detecting resonance requires comparing the persistence of activity patterns across different field sizes or boundary conditions — if certain configurations produce significantly longer-lived patterns, resonance is likely occurring.

---

## Analysis Patterns

While the event system provides the mechanism for observing field evolution and the [field phenomena](#field-phenomena) section describes what emerges from those mechanisms, analysis patterns describe *how to observe* those phenomena. These are model-agnostic guidelines for extracting meaningful insights from any Gagan field simulation.

### Density and Distribution

Track how the concentration of active values (`1`s) changes across the field over time. Density patterns reveal whether activity is spreading, concentrating, or fluctuating randomly.

**Spacetime Implementation:**

- **Temporal density** — Count constituents with `t = 1` at each step. A rising temporal density may indicate increasing causal activity; a declining density may suggest the field is "cooling."
- **Spatial density** — Count constituents with any spatial dimension active (`x = 1` or `y = 1` or `z = 1`). Compare against temporal density to assess the spatial–temporal balance of the field.

### Clustering and Contiguity

Identify groups of adjacent constituents that share similar active states. Clustering patterns indicate local coherence in the field.

**Spacetime Implementation:**

- **Spatial clusters** — Contiguous groups of constituents where all spatial dimensions are active (`[1, 1, 1, *]`). Large spatial clusters may represent localized regions of high spatial presence — analogous to matter concentration in a cosmological context.
- **Temporal chains** — Sequences of constituents connected by spatial–temporal config mismatch interactions. Long chains may represent extended causal relationships across the field.

### Interaction Frequency

Monitor how often different interaction rules fire. Interaction frequency reveals the dominant dynamics in the field.

**Spacetime Implementation:**

- **Spatial–spatial interaction rate** — Frequency of mode interaction events between spatial configurations. High rates suggest active spatial coupling.
- **Spatial–temporal interaction rate** — Frequency of causal interaction events. Tracking this rate over time reveals whether causal relationships are intensifying or diminishing.
- **Equilibrium fraction** — Proportion of adjacent pairs in equilibrium (same config, same values) at each step. A rising equilibrium fraction indicates the field is settling into a stable state.

### Field Topology

Observe how the field's shape and size change over time. Topology patterns are directly influenced by boundary conditions.

**Spacetime Implementation:**

- **Under fixed boundaries** — The field size is constant. Topology analysis focuses on the distribution and connectivity of active regions within the fixed volume.
- **Under free boundaries** — Track the rate of edge creation events. Rapid expansion may indicate a "big bang"–like initial phase; slowing expansion may suggest the field is reaching a dynamic equilibrium with its boundary.
- **Under absorbing boundaries** — Track the rate of edge removal events. Rapid contraction may indicate activity is draining from the field faster than it is being generated stochastically.
- **Under periodic boundaries** — No size changes. Topology analysis can focus on whether activity wraps around the periodic edges, forming globally connected patterns.

### Configuration Distribution

Monitor how configurations are distributed across the field and whether this distribution changes over time.

**Spacetime Implementation:**

- **Configuration ratio** — Proportion of constituents in each of the four configurations (x-conf, y-conf, z-conf, t-conf) at each step. A uniform distribution suggests no preferred orientation; a skewed distribution may indicate emergent anisotropy.
- **Configuration clustering** — Whether constituents of the same configuration tend to group together. Spatial clustering of a single configuration may represent a region where one dimension dominates the dynamics.

---

## Worked Example

**Spacetime Implementation — Step-by-Step Walkthrough**

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

## Extending the Model

The spacetime implementation serves as a foundation. Directions for extension:

- **Weighted stochastic generation** — Instead of uniform 50/50, bias random assignment based on neighboring values (e.g., a constituent is more likely to adopt a neighbor's spatial value, simulating locality).
- **Coupling strength parameter** — Introduce a parameter λ ∈ [0, 1] that controls the neighbor influence bias. λ = 0 is pure stochastic; λ = 1 is fully deterministic (a neighbor's value is always copied). Intermediate values produce varying degrees of coherence.
- **Configuration reassignment** — Allow configurations to change between steps based on interaction outcomes, not just at initialization.
- **Damping** — Introduce a probability that an active constituent becomes `[[0, 0, ...]]` independent of its neighbors, simulating energy dissipation.
- **Larger fields and batching** — Scale to 100×100 or larger fields and batch-process evolution steps to study macro-scale pattern emergence.
- **Visualization** — Map each step's field state to a color-coded grid where spatial dimensions map to RGB channels and temporal presence maps to brightness, producing a frame sequence playable as video.
- **Continuous-valued extension** — Relax the binary constraint to allow values in [0, 1] or a wider integer range, enabling smoother representation at the cost of computational complexity.