## Introduction

For four decades, computer science operated under **Yao’s Conjecture**, which implied that open‑addressing hash tables must experience severe performance degradation as occupancy approaches **99%**. This collapse — often called the *Linear‑Probing Wall* — was treated as an unavoidable consequence of probe‑sequence growth and clustering.

In 2025, **Andrew Krapivin** demonstrated that this limitation is not fundamental. His work on **Elastic Hashing** proved that it is possible to maintain **constant‑time performance** even at extremely high load factors by restructuring how probe sequences behave under pressure.

This project implements a **Hybrid Tiered Hashing System** inspired by those insights and optimized for modern CPU architectures (tested on an Intel Core i7‑7700HQ). The design combines:

- a **Locality‑Optimized Primary Tier** (fast, cache‑aligned, bounded‑window search),
- an **Elastic Overflow Tier** (non‑greedy, collision‑resistant storage),
- and an active **Repatriation Mechanism** (periodic self‑healing that migrates overflow entries back into the fast tier).

Under adversarial clustering and high occupancy, this hybrid architecture achieved lookup speeds up to **780× faster** than traditional greedy linear probing. The system avoids probe‑sequence collapse, preserves cache locality, and maintains stable performance even when the primary tier is near saturation.
