# Gagan Framework

Gagan represents any field as a matrix of matrices. Each inner matrix — called a **constituent** — represents a single point in the field. This document defines the framework's core concepts: mechanisms, field phenomena, and analysis patterns. Concrete implementations illustrate how these concepts apply to specific physical domains:

- [1D Heat Rod](./1D-HEAT-ROD.md) — heat conduction along a one-dimensional rod
- [2D Elastic Membrane](./2D-MEMBRANE.md) — wave mechanics on a vibrating surface
- [3D Acoustics](./3D-ACOUSTICS.md) — elastic wave propagation in a three-dimensional solid
- [4D Spacetime](./4D-SPACETIME.md) — spatial presence and causal structure in discretized spacetime

---

## Field Representation

### Constituent Matrices

A field is an outer matrix whose elements are constituent matrices. Each constituent represents one discrete point in the modeled space.

Every constituent matrix has the same number of elements as the number of dimensions in the field:

- A one-dimensional field with a single constituent: `[[1]]`
- A point in a two-dimensional field: `[[1, 0]]`
- A point in a three-dimensional field: `[[0, 1, 0]]`
- A point in a four-dimensional field: `[[0, 1, 0, 0]]`

### Value Semantics

Each element in a constituent matrix holds a binary value — either `0` or `1`. The meaning of these values is predefined by the model. This binary constraint keeps the analysis simple and is well-suited to the underlying computational approach.

Value semantics are domain-specific. For example:
- In a heat rod, `0` = ambient temperature, `1` = thermally excited
- In a membrane, `0` = at equilibrium, `1` = displaced
- In acoustics, `0` = undisturbed medium, `1` = wave-active
- In spacetime, `0` = absent along that dimension, `1` = present

See the individual implementation documents for their value semantics definitions.

### Concrete Field Example

A field is a matrix of constituent matrices. Each constituent occupies a position in the outer matrix and represents one discrete point.

Example — a 3×3 two-dimensional field (nine points):

```text
[
  [[1, 0], [0, 0], [0, 1]],
  [[0, 0], [1, 1], [0, 0]],
  [[0, 1], [0, 0], [1, 0]]
]
```

| Position | Constituent   | Interpretation (membrane domain) |
|----------|---------------|----------------------------------|
| (0, 0)   | `[[1, 0]]`    | Displaced along first axis only  |
| (0, 1)   | `[[0, 0]]`    | At rest                          |
| (0, 2)   | `[[0, 1]]`    | Displaced along second axis only |
| (1, 0)   | `[[0, 0]]`    | At rest                          |
| (1, 1)   | `[[1, 1]]`    | Bi-axially displaced (center)    |
| (1, 2)   | `[[0, 0]]`    | At rest                          |
| (2, 0)   | `[[0, 1]]`    | Displaced along second axis only |
| (2, 1)   | `[[0, 0]]`    | At rest                          |
| (2, 2)   | `[[1, 0]]`    | Displaced along first axis only  |

