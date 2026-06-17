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

