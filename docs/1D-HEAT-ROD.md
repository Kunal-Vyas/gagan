# 1D Heat Rod Implementation

This document describes a concrete application of the Gagan framework (see [CONCEPTS.md](./CONCEPTS.md) for the full framework definition) to model heat conduction along a one-dimensional rod — a discrete chain of thermal lattice points, each capable of carrying thermal energy. The heat rod model illustrates fundamental transport phenomena including diffusion, boundary reflection, thermal equilibrium, and damping.

---

## Physical Background

A heat-conducting rod is a one-dimensional object through which thermal energy propagates from hotter regions to cooler regions. When discretized into a chain of lattice points connected by thermal coupling, the rod becomes a computationally tractable system. Each lattice point carries a binary thermal state — either at ambient temperature or thermally excited — and energy transfers between neighbors through nearest-neighbor coupling.

Key physical phenomena observable in this model:

- **Thermal diffusion** — Heat energy spreads from hot regions to cold regions through nearest-neighbor coupling, equalizing temperature over time.
- **Boundary reflection** — Heat reaching an insulated end reflects back into the rod; heat reaching an open end dissipates.
- **Thermal equilibrium** — A uniform temperature state where no net heat flows between any adjacent points.
- **Damping** — Heat energy dissipates over time due to environmental losses, causing the rod to cool toward ambient temperature.
- **Standing thermal waves** — Under periodic boundaries, thermal oscillations can form stationary patterns at certain rod lengths.

---

## Dimension Mapping

The single dimension is defined as follows within each constituent matrix:

| Index | Symbol | Type   | Description                     |
|-------|--------|--------|---------------------------------|
| 0     | θ      | Thermal | Thermal state of the lattice point |

This single dimension represents whether the lattice point carries thermal energy above ambient. Unlike multi-dimensional models where different dimensions can interact through mode coupling, the 1D heat rod has only one mode of energy transport.

---

## Value Semantics

Each element in a constituent matrix holds either `0` or `1`:

- **0** — The lattice point is at *ambient temperature* (no thermal energy above equilibrium).
- **1** — The lattice point is *thermally excited* (carries thermal energy).

The two possible states for a constituent:

| State      | Interpretation                    |
|------------|-----------------------------------|
| `[[0]]`    | At ambient temperature (cold)     |
| `[[1]]`    | Thermally excited (hot)           |

This binary discretization approximates continuous temperature: a point is either significantly above ambient or it is not. While coarse, this abstraction captures the qualitative behavior of heat diffusion and is computationally efficient. A "hot" point represents a region where temperature exceeds some threshold; a "cold" point represents a region at or below that threshold.

---

## Field Representation

The heat rod is represented as a one-dimensional array (row vector) of constituent matrices. Each constituent occupies a position in the array and represents one lattice point along the rod.

Example — a rod with 7 lattice points:

```text
[[[0], [1], [1], [0], [0], [1], [0]]]
```

| Position | Constituent | Interpretation      |
|----------|-------------|---------------------|
| 0        | `[[0]]`     | Cold                |
| 1        | `[[1]]`     | Hot                 |
| 2        | `[[1]]`     | Hot                 |
| 3        | `[[0]]`     | Cold                |
| 4        | `[[0]]`     | Cold                |
| 5        | `[[1]]`     | Hot                 |
| 6        | `[[0]]`     | Cold                |

In this example, three lattice points are thermally excited while four are at ambient. The two adjacent hot points at positions 1 and 2 form a contiguous hot region — a thermal cluster that will tend to diffuse outward through coupling with its cold neighbors.

---

## Configurations

Since the heat rod has only one dimension, each constituent can only be in one configuration. A configuration determines which dimension the lattice point is oriented toward — in this case, there is only the thermal dimension.

Using superscript notation:

| Configuration | Label       | Oriented dimension | Notation example |
|---------------|-------------|--------------------|------------------|
| C₁            | θ-conf      | θ (index 0)        | `[[1]]^θ`        |

All constituents in the field share the same configuration. This is a fundamental simplification of the 1D case compared to multi-dimensional models: there are no configuration mismatches, and therefore no mode coupling or scattering phenomena. The interaction dynamics are determined entirely by value differences between neighbors.

### Physical Interpretation of the Single Configuration

The absence of configuration variety means the rod is isotropic with respect to its single transport mode. Every lattice point interacts with its neighbors through the same thermal coupling channel. This models a homogeneous rod with uniform thermal conductivity — a reasonable approximation for many physical systems.

To introduce anisotropy in a 1D system, one could extend the model to two dimensions (e.g., adding a "material type" dimension that distinguishes between rod segments with different conductivities). The [2D membrane implementation](./2D-MEMBRANE.md) demonstrates how configuration variety enables mode coupling and scattering.

---

## Interaction Rules

When two adjacent constituents interact, the outcome depends on their values. Since all constituents share the same configuration, the interaction rule set is simplified compared to multi-dimensional models.

### Rule Set

- **Same values** — *Equilibrium.* No heat transfer. Two adjacent points at the same thermal state have no temperature gradient between them and therefore no driving force for heat flow. This models a uniform region of the rod — either uniformly cold or uniformly hot.

- **Different values** — *Coherent propagation.* Heat flows from the hot point to the cold point. The cold point becomes more likely to adopt the hot state on the next stochastic assignment, simulating thermal diffusion down a temperature gradient.

### Interaction Outcome Table

| Values       | Configs      | Outcome            | Physical Analog              |
|--------------|--------------|--------------------|------------------------------|
| Same         | Same         | Equilibrium        | No temperature gradient      |
| Different    | Same         | Coherent propagation | Heat diffusion             |
| Same         | Mismatch     | N/A                | Not possible in 1D           |
| Different    | Mismatch     | N/A                | Not possible in 1D           |

Note that the "same values, config mismatch" and "different values, config mismatch" cases are impossible in a 1D field with a single configuration. This eliminates mode coupling and scattering — phenomena that require configuration variety in multi-dimensional fields.

---

## Boundary Conditions

The endpoints of the array define the rod boundaries. Boundary behavior significantly affects thermal dynamics and is configurable:

| Boundary Type   | Description                                                              |
|-----------------|--------------------------------------------------------------------------|
| Fixed           | Endpoints cannot be created. Thermal energy at endpoints reflects inward (insulated ends). |
| Free            | Endpoints can expand outward via creation rules. Energy reflects without dissipation. |
| Periodic        | The rod wraps around — opposite endpoints are treated as adjacent, forming a ring. No endpoint effects. |
| Absorbing       | Endpoints in the `[[0]]` state for consecutive steps are removed. Heat energy leaves the rod. |

Boundary type is specified as part of the initial conditions.

### Physical Interpretation of Boundary Types

- **Fixed (insulated)** — The rod ends are thermally insulated. No heat escapes; all energy remains in the system. This models a rod with adiabatic boundary conditions. Over time, the rod approaches uniform temperature (all hot or all cold, depending on initial conditions and stochastic generation).

- **Free** — The rod can physically extend. Thermal energy at the endpoints may drive expansion, modeling a rod that grows under thermal stress.

- **Periodic** — The rod forms a closed loop (a thermal ring). Heat can circulate indefinitely without endpoint losses. This topology supports standing thermal waves at certain ring circumferences.

- **Absorbing (open)** — The rod ends are exposed to an ambient environment. Heat dissipates through the endpoints, causing the rod to cool over time. This models a rod with Dirichlet boundary conditions (fixed temperature at endpoints equal to ambient).

---

## Initial Conditions

A heat rod analysis begins with predefined initial conditions:

1. **Rod length** — The number of lattice points (e.g., 10, 20, 50).
2. **Boundary type** — Fixed, free, periodic, or absorbing.
3. **Initial constituent values** — The thermal pattern at step 0. Common seed patterns:
   - *Center hot spot* — A single `[[1]]` at the center, all others `[[0]]`. Models a localized heat pulse that diffuses symmetrically.
   - *Left-end heating* — The leftmost N points set to `[[1]]`, all others `[[0]]`. Models one end of the rod in contact with a heat source.
   - *Alternating* — Alternating `[[1]]` and `[[0]]` along the rod. Models a high-frequency thermal oscillation that will smooth out through diffusion.
   - *Random seed* — Random binary values along the rod. Models thermal noise or a randomly excited initial state.
4. **Configurations** — All constituents are set to θ-conf (the only option in 1D).
5. **Evolution parameters** — Number of steps, coupling strength, damping rate, etc.

### Example Initial Conditions

**Center hot spot, fixed boundary:**

```text
Rod length: 11
Boundary: fixed
Seed: position 5 set to [[1]], all others [[0]]
Configurations: all θ-conf
Steps: 50
```

**Left-end heating, absorbing boundary:**

```text
Rod length: 15
Boundary: absorbing
Seed: positions 0-3 set to [[1]], all others [[0]]
Configurations: all θ-conf
Steps: 100
```

**Alternating pattern, periodic boundary:**

```text
Rod length: 20
Boundary: periodic
Seed: alternating [[1]], [[0]], [[1]], [[0]], ...
Configurations: all θ-conf
Steps: 40
```

---

## Stochastic Evolution

At each evolution step, every element of every constituent matrix is randomly assigned `0` or `1`. This stochastic process models thermal fluctuations — random perturbations that exist in any real physical system due to microscopic degrees of freedom.

The evolution loop:

```text
1. For each step S from 1 to N:
   a. For each constituent C in the rod:
      i.   Generate random value V ∈ {0, 1}
      ii.  Assign C[θ] = V
      iii. Publish event VALUE_ASSIGNED(C, θ, V, S)
   b. Evaluate interaction rules between adjacent constituents
      (respecting boundary conditions for neighbor identification)
   c. Evaluate creation/removal conditions
      (respecting boundary type for eligibility)
   d. Apply any event-driven actions
```

For a rod of length L, each step generates L random binary values.

### Note on Stochastic vs. Deterministic Evolution

Pure stochastic evolution (uniform 50/50 random assignment) produces a thermally noisy rod with no coherent heat diffusion. To observe diffusion-like behavior, the model should be extended with **biased stochastic generation** (see Extending the Model) where neighbor values influence the probability of assignment, simulating thermal coupling. The interaction rules then shape the emergent patterns on top of this coupled random process.

With biased generation, a cold point adjacent to a hot point has a higher probability of becoming hot on the next step — modeling heat flow down a temperature gradient. Without bias, hot and cold states appear randomly regardless of neighbors.

---

## Event System

The event system is identical to the framework standard:

| Event                 | Payload                              | Description                                  |
|-----------------------|--------------------------------------|----------------------------------------------|
| `VALUE_ASSIGNED`      | `(constituent, dimension, value, step)` | A thermal state was assigned a value           |
| `CONSTITUENT_CREATED` | `(position, values, config, step)`   | A new lattice point was added (free boundary)  |
| `CONSTITUENT_REMOVED` | `(position, step)`                   | A lattice point was removed (absorbing boundary) |
| `INTERACTION`         | `(c1, c2, rule, step)`               | Two adjacent points interacted via a rule     |

### Heat Rod-Specific Subscribers

- **Temperature profile tracker** — Subscribes to `VALUE_ASSIGNED`, records the thermal state of every lattice point at each step, and plots the temperature profile along the rod over time. This produces a space-time diagram showing how the hot region spreads and evolves.

- **Thermal energy monitor** — Counts constituents with `θ = 1` at each step, plotting total thermal energy over time. Under absorbing boundaries, energy should decline; under fixed boundaries, energy is conserved (on average, modulo stochastic fluctuations).

- **Diffusion front tracker** — Identifies the leading edge of the hot region at each step — the position of the leftmost and rightmost `[[1]]` — and plots front position over time. The slope of this curve gives the apparent diffusion speed.

- **Equilibrium detector** — Monitors the fraction of adjacent pairs in equilibrium (same values) at each step. A rising equilibrium fraction indicates the rod is approaching thermal uniformity.

- **Hot cluster analyzer** — Identifies contiguous groups of `[[1]]` constituents and tracks their size and position over time. Multiple clusters indicate incomplete diffusion; a single large cluster indicates merging of thermal regions.

---

## Creation and Removal of Constituents

Boundary conditions govern whether the rod can grow or shrink.

### Creation (Free Boundary)

New lattice points are created at the rod endpoints when:

- **Thermal expansion** — An endpoint constituent with `[[1]]` (hot) has no neighbor in the outward direction. A new `[[0]]` constituent is created in that direction with θ-conf. This models a rod that physically extends under thermal stress.

- **Sustained pressure expansion** — An endpoint constituent has been `[[1]]` for more than K consecutive steps, causing new constituents to be created in the outward position. This models sustained heating at the endpoint driving rod growth.

### Removal (Absorbing Boundary)

Lattice points are removed when:

- **Endpoint dissipation** — An endpoint constituent has been `[[0]]` for more than M consecutive steps. It is removed, shortening the rod. This models material loss at an exposed end where thermal energy has dissipated.

- **Interior collapse** — A non-endpoint constituent has been `[[0]]` for more than M steps *and* all its neighbors are also `[[0]]`. It is removed, creating a gap in the rod. This models structural failure under sustained absence of thermal energy (e.g., brittle fracture in a cooled material).

Fixed and periodic boundaries do not trigger creation or removal — the rod length remains constant.

---

## Worked Example

### Setup

```text
Rod length: 5
Boundary: absorbing
Initial state:
  (0): [[0]]^θ    (1): [[0]]^θ    (2): [[1]]^θ    (3): [[0]]^θ    (4): [[0]]^θ
Subscribed actions: endpoint dissipation (M = 2), interior collapse (M = 3)
```

The center point (2) carries thermal energy. All other points are at ambient temperature. Positions 0 and 4 are endpoints eligible for removal under the absorbing boundary.

### Step 1 — Stochastic Assignment

```text
  (0): [[1]]^θ    (1): [[0]]^θ    (2): [[1]]^θ    (3): [[1]]^θ    (4): [[0]]^θ
```

Events published: 5 `VALUE_ASSIGNED` events.

**Interaction check:**
- (0) `[[1]]` and (1) `[[0]]`: different values, same config → *coherent propagation*. Heat flows from position 0 toward position 1.
- (1) `[[0]]` and (2) `[[1]]`: different values, same config → *coherent propagation*. Heat flows from position 2 toward position 1.
- (2) `[[1]]` and (3) `[[1]]`: same values, same config → *equilibrium*. No heat flow.
- (3) `[[1]]` and (4) `[[0]]`: different values, same config → *coherent propagation*. Heat flows from position 3 toward position 4.

Three coherent propagation events fired. The thermal energy at the center has begun to diffuse outward, and the stochastic assignment happened to place hot points at positions 0 and 3 as well.

**Removal check:** Endpoints (0) and (4) are `[[1]]` and `[[0]]` respectively — neither has been `[[0]]` for consecutive steps yet. No removal.

### Step 2 — Stochastic Assignment

```text
  (0): [[0]]^θ    (1): [[1]]^θ    (2): [[0]]^θ    (3): [[0]]^θ    (4): [[0]]^θ
```

**Interaction check:**
- (0) `[[0]]` and (1) `[[1]]`: different values → *coherent propagation*. Heat flows toward position 0.
- (1) `[[1]]` and (2) `[[0]]`: different values → *coherent propagation*. Heat flows toward position 2.
- (2) `[[0]]` and (3) `[[0]]`: same values → *equilibrium*. No heat flow.
- (3) `[[0]]` and (4) `[[0]]`: same values → *equilibrium*. No heat flow.

The hot region has shifted to position 1 only. The previous thermal energy at positions 0, 2, and 3 was lost to stochastic reassignment.

**Removal check:** Endpoint (0) has been `[[0]]` for 1 consecutive step. Endpoint (4) has been `[[0]]` for 2 consecutive steps. The threshold M = 2 is met for (4). Position (4) is removed via endpoint dissipation.

Field after step 2:

```text
  (0): [[0]]^θ    (1): [[1]]^θ    (2): [[0]]^θ    (3): [[0]]^θ
```

The rod has shortened from 5 to 4 lattice points. This illustrates how absorbing boundaries can dominate over thermal diffusion in small systems — the heat energy was not able to propagate fast enough (or sustain long enough) to keep the endpoint active.

### Step 3 — Stochastic Assignment

```text
  (0): [[0]]^θ    (1): [[0]]^θ    (2): [[1]]^θ    (3): [[0]]^θ
```

**Interaction check:**
- (0) `[[0]]` and (1) `[[0]]`: same values → *equilibrium*.
- (1) `[[0]]` and (2) `[[1]]`: different values → *coherent propagation*.
- (2) `[[1]]` and (3) `[[0]]`: different values → *coherent propagation*.

**Removal check:** Endpoint (0) has now been `[[0]]` for 3 consecutive steps (exceeds M = 2). Its only neighbor is (1), which is `[[0]]` — no active neighbor. Position (0) is removed via endpoint dissipation. Endpoint (3) has been `[[0]]` for 3 consecutive steps, but its neighbor (2) is `[[1]]` — has an active neighbor, so not removed.

Field after step 3:

```text
  (1): [[0]]^θ    (2): [[1]]^θ    (3): [[0]]^θ
```

The rod has contracted to 3 lattice points. The remaining thermal energy at position 2 sustains the interior, but the absorbing boundary continues to erode the cold endpoints.

---

## Physical Observations

The [Field Phenomena](./CONCEPTS.md#field-phenomena) section in CONCEPTS.md defines the physical concepts that emerge from the framework's mechanisms. This section describes how each phenomenon manifests specifically in the heat rod model.

### Thermal Diffusion Rate

In a biased stochastic model (see Extensions), the rate at which the hot region expands depends on the coupling bias strength. Higher bias → faster apparent diffusion. The diffusion front tracker subscriber can quantify this by measuring the slope of front position vs. time.

In the absence of biased generation, diffusion is dominated by stochastic noise — hot and cold points appear randomly, and any apparent spreading is statistical rather than deterministic.

### Thermal Equilibrium

Under fixed boundaries with no damping, the rod should eventually reach a uniform state (all hot or all cold) as stochastic fluctuations smooth out temperature differences. The equilibrium detector subscriber tracks this by measuring the fraction of adjacent pairs in the same state.

In a biased model, equilibrium is reached faster because coherent propagation actively drives the system toward uniformity. The time to equilibrium depends on rod length and coupling strength.

### Boundary Reflection

Under fixed (insulated) boundaries, thermal energy reaching the endpoint cannot escape. With biased generation, the endpoint tends to retain its thermal state, effectively reflecting heat back into the rod. A hot pulse initiated at the center will spread outward, reach both endpoints, and "reflect" — the endpoints remain hot and continue to transfer energy inward.

Under absorbing boundaries, as shown in the worked example, endpoints dissipate and are removed. There is no reflection — heat is lost at the boundaries.

Under periodic boundaries, heat reaching one endpoint emerges at the other, circulating around the ring indefinitely.

### Damping

Without explicit damping, thermal energy is sustained only by stochastic generation, which is equally likely to create or destroy hot states. When damping is introduced (see Extending the Model), hot points have a probability of returning to cold each step, independent of their neighbors.

With damping active, the thermal energy monitor will show declining total energy over time unless the diffusion rate (coupling strength) exceeds the damping rate. Under fixed boundaries with no damping, energy is approximately conserved; with damping, energy decays exponentially.

### Standing Thermal Waves

Standing waves in the heat rod manifest as positions that oscillate between hot and cold with regular periodicity while neighboring positions remain relatively static. This requires three conditions:

1. Coherent propagation (biased stochastic generation)
2. Boundary reflection (fixed or periodic boundaries)
3. Interference (reflected waves overlapping with outgoing waves)

Standing thermal waves are more observable under periodic boundaries, where heat can circulate and constructively interfere with itself. The rod length must support integer half-wavelength patterns for standing waves to form. The equilibrium detector can identify potential standing wave positions by tracking the temporal autocorrelation of individual lattice points.

### Resonance

In periodic boundary conditions with biased stochastic evolution, certain rod lengths may produce thermal oscillation patterns that persist longer than others — analogous to resonant frequencies of a thermal ring. The standing wave detector can identify these by comparing pattern persistence across different rod lengths.

---

## Comparison to Multi-Dimensional Models

The 1D heat rod has only one configuration, which eliminates mode coupling and scattering while preserving all other phenomena (diffusion, reflection, damping, equilibrium, standing waves, resonance). This makes it an excellent pedagogical starting point before progressing to multi-dimensional models where configuration variety enables richer interaction dynamics.

See [Dimensionality Effects](./CONCEPTS.md#dimensionality-effects) in CONCEPTS.md for a comparison table of how dimensionality affects available configurations, phenomena, and analysis patterns across 1D, 2D, and 4D implementations.

---

## Extending the Model

The basic heat rod implementation can be extended in several directions:

- **Biased stochastic generation** — Instead of uniform 50/50, set the probability of `V = 1` based on neighbor values. For example, if a neighbor has `θ = 1`, set `P(θ = 1) = 0.7` for the current point. This simulates thermal coupling and produces coherent diffusion instead of pure thermal noise.

- **Coupling strength parameter** — Introduce a parameter λ ∈ [0, 1] that controls the neighbor influence bias. λ = 0 is pure stochastic (no coupling); λ = 1 is fully deterministic (a neighbor's thermal state is always copied). Intermediate values produce varying degrees of diffusion coherence.

- **Damping** — Introduce a probability δ that any hot point becomes `[[0]]` independent of its neighbors, simulating environmental heat loss. Higher δ → faster cooling. The relationship between coupling strength λ and damping δ determines whether the rod heats, cools, or reaches dynamic equilibrium.

- **Variable thermal conductivity** — Extend to two dimensions by adding a "material type" dimension that distinguishes between rod segments. Different material types could have different effective coupling strengths, modeling a composite rod with spatially varying conductivity.

- **Heat source/sink** — Designate certain positions as persistent heat sources (always `[[1]]`) or heat sinks (always `[[0]]`), simulating contact with thermal reservoirs. This enables steady-state analysis where diffusion balances against fixed boundary temperatures.

- **Continuous-valued extension** — Relax the binary constraint to allow values in [0, 1] representing continuous temperature. This enables smoother temperature profiles and more accurate diffusion modeling at the cost of computational complexity.

- **Visualization** — Map each step's rod state to a 1D color bar (e.g., blue for cold, red for hot). Stacking bars vertically produces a space-time diagram directly showing diffusion fronts, reflections, and standing wave patterns.

- **Multi-rod coupling** — Create a 2D array of 1D rods coupled at their endpoints or through cross-links, modeling a thermal network or heat exchanger. This demonstrates how the 1D building block scales to more complex geometries.