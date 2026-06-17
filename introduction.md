## How This Work Differs From Krapivin’s Elastic Hashing

This project is inspired by Andrew Krapivin’s 2025 breakthrough on **Elastic Hashing**, but it is not an implementation of his algorithm. The two approaches share a conceptual motivation — avoiding probe‑sequence collapse at high occupancy — yet they diverge in architecture, goals, and underlying mechanisms.

### 1. Different Problem Framing
Krapivin’s work is fundamentally **theoretical**. His contribution shows that the classical “Linear‑Probing Wall” is not a mathematical inevitability. He constructs a probe sequence that guarantees constant‑time behavior even at extreme load factors.

This project is fundamentally **systems‑oriented**. The goal is to design a hashing architecture that:

- respects modern CPU cache hierarchies  
- minimizes unpredictable memory jumps  
- bounds probe lengths to a single cache‑line window  
- provides stable performance under adversarial clustering  

The hybrid system is not a proof technique — it is a practical design for real hardware.

### 2. Different Architectural Model
Krapivin’s Elastic Hashing uses **multi‑level logical arrays** (A₁, A₂, A₃…) to bound probe complexity.

This project uses a **tiered physical model**:

- **Tier 1 (Locality Tier):** a cache‑aligned, bounded‑window search region  
- **Tier 2 (Vault Tier):** a non‑greedy overflow structure  
- **Repatriation:** a self‑healing mechanism that migrates overflow entries back into Tier 1 when space becomes available  

The tiers in this system correspond to **hardware locality**, not abstract probe levels.

### 3. Different Handling of Overflow
Krapivin’s algorithm ensures that probe sequences remain short by construction.

The hybrid system takes a different approach:

- If a key cannot be placed within the locality window, it is redirected to the **Vault Tier**.  
- A lightweight **breadcrumb** is left behind to preserve lookup determinism.  
- When deletions or vacated slots appear, **repatriation** pulls keys back into the fast tier.

This creates a dynamic equilibrium that maintains locality without requiring a mathematically constructed probe sequence.

### 4. Different Performance Goals
Krapivin’s work proves that constant‑time behavior is *possible*.

This project demonstrates that constant‑time behavior is *achievable in practice* on commodity hardware, even under:

- adversarial clustering  
- high occupancy  
- constrained cache‑line locality  

In stress tests, the hybrid system achieved up to **780× faster** lookups than traditional greedy linear probing at 90% load.

### Summary
Krapivin showed that the wall can be broken in theory.  
This project shows how to build a hashing system that avoids the wall **in real hardware**, using a hybrid tiered design that emphasizes:

- cache locality  
- bounded probe windows  
- overflow elasticity  
- dynamic self‑healing  

The two approaches are complementary, but not equivalent.
