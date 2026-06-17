## Relationship to Krapivin’s “No Reordering” Constraint

Krapivin’s 2025 paper focuses on a very specific and historically important variant of open‑addressed hashing:

> **Insertions may not reorder or move any previously inserted elements.**

This constraint is central to the classical formulation of the problem.  
The insertion algorithm must choose an empty slot from the probe sequence, but it may **not**:

- relocate an earlier key,
- compact clusters,
- or perform any form of backtracking or migration.

Krapivin’s breakthrough shows that even under this strict rule set, it is possible to design probe sequences that avoid the classical “Linear‑Probing Wall” and achieve dramatically better expected probe complexity.

### How This Project Differs

This hybrid system does **not** operate under the “no reordering” restriction.  
It intentionally introduces a mechanism that Krapivin’s model forbids:

### **Repatriation (Backtracking Migration)**  
When space becomes available in the fast tier (Tier 1), the system may:

- identify an element stored in the overflow tier (Tier 2),
- remove it from the overflow structure,
- and migrate it back into the locality‑optimized region.

This is a form of **post‑insertion movement**, which is explicitly disallowed in Krapivin’s theoretical framework.

### Why This Is Not a Violation — But a Different Problem Class

Krapivin’s results apply to the classical model of open addressing **without reordering**.  
Your system is designed for **real hardware**, not for that classical model.

The goals differ:

| Krapivin’s Goal | This Project’s Goal |
|-----------------|---------------------|
| Prove that constant‑time search is possible without ever moving elements. | Achieve stable, cache‑friendly performance on real CPUs, even under adversarial clustering. |
| Work within a strict theoretical constraint. | Exploit modern hardware behavior (cache lines, locality windows, dynamic migration). |
| Construct a mathematically optimal probe sequence. | Build a practical hybrid architecture with bounded locality and self‑healing. |

### Does Repatriation “Break the Rules”?

Under Krapivin’s rules: **yes** — repatriation is not allowed.

Under real‑world systems design: **no** — repatriation is a powerful optimization that:

- reduces entropy,
- restores locality,
- drains pressure from the overflow tier,
- and prevents long‑term degradation.

Your system is not trying to solve the same constrained problem he solved.  
It is solving a **systems‑level performance problem** that his paper does not address.

### How the Two Approaches Relate

- Krapivin proves that the wall is not mathematically inevitable.  
- This hybrid system shows how to avoid the wall **in practice**, using mechanisms that the classical model forbids but real hardware benefits from.

In other words:

**Krapivin breaks the wall in theory.  
This project breaks the wall in practice.**

They are complementary, not competing.

### Hardware Focus

This project is tuned for the Intel i7-7700HQ:

Cache Line Awareness: The search window is set to 16 slots (64 bytes), ensuring the entire "Proximity Search" stays within a single L1/L2 cache fetch.
L3 Saturation: The test size is kept under 6MB to model performance when the entire data structure resides in the processor's last-level cache.
