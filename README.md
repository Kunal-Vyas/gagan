# Gagan

Gagan models any n-dimensional field as a matrix of binary constituent matrices. Each point in the field — called a **constituent** — carries a set of binary values (one per dimension), and the field evolves through stochastic processes governed by configurable interaction rules.

The core idea behind Gagan is that **simple binary mechanisms produce complex emergent phenomena**. By defining how constituents interact, how they are configured, and how the field is bounded, behaviors like wave propagation, interference, scattering, resonance, and standing waves emerge naturally — without being explicitly programmed. The same framework models four-dimensional spacetime, a vibrating elastic membrane, or any other field domain, because the mechanisms are domain-agnostic while the phenomena are universal.

## How It Works

Gagan defines a small set of mechanisms that together drive field evolution:

- **Constituents** — Each point in the field is a binary matrix with one element per dimension (0 or 1). A 4D spacetime point is `[[0, 1, 0, 0]]`; a 2D membrane point is `[[1, 0]]`.
- **Configurations** — Each constituent is oriented toward one of its dimensions, determining which dimension governs its interactions with neighbors.
- **Interaction rules** — When adjacent constituents meet, their values and configurations determine the outcome: equilibrium, mode coupling, coherent propagation, or scattering.
- **Boundary conditions** — The field edges can be fixed, free, periodic, or absorbing, shaping how activity reflects, expands, or dissipates.
- **Stochastic evolution** — At each step, every dimension of every constituent is randomly assigned 0 or 1, then interactions and events are evaluated.
- **Event system** — Every value assignment and interaction publishes an event that can be subscribed to for analysis.

## What Emerges

From these mechanisms, physical phenomena arise without being encoded directly:

- **Coupling** — Neighbors influence each other through interaction rules
- **Propagation** — Activity spreads outward from sources
- **Reflection** — Activity bounces off field boundaries
- **Interference** — Overlapping propagation fronts combine constructively or destructively
- **Scattering** — Propagation disperses at configuration boundaries
- **Damping** — Activity decays over time
- **Standing waves** — Reflected propagation creates stationary oscillating patterns
- **Resonance** — Certain field geometries self-reinforce, producing persistent patterns

## Getting Started

- [CONCEPTS.md](./docs/CONCEPTS.md) — Full framework definition: mechanisms, field phenomena, analysis patterns, and a four-dimensional spacetime implementation.
- [2D-MEMBRANE.md](./docs/2D-MEMBRANE.md) — A two-dimensional elastic membrane implementation demonstrating wave propagation, interference, mode coupling, and boundary effects.