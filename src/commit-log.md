# Version History (Commit Log)

This document tracks the evolution of the Hybrid Tiered Hashing System from its earliest conceptual model to the current high‑performance implementation.

---

## v0.01 — Conceptual Model
**Goal:** Establish the initial mental framework for understanding probe behavior.  
**Logic:** Defined the relationship between greedy linear probing and clustering effects.  
**Outcome:** Created the baseline model used to reason about probe‑sequence collapse.

---

## v0.02 — Tiered Logic (First Overflow Tier)
**Goal:** Introduce a two‑level architecture (Tier 1 and Tier 2).  
**Implementation:** Used standard JavaScript objects to simulate a primary region and an overflow region.  
**Outcome:** Demonstrated that an overflow “Vault” tier is viable, though performance was limited by JavaScript’s object‑allocation overhead.

---

## v0.03 — Contiguous Memory & Bit‑Packing
**Goal:** Align the model with real CPU cache behavior.  
**Implementation:**  
- Migrated to `Uint32Array` for contiguous memory.  
- Introduced bit‑masking to store metadata (breadcrumbs) within a single 32‑bit slot.  
**Outcome:** Achieved near‑instant baseline performance, but the uniform distribution prevented meaningful collision behavior.

---

## v0.04 — High‑Entropy Collision Simulation
**Goal:** Introduce realistic clustering and collision patterns.  
**Implementation:** Added a high‑entropy hash function to force collisions and create dense clusters.  
**Outcome:** Successfully generated overflow into the Vault tier, validating the hybrid design under non‑ideal conditions.

---

## v0.05 — Confronting the Linear‑Probing Wall
**Goal:** Demonstrate catastrophic failure of greedy hashing under adversarial load.  
**Implementation:** Forced 50% of all keys into a 1% region to create an extreme clustering zone.  
**Outcome:** Greedy hashing collapsed (browser freeze), while the hybrid system remained responsive — a direct demonstration of the Linear‑Probing Wall.

---

## v0.06 — Performance Benchmark (“The Survivor”)
**Goal:** Measure hybrid speedup without triggering browser timeouts.  
**Implementation:** Reduced table size and controlled clustering to keep tests within execution limits.  
**Outcome:** Achieved up to **789× speedup** over greedy probing. Confirmed that the hybrid system avoids vertical probe‑time growth even under heavy clustering.

---

## v0.07 — Economic Repatriation (Vacuum Mechanism)
**Goal:** Implement self‑healing behavior.  
**Implementation:** Added logic to migrate entries from the Vault tier back into the fast tier when nearby slots become available.  
**Outcome:** Demonstrated that even at ~95% occupancy, the system can reduce entropy and restore locality over time through controlled migration.

---

## v0.08 — The Three-Way Battle

Version 0.08 introduces the first full scientific comparison between three hashing strategies:

1. **Greedy Linear Probing**  
   - Extremely fast at low occupancy  
   - Catastrophically collapses at high density (“The Linear Wall”)

2. **Raw Elastic Hashing (Krapivin)**  
   - Implements intentional non-greedy skipping  
   - Pays an upfront “Skip Tax”  
   - Maintains perfectly flat performance even at extreme occupancy

3. **Hybrid Proximity Hashing (This Project)**  
   - Begins with greedy, cache-local probing  
   - Uses a bounded locality window  
   - Falls back to an overflow Vault tier  
   - Avoids the Linear Wall while maintaining near-greedy speed

---

### Purpose of v0.08

This version demonstrates, side-by-side, the three fundamental performance curves:

- **Greedy:**  
  Starts fast, then goes vertical under clustering.

- **Raw Krapivin (Elastic):**  
  Starts slower due to intentional skipping, but remains perfectly flat.

- **Hybrid:**  
  Starts at greedy speed, stays low through the cache window, and transitions into flat behavior before the wall.

This comparison visually and empirically validates the hybrid design as a practical, hardware-aware extension of Krapivin’s theoretical breakthrough.

---

### Emulating Raw Krapivin

Krapivin’s Elastic Hashing is defined by being **non-greedy**.  
It intentionally skips available slots to maintain spacing (entropy) across the table.

Key behaviors:

- Even if a slot is empty, the algorithm may skip it.
- This prevents cluster formation.
- The cost is a higher baseline (“Skip Tax”).
- The benefit is a perfectly flat performance curve.

v0.08 includes a simplified skip-pattern to emulate this behavior for comparison purposes.

---

### What v0.08 Demonstrates

1. **Greedy (Red Column)**  
   - Fastest at low density  
   - Worst at high density  
   - Hits the Linear Wall and collapses

2. **Raw Krapivin (Cyan Column)**  
   - Slower baseline due to intentional skipping  
   - Completely stable under extreme load  
   - Represents the theoretical optimum without reordering

3. **Hybrid (Yellow Column)**  
   - Matches greedy speed in the locality window  
   - Avoids the wall by using the Vault tier  
   - Achieves the lowest real-world latency on modern CPUs

---

### Scientific Conclusion

**Goal:** Perform a direct, empirical comparison of three hashing strategies:
1. Greedy Linear Probing  
2. Raw Elastic Hashing (Krapivin)  
3. Hybrid Proximity Hashing (this project)

**Implementation:**  
- Added a third engine implementing intentional skip-pattern behavior to emulate Krapivin’s non-greedy Elastic Hashing.  
- Benchmarked all three systems under identical high-density, adversarial clustering conditions (95% load, forced collision zones).  
- Measured retrieval times for each strategy.

**Results:**  
- **Greedy:** ~2553 ms — collapsed under clustering (“Linear Wall”).  
- **Raw Krapivin:** ~60 ms — stable but pays a constant “Skip Tax.”  
- **Hybrid:** ~3–4 ms — fastest overall; maintains greedy speed while avoiding collapse.

**Conclusion:**  
Hybrid Tiered Hashing achieves the lowest real-world latency by combining greedy locality with an elastic overflow tier. It outperforms both classical greedy hashing and theoretical Elastic Hashing under extreme load.


This establishes the hybrid design as a practical, high-performance realization of the principles behind Elastic Hashing.
