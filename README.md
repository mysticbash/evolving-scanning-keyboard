# Evolving Scanning Keyboard

*A design document for an evolutionary-algorithm-based adaptive interface for single-switch assistive typing. This repository currently contains the design and algorithm write-up; implementation is in progress.*

## The Problem

Traditional single-switch assistive communication interfaces — like the scanning grid system famously used by Stephen Hawking — rely on static layouts with fixed scanning speeds. This architecture fails real-world users for two reasons:

- A patient's vocabulary is highly personalized.
- Their physical response latency degrades continuously over a multi-year biological timeline.

To address this, this project proposes an **Evolutionary Algorithm (EA)** that simultaneously co-optimizes the physical grid geometry (spatial layout) and the scanner timing intervals. Using a patient's rolling 4–5 year written history and periodic physical response benchmarks, the system would dynamically adapt the interface to maximize typing speed while actively preventing user fatigue.

## Why Evolution

The interface maps a character set of **N = 42 keys** — 26 letters, 10 digits, essential punctuation (`,` `.` `?`), Space, Backspace, and a state-switching toggle (Caps Lock/Symbols).

This creates a search space that mathematically justifies a heuristic approach over brute-force methods:

- **Permutation complexity:** shuffling 42 elements on a fixed grid yields 42! (≈ 1.4×10⁵¹) possible structural arrangements.
- **Topology complexity:** grid dimensions are also treated as variables, so the algorithm must evaluate asymmetric, non-uniform row-column configurations (e.g. a staggered 6-6-7-7-8-8 layout vs. a standard matrix).

## The Design

### How a Layout Is Stored

Each layout is represented using three linked arrays:

1. **Shape array (`row_sizes`)** — defines the grid structure, allowing uneven rows instead of a fixed rectangle, to explore whether that reduces scanning steps.
2. **Key placement array (`key_map`)** — length-42 array holding the exact row/column position of every character and button.
3. **Speed array (`time_usage`)** — per-row/column wait times (ms), letting scanning speed match the user's physical response capability.

### Crossover and Mutation

Because the interface requires an exact set of 42 distinct characters, a naive random shuffle during crossover would duplicate some letters while dropping others entirely.

To prevent this, the design uses a **permutation-safe copying method**: when combining two layout plans, the algorithm copies a block of keys from the first layout, then fills remaining slots using only the characters missing from the second layout — guaranteeing every generated layout contains exactly one copy of all 42 required keys.

### Fitness Function

```
Total score = Typing speed score − Fatigue penalty
```

- **Typing speed score:** simulated using the user's actual written message history from the last 4–5 years (not a generic dictionary), rewarding layouts that place frequently-used characters early in the scanning cycle.
- **Fatigue penalty:** takes the user's current physical reaction time (from periodic check-ups) as input; if their condition has progressed, the function heavily penalizes layouts that move too quickly, favoring safer, longer scanning delays.

### Selection and Mutation Strategy

- **Tournament selection** picks the highest-scoring layouts from random mini-pools to breed the next generation.
- **Mutation** swaps the positions of two keys or adjusts a scanning interval by ±10ms to explore new possibilities.

## Non-Obviousness

The obvious pitch is "use an EA to find a good keyboard layout" — which is close to what most people (or a chatbot) would suggest for an EA idea in accessibility. That's not the interesting part.

What's interesting is that the optimization target keeps changing on its own, because the patient's body keeps changing. Rather than running the EA once and stopping, the system is designed to re-run according to the patient's history and response time — continuously adapting rather than converging to a single "best" answer.

## Status

This repo currently holds the algorithm design and rationale. A working implementation (Python, simulating the EA loop against sample message histories) is planned as a next step.
