# Agent Guide

## Project Overview

**Gagan** is a mathematical analysis framework that models any n-dimensional field as a matrix of binary constituent matrices. The core principle is that simple binary mechanisms produce complex emergent phenomena — wave propagation, interference, scattering, resonance, and standing waves arise naturally from configurable interaction rules rather than being explicitly programmed.

The framework is domain-agnostic: the same mechanisms model four-dimensional spacetime, a vibrating elastic membrane, or a heat-conducting rod. The phenomena are universal; only the physical interpretation changes.

## Project Structure

```
gagan/
├── AGENTS.md           # This file — agent instructions
├── LICENSE             # MIT
├── README.md           # High-level overview and getting started links
└── docs/
    ├── CONCEPTS.md     # Full framework definition (mechanisms, phenomena, analysis patterns)
    ├── 1D-HEAT-ROD.md  # 1D implementation — heat conduction along a rod
    ├── 2D-MEMBRANE.md  # 2D implementation — vibrating elastic membrane
    └── 4D-SPACETIME.md # 4D implementation — spatial presence and causal structure
```

This is currently a documentation-only project defining the framework's design. No source code exists yet.

## Documentation Map

| File | Purpose | Key Sections |
|------|---------|--------------|
| `README.md` | Entry point — overview, mechanisms summary, emergent phenomena list, getting started links | How It Works, What Emerges |
| `docs/CONCEPTS.md` | Canonical framework definition — read this first for any implementation work | Field Representation, Configurations, Interaction Rules, Boundary Conditions, Stochastic Evolution, Event System, Field Phenomena, Analysis Patterns, Dimensionality Effects |
| `docs/1D-HEAT-ROD.md` | Simplest implementation — good pedagogical starting point | Dimension Mapping, Value Semantics, Interaction Rules (simplified — no mode coupling/scattering) |
| `docs/2D-MEMBRANE.md` | Multi-dimensional implementation with configuration variety | Configurations, Mode Coupling, Scattering |
| `docs/4D-SPACETIME.md` | Full framework expressiveness with rich configuration dynamics | Configurations, Causal Interaction, Vacuum Collapse, Spatial–Temporal Coupling |

All implementation docs follow a consistent structure:
1. Physical Background
2. Dimension Mapping
3. Value Semantics
4. Field Representation (with concrete examples)
5. Configurations (with physical interpretation)
6. Interaction Rules (with outcome table)
7. Boundary Conditions (with physical analogs)
8. Initial Conditions (with example setups)
9. Stochastic Evolution
10. Event System (with domain-specific subscribers)
11. Creation and Removal of Constituents
12. Worked Example (step-by-step walkthrough)
13. Physical Observations (mapping framework phenomena to domain)
14. Extending the Model

## Core Mechanisms

Agents must understand these six mechanisms — they are the building blocks of every implementation:

1. **Constituents** — Each field point is a binary matrix `[[v₀, v₁, ..., vₙ₋₁]]` with one element per dimension. Values are always 0 or 1.

2. **Configurations** — Each constituent is oriented toward one dimension (C₁ through Cₙ). Configuration determines which dimension governs interactions. An n-dimensional field has exactly n complementary configurations. Notation: `[[1, 0]]ᵘ` (u-config) vs `[[1, 0]]ᵛ` (v-config).

3. **Interaction Rules** — Adjacent constituents interact based on their values and configurations:
   - Same values, same config → Equilibrium
   - Same values, config mismatch → Mode coupling (only in n ≥ 2)
   - Different values, same config → Coherent propagation
   - Different values, config mismatch → Scattering (only in n ≥ 2)

4. **Boundary Conditions** — Four types with distinct behaviors:
   - Fixed: constant size, activity reflects inward
   - Free: can expand via creation rules
   - Periodic: wraps around, no edge effects
   - Absorbing: can shrink via removal rules

5. **Stochastic Evolution** — Each step: randomly assign 0/1 to every dimension of every constituent, then evaluate interactions and boundary rules. Pure stochastic produces noise; biased stochastic (neighbor influence) produces coherent patterns.

6. **Event System** — Four event types: `VALUE_ASSIGNED`, `CONSTITUENT_CREATED`, `CONSTITUENT_REMOVED`, `INTERACTION`. Subscribers perform analysis without modifying core logic.

## Emergent Field Phenomena

These arise from the mechanisms — agents should recognize them when analyzing or describing implementations:

- **Coupling** — neighbors influence each other (realized through interaction rules)
- **Propagation** — activity spreads from sources (requires biased stochastic generation)
- **Reflection** — activity bounces off boundaries (fixed/periodic only)
- **Interference** — overlapping wavefronts combine (constructive or destructive)
- **Scattering** — propagation disperses at configuration boundaries (n ≥ 2 only)
- **Damping** — activity decays over time (requires explicit damping extension)
- **Standing Waves** — stationary oscillating patterns (requires propagation + reflection + interference)
- **Resonance** — self-reinforcing patterns at certain geometries (higher-order emergent)

## Analysis Patterns

Standard approaches for observing field behavior (model-agnostic):

- **Density and Distribution** — track concentration of active values over time
- **Clustering and Contiguity** — identify adjacent groups with similar states
- **Interaction Frequency** — monitor how often each rule fires
- **Field Topology** — observe shape/size changes (boundary-dependent)
- **Configuration Distribution** — track config ratios and clustering (n ≥ 2 only)

## Dimensionality Effects

When creating or reviewing implementations, note how dimensionality affects available phenomena:

| Dimensionality | Configurations | Mode Coupling | Scattering | Complexity |
|----------------|----------------|---------------|------------|------------|
| 1D | 1 (no choice) | Impossible | Impossible | Minimal — good starting point |
| 2D | 2 | Possible | Possible | Moderate — demonstrates multi-mode interaction |
| 4D | 4 | Rich | Rich | High — full framework expressiveness |

## Conventions for New Implementations

When adding a new implementation document to `docs/`:

1. **Choose a real physical phenomenon** that maps naturally to an n-dimensional field (e.g., electromagnetic wave propagation, fluid dynamics, quantum lattice, population dynamics).

2. **Define dimension mapping early** — explicitly state what each dimension index represents in the physical domain.

3. **Define value semantics explicitly** — state what 0 and 1 mean in physical terms for this domain.

4. **Follow the standard document structure** listed above. Every section should appear; if a section has no meaningful content (e.g., Creation/Removal under fixed boundaries), explicitly state why rather than omitting it.

5. **Include a worked example** with at least 3 evolution steps showing stochastic assignment, interaction evaluation, and any creation/removal checks. Use concrete values, not abstract descriptions.

6. **Map framework phenomena to domain observations** — the Physical Observations section should explicitly connect each phenomenon (coupling, propagation, etc.) to its domain-specific manifestation.

7. **Cross-reference CONCEPTS.md** — link to the relevant framework definition sections rather than redefining mechanisms.

8. **Update README.md** — add a link to the new doc in the Getting Started section with a one-line description of the domain and key phenomena demonstrated.

9. **Use consistent notation** — constituent matrices use `[[...]]`, configurations use superscripts (ᵘ, ᵛ, ˣ, ʸ, ᶻ, ᵗ for Unicode; `^θ` where no Unicode superscript exists), positions use `(row, col)` or `(index)` notation.

## Common Extension Patterns

Implementations typically list extensions. These are the standard patterns that apply across domains:

- **Biased stochastic generation** — neighbor values influence assignment probability (essential for coherent propagation)
- **Coupling strength parameter** — λ ∈ [0, 1] controlling neighbor influence
- **Damping** — probability of resetting active constituents to zero
- **Configuration reassignment** — configs change based on interaction outcomes
- **Continuous-valued extension** — relax binary constraint to [0, 1] or wider range
- **Visualization** — map field state to color/geometry per step

## Key Terminology

| Term | Definition |
|------|------------|
| Constituent | A single point in the field; a binary matrix `[[v₀, ..., vₙ₋₁]]` |
| Configuration | The dimension a constituent is oriented toward (C₁ through Cₙ) |
| Equilibrium | Interaction outcome when adjacent constituents have same values and same config |
| Mode coupling | Interaction outcome when same values but config mismatch (energy transfer between dimensions) |
| Coherent propagation | Interaction outcome when different values but same config (activity spreads) |
| Scattering | Interaction outcome when different values and config mismatch (activity disperses) |
| Biased stochastic | Random assignment influenced by neighbor values (vs. pure 50/50) |