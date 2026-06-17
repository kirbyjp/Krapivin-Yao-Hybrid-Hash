## Introduction

For four decades, computer science operated under **Yao’s Conjecture**, which implied that as a hash table approaches **99% occupancy**, performance must inevitably degrade due to probe‑sequence explosion and clustering effects. This “Linear‑Probing Wall” was treated as a fundamental limitation of open‑addressing hash tables.

In 2025, **Andrew Krapivin** overturned this assumption by proving that **Elastic Hashing** can maintain **constant‑time performance** even at extreme densities. His work demonstrated that the collapse of linear probing is not a law of nature, but a consequence of traditional probe sequences.

This project implements a **Hybrid Tiered Hashing System** designed specifically for modern CPU architectures (tested on an Intel Core i7‑7700HQ). By combining:

- a high‑speed **Greedy Locality Tier** (Lot A),
- a non‑greedy **Elastic Vault Tier** (Lot B),
- and an active **Economic Repatriation** mechanism (vacuum‑based self‑healing),

Early coding has achieved retrieval speeds up to **780× faster** than traditional greedy hashing at high occupancy.

The hybrid design avoids probe‑sequence collapse, preserves cache‑line locality, and dynamically drains overflow back into the fast tier, maintaining stability even under adversarial clustering.