See [2D-MEMBRANE.md](./2D-MEMBRANE.md#field-representation), [3D-ACOUSTICS.md](./3D-ACOUSTICS.md#field-representation), and [4D-SPACETIME.md](./4D-SPACETIME.md#field-representation) for additional field representation examples.

---

## Configurations

### Complementary Orientations

A field with *n* dimensions has exactly *n* possible configurations. A configuration determines which dimension a constituent is *oriented* toward — i.e., which dimension primarily governs its interaction and evolution within the field.

The configurations are complementary: no two configurations activate the same dimension, and a constituent in one configuration cannot simultaneously be in another.

Using superscript notation to mark the active configuration, a two-dimensional field has:

- `[[0, 1]]¹` — oriented toward dimension 1
- `[[0, 1]]²` — oriented toward dimension 2

The values remain the same; only the configuration label changes.

A one-dimensional field has only one configuration — all constituents share it, and configuration mismatches are impossible. A four-dimensional field has four configurations, enabling rich interaction dynamics. See the [Dimensionality Effects](#dimensionality-effects) section for a comparison.

See [2D-MEMBRANE.md](./2D-MEMBRANE.md#configurations), [3D-ACOUSTICS.md](./3D-ACOUSTICS.md#configurations), and [4D-SPACETIME.md](./4D-SPACETIME.md#configurations) for configuration tables with domain-specific interpretations.

### Interaction Rules

When two adjacent constituents interact, the outcome depends on both their values and their configurations. These rules are customizable per implementation.

The four canonical interaction cases:

| Values       | Configs      | Outcome            | Description                                          |
|--------------|--------------|--------------------|------------------------------------------------------|
| Same         | Same         | Equilibrium        | No interaction — identical states, no driving force  |
| Same         | Mismatch     | Mode coupling      | Energy transfer between configured dimensions        |
| Different    | Same         | Coherent propagation | Activity spreads from active to inactive neighbor   |
| Different    | Mismatch     | Scattering         | Activity disperses at configuration boundary         |

**Dimensionality note:** In a one-dimensional field, there is only one configuration, so "config mismatch" cases are impossible — mode coupling and scattering cannot occur. These phenomena require n ≥ 2.

See individual implementation documents for their specific rule sets:
- [1D-HEAT-ROD.md](./1D-HEAT-ROD.md#interaction-rules) — simplified rules (equilibrium + coherent propagation only)
- [2D-MEMBRANE.md](./2D-MEMBRANE.md#interaction-rules) — full rule set with mode coupling and scattering
- [3D-ACOUSTICS.md](./3D-ACOUSTICS.md#interaction-rules) — extended rule set with mode conversion and shear splitting
- [4D-SPACETIME.md](./4D-SPACETIME.md#interaction-rules) — extended rule set with spatial–temporal causal interaction

---

## Initial Conditions

Analysis begins with predefined initial conditions:

1. **Field dimensions** — The size of the outer matrix (e.g., 3×2, 5×5, 10×10).
2. **Boundary type** — How the field behaves at its edges (see [Boundary Conditions](#boundary-conditions)).
3. **Initial constituent values** — Either all zeros (empty field) or a specific seed pattern (e.g., center excitation, line source, random).
4. **Initial configurations** — How constituents are assigned configurations (e.g., uniformly at random, all set to one configuration, alternating pattern).
5. **Evolution parameters** — Number of steps, whether configurations are reassigned between steps, etc.

See individual implementation documents for example initial condition setups.

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

See individual implementation documents for boundary type interpretations in each physical domain:
- [1D-HEAT-ROD.md](./1D-HEAT-ROD.md#boundary-conditions) — insulated, expanding, ring, and open rod ends
- [2D-MEMBRANE.md](./2D-MEMBRANE.md#boundary-conditions) — membrane edge behaviors
- [3D-ACOUSTICS.md](./3D-ACOUSTICS.md#boundary-conditions) — solid medium boundary behaviors
- [4D-SPACETIME.md](./4D-SPACETIME.md#boundary-conditions) — bounded region, expanding universe, closed topology, causal horizon

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

### Note on Stochastic vs. Deterministic Evolution

Pure stochastic evolution (uniform 50/50 random assignment) produces a noisy field with no coherent structure. To observe structured behavior (e.g., wave propagation, clustering, diffusion), the model should be extended with **biased stochastic generation** where neighbor values influence the probability of assignment. The interaction rules then shape the emergent patterns on top of this coupled random process.

See [Extending the Model](#extending-the-model) for details on biased stochastic generation and other extensions.

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
- **Interaction frequency monitor** — Subscribes to `INTERACTION` events, counts how often each rule fires, and plots frequencies over time.

See individual implementation documents for domain-specific subscribers (e.g., wavefront tracker, thermal energy monitor, causal chain observer).

---

## Creation and Removal of Constituents

Under predefined conditions, the field can grow or shrink. These operations are event-driven — triggered by subscribing to `VALUE_ASSIGNED` or `INTERACTION` events and evaluating the defined conditions. Eligibility for creation or removal is constrained by the [boundary type](#boundary-conditions).

### Creation

New constituents are inserted into the field when creation conditions are met. Only **free boundaries** permit edge creation.

Creation rules are domain-specific. Common patterns:
- **Activity-driven expansion** — A boundary constituent with active values in certain dimensions triggers creation of new constituents in empty outward positions.
- **Sustained pressure expansion** — A boundary constituent that has been active for more than K consecutive steps causes creation in all empty outward positions.

See [1D-HEAT-ROD.md](./1D-HEAT-ROD.md#creation-free-boundary), [2D-MEMBRANE.md](./2D-MEMBRANE.md#creation-free-boundary), [3D-ACOUSTICS.md](./3D-ACOUSTICS.md#creation-free-boundary), and [4D-SPACETIME.md](./4D-SPACETIME.md#creation-free-boundary) for domain-specific creation rules.

### Removal

Constituents are removed from the field when removal conditions are met. Only **absorbing boundaries** permit edge removal.

Removal rules are domain-specific. Common patterns:
- **Edge dissipation** — A boundary constituent that has been fully inactive for more than M consecutive steps is removed, shrinking the field.
- **Interior collapse** — A non-boundary constituent that has been fully inactive for more than M steps *and* has no active neighbors is removed, creating a gap in the field.

Fixed and periodic boundaries do not trigger creation or removal — the field size remains constant throughout evolution.

See [1D-HEAT-ROD.md](./1D-HEAT-ROD.md#removal-absorbing-boundary), [2D-MEMBRANE.md](./2D-MEMBRANE.md#removal-absorbing-boundary), [3D-ACOUSTICS.md](./3D-ACOUSTICS.md#removal-absorbing-boundary), and [4D-SPACETIME.md](./4D-SPACETIME.md#removal-absorbing-boundary) for domain-specific removal rules.

---

## Field Phenomena

The preceding sections define the framework's *mechanisms* — how constituents are structured, how they interact, how boundaries behave, and how the field evolves stochastically. This section describes the *phenomena* that emerge from those mechanisms. These are physical concepts observable in any Gagan field, regardless of the specific domain being modeled.

### Coupling

Coupling is the mechanism by which adjacent constituents influence each other's evolution. It is the property that makes a field a connected system rather than a collection of independent points. Coupling can be characterized by its strength (how strongly neighbors influence each other), its directionality (isotropic vs. anisotropic), and its range (nearest-neighbor vs. longer-range).

In the base framework, coupling is realized through interaction rules — discrete events that fire when specific value and configuration conditions are met. Coupling can be extended with a strength parameter (see [Extending the Model](#extending-the-model)) that controls the probability of neighbor influence.

### Propagation

Propagation is the spread of activity through the field from one constituent to its neighbors. It occurs when an interaction causes a neighboring constituent to adopt a state similar to the source. Propagation has a direction (outward from the source), a rate (how quickly activity spreads), and a mode (which dimension the activity spreads along).

Pure stochastic evolution produces no coherent propagation — activity appears randomly everywhere. Coherent propagation requires biased stochastic generation where neighbor values influence assignment probability.

### Reflection

Reflection is how activity behaves when it reaches a field boundary. Reflection is governed by the boundary type and can occur with inversion (activity reverses upon reflection) or without inversion (activity preserves its state).

Reflection is a consequence of boundary conditions combined with propagation. It is only observable under fixed or periodic boundaries; free boundaries trigger expansion instead of reflection, and absorbing boundaries dissipate activity rather than reflecting it.

### Interference

Interference occurs when two or more propagation fronts overlap in the field. It can be constructive (activity from multiple sources reinforces, producing higher activity density in the overlap region) or destructive (activity from multiple sources cancels or diminishes in the overlap region).

Interference is a universal field phenomenon that emerges from propagation + overlapping sources. It is distinct from clustering (which describes spatial grouping) — interference specifically describes the combination of propagating wavefronts.

### Scattering

Scattering occurs when propagation encounters a region where the interaction rules change — typically at a configuration boundary between adjacent constituents with different configurations. Scattering redirects, disperses, or partially converts propagating activity rather than allowing it to continue coherently.

Scattering is the counterpart to coherent propagation. When propagation encounters a region of uniform configuration, coherent propagation can occur; when it encounters a configuration mismatch, scattering occurs instead.

Scattering requires n ≥ 2 — in a one-dimensional field with a single configuration, configuration boundaries do not exist.

### Damping

Damping is the tendency for activity in the field to decay over time, independent of stochastic generation. Without damping, the only way activity decreases is through stochastic reassignment (which is equally likely to increase activity) or through boundary removal. With damping, activity has a systematic bias toward decreasing.

Damping interacts with propagation: if propagation rate exceeds damping rate, activity can sustain and spread; if damping exceeds propagation, activity contracts and the field cools toward quiescence.

Damping is not present in the base framework — it must be introduced as an explicit extension (see [Extending the Model](#extending-the-model)).

### Standing Waves

Standing waves are stationary patterns that form when propagation fronts reflect off boundaries and interfere with themselves or with each other. Unlike traveling waves (which move through the field), standing waves persist at fixed positions — oscillating in place rather than propagating.

Standing waves are an emergent phenomenon that requires three conditions: propagation (to carry activity), reflection (to return it), and interference (to create stationary patterns). They are most observable under fixed or periodic boundaries.

Standing waves require coherent propagation, which in turn requires biased stochastic generation. Under pure stochastic evolution, standing waves are unlikely because propagation is not coherent.

### Resonance

Resonance occurs when field dynamics self-reinforce at certain field sizes, geometries, or parameter values. Resonant configurations produce patterns that persist longer, are more stable, or have higher amplitude than non-resonant configurations.

Resonance is a higher-order emergent phenomenon — it arises from the interaction of propagation, reflection, interference, and standing waves. It is most observable when the field geometry supports patterns that reflect back to their origin in phase with themselves.

Resonance is more likely under periodic boundaries, where propagation can circulate indefinitely without edge losses. Under absorbing boundaries, resonance is suppressed because energy leaves the field.

---

## Analysis Patterns

While the event system provides the mechanism for observing field evolution and the [field phenomena](#field-phenomena) section describes what emerges from those mechanisms, analysis patterns describe *how to observe* those phenomena. These are model-agnostic guidelines for extracting meaningful insights from any Gagan field simulation.

### Density and Distribution

Track how the concentration of active values (`1`s) changes across the field over time, both overall and per-dimension. Density patterns reveal whether activity is spreading, concentrating, or fluctuating randomly. In multi-dimensional fields, comparing per-dimension density identifies which modes are dominant.

### Clustering and Contiguity

Identify groups of adjacent constituents that share similar active states. Clustering patterns indicate local coherence in the field.

Scan for contiguous groups where certain dimension values are all `1`. Track cluster sizes, positions, and lifetimes over the evolution. Large, persistent clusters suggest stable structures; small, short-lived clusters suggest transient fluctuations.

### Interaction Frequency

Monitor how often different interaction rules fire. Interaction frequency reveals the dominant dynamics in the field.

Break down by rule type: equilibrium vs. mode coupling vs. coherent propagation vs. scattering. A high equilibrium fraction indicates the field is settling; a high coherent propagation rate indicates active spreading; a high scattering rate indicates frequent configuration boundary encounters.

### Field Topology

Observe how the field's shape and size change over time. Topology patterns are directly influenced by boundary conditions.

- **Under fixed boundaries** — The field size is constant. Topology analysis focuses on the distribution and connectivity of active regions within the fixed volume.
- **Under free boundaries** — Track the rate of edge creation events to measure expansion speed.
- **Under absorbing boundaries** — Track the rate of edge removal events to measure contraction speed.
- **Under periodic boundaries** — No size changes. Topology analysis can focus on whether activity wraps around the periodic edges, forming globally connected patterns.

### Configuration Distribution

Monitor how configurations are distributed across the field and whether this distribution changes over time. This pattern is only applicable to fields with n ≥ 2 dimensions.

- **Configuration ratio** — Proportion of constituents in each configuration at each step. A uniform distribution suggests no preferred orientation; a skewed distribution may indicate emergent anisotropy.
- **Configuration clustering** — Whether constituents of the same configuration tend to group together. Spatial clustering of a single configuration may represent a region where one dimension dominates the dynamics.

### Dimensionality Effects

The available phenomena and analysis patterns depend on the number of dimensions:

| Dimensionality | Configurations | Mode Coupling | Scattering | Applicable Analysis Patterns                  |
|----------------|----------------|---------------|------------|-----------------------------------------------|
| 1D             | 1 (no choice)  | Impossible    | Impossible | Density, Clustering, Interaction Freq, Topology |
| 2D             | 2              | Possible      | Possible   | All patterns                                   |
| 3D+            | 3+             | Rich          | Rich       | All patterns + Configuration Distribution      |

See individual implementation documents for physical observations mapping these phenomena to specific domains:
- [1D-HEAT-ROD.md — Physical Observations](./1D-HEAT-ROD.md#physical-observations)
- [2D-MEMBRANE.md — Physical Observations](./2D-MEMBRANE.md#physical-observations)
- [3D-ACOUSTICS.md — Physical Observations](./3D-ACOUSTICS.md#physical-observations)
- [4D-SPACETIME.md — Physical Observations](./4D-SPACETIME.md#physical-observations)

---

## Worked Examples

Step-by-step walkthroughs demonstrating stochastic assignment, interaction evaluation, and creation/removal checks with concrete values are provided in each implementation document:

- [1D-HEAT-ROD.md — Worked Example](./1D-HEAT-ROD.md#worked-example) — a 5-point rod under absorbing boundary showing thermal diffusion and endpoint removal over 3 steps.
- [2D-MEMBRANE.md — Worked Example](./2D-MEMBRANE.md#worked-example) — a 3×3 membrane under absorbing boundary showing mode coupling and edge dissipation over 3 steps.
- [3D-ACOUSTICS.md — Worked Example](./3D-ACOUSTICS.md#worked-example) — a 3×3 acoustic field under absorbing boundary showing scattering at configuration boundaries and medium collapse over 3 steps.
- [4D-SPACETIME.md — Worked Example](./4D-SPACETIME.md#worked-example) — a 2×2 spacetime field under absorbing boundary showing vacuum collapse over 3 steps.

---

## Extending the Model

The base framework provides a foundation. The following extension patterns apply across all implementations:

- **Biased stochastic generation** — Instead of uniform 50/50, bias random assignment based on neighboring values (e.g., a constituent is more likely to adopt a neighbor's value in the same dimension). This is essential for producing coherent propagation and wave-like behavior. Without it, the field remains noisy.
- **Coupling strength parameter** — Introduce a parameter λ ∈ [0, 1] that controls the neighbor influence bias. λ = 0 is pure stochastic; λ = 1 is fully deterministic (a neighbor's value is always copied). Intermediate values produce varying degrees of coherence.
- **Damping** — Introduce a probability that any active constituent is reset to all-zeros, independent of its neighbors or configuration. This models energy dissipation. The relationship between coupling strength and damping determines whether the field sustains activity or decays.
- **Configuration reassignment** — Allow configurations to change between steps based on interaction outcomes, not just at initialization. This enables dynamic anisotropy.
- **Continuous-valued extension** — Relax the binary constraint to allow values in [0, 1] or a wider integer range, enabling smoother representation at the cost of computational complexity.
- **Visualization** — Map each step's field state to a visual representation (color-coded grids, 1D bars, etc.) to produce frame sequences that make emergent patterns directly observable.
- **Larger fields and batching** — Scale to 100×100 or larger fields and batch-process evolution steps to study macro-scale pattern emergence.

See individual implementation documents for domain-specific extensions (e.g., heat sources/sinks, variable conductivity, temporal branching).