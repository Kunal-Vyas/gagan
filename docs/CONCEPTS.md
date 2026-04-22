# Gagan Framework

Gagan represents any field as a matrix of matrices. Each inner matrix — called a **constituent** — represents a single point in the field. This document describes the framework's core concepts, illustrated throughout with a concrete implementation for four-dimensional spacetime.

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

When two adjacent constituents share the same values but differ in configuration, the model defines how they interact. These rules are customizable.

**Spacetime Implementation — Example Rule Set:**

- **Same configuration, same values** — No interaction; the constituents are in equilibrium.
- **Same values, spatial–spatial config mismatch** (e.g., x-conf vs. z-conf) — The constituents exert a mutual spatial displacement influence; the field may propagate activity between the two configured dimensions.
- **Same values, spatial–temporal config mismatch** (e.g., x-conf vs. t-conf) — The spatially configured constituent is "frozen" for one evolution step while the temporally configured constituent advances, simulating a causal relationship.
- **Different values, any config mismatch** — No special interaction; standard stochastic evolution applies.

---

## Initial Conditions

Analysis begins with predefined initial conditions:

1. **Field dimensions** — The size of the outer matrix (e.g., 3×2, 5×5, 10×10).
2. **Initial constituent values** — Either all zeros (empty field) or a specific seed pattern.
3. **Initial configurations** — How constituents are assigned configurations (e.g., uniformly at random, or all set to one configuration).
4. **Evolution parameters** — Number of steps, whether configurations are reassigned between steps, etc.

**Spacetime Implementation — Example Initial Condition:**

```text
Field: 3×3
Seed: center constituent set to [[1, 1, 1, 1]], all others [[0, 0, 0, 0]]
Configurations: all set to t-conf
Steps: 100
```

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
   c. Evaluate creation/removal conditions
   d. Apply any event-driven actions
```

**Spacetime Implementation:**

For a 3×3 field, each step generates 3 × 3 × 4 = 36 random binary values — one per dimension per constituent.

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

Under predefined conditions, the field can grow or shrink. These operations are event-driven — triggered by subscribing to `VALUE_ASSIGNED` or `INTERACTION` events and evaluating the defined conditions.

### Creation

New constituents are inserted into the field when creation conditions are met.

**Spacetime Implementation:**

- **Spatial expansion** — A constituent with all spatial dimensions active (`[1, 1, 1, *]`) and in a spatial configuration has no neighbor in at least one adjacent position. A new `[[0, 0, 0, 0]]` constituent is created in the empty position with a randomly assigned configuration.
- **Temporal branching** — A constituent in t-conf with `t = 1` that has persisted for more than K consecutive steps causes a new constituent to be created at its position with the same values but a randomly chosen spatial configuration (simulating a temporal "fork" into spatial evolution).

### Removal

Constituents are removed from the field when removal conditions are met.

**Spacetime Implementation:**

- **Vacuum collapse** — A constituent has been `[[0, 0, 0, 0]]` for more than M consecutive steps and has no active neighbors. It is removed, shrinking the field.
- **Causal isolation** — A constituent in a spatial configuration has been spatially inactive (`x = 0, y = 0, z = 0`) for more than L steps while all its neighbors are in temporal configurations. It is removed, simulating the disappearance of a spatially disconnected point.

---

## Worked Example

**Spacetime Implementation — Step-by-Step Walkthrough**

### Setup

```text
Field: 2×2
Initial state:
  (0,0): [[0, 0, 0, 0]]ᵗ
  (0,1): [[0, 0, 0, 0]]ᵗ
  (1,0): [[1, 1, 1, 1]]ᵗ
  (1,1): [[0, 0, 0, 0]]ᵗ
Subscribed actions: spatial expansion, vacuum collapse (M = 2)
```

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

**Creation check:** (1,0) has spatial values `[1, 1, 1]` and is now in x-conf (spatial). It has no neighbor at position (1,2) — the field boundary. Boundary behavior is model-specific and may or may not trigger expansion.

**Removal check:** (0,0) and (0,1) have been `[[0, 0, 0, 0]]` for 2 consecutive steps. (0,0)'s only neighbor is (0,1), which is also all-zero — no active neighbor. (0,0) is removed via vacuum collapse. Similarly for (0,1) — its neighbor (0,0) is being removed, and its other neighbor (1,1) is all-zero — removed.

Field after step 3:

```text
  (1,0): [[1, 1, 1, 1]]ˣ
  (1,1): [[0, 0, 0, 0]]ᵗ
```

The field contracted from 2×2 to an effective 1×2 as vacant points collapsed — illustrating how stochastic generation, complementary configurations, and event-driven rules combine to produce dynamic field evolution.

---

## Extending the Model

The spacetime implementation serves as a foundation. Directions for extension:

- **Weighted stochastic generation** — Instead of uniform 50/50, bias random assignment based on neighboring values (e.g., a constituent is more likely to adopt a neighbor's spatial value, simulating locality).
- **Configuration reassignment** — Allow configurations to change between steps based on interaction outcomes, not just at initialization.
- **Larger fields and batching** — Scale to 100×100 or larger fields and batch-process evolution steps to study macro-scale pattern emergence.
- **Visualization** — Map each step's field state to a color-coded grid where spatial dimensions map to RGB channels and temporal presence maps to brightness, producing a frame sequence playable as video.