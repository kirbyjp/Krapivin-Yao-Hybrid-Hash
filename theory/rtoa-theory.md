---
title: "Repatriatory Tiered Open Addressing (RTOA): A Hierarchy-Aware Extension to Optimal Open Addressing"
authors: [Your Name], [Collaborator Names]
date: "June 2025"
subject: "Data Structures and Algorithms (cs.DS); Hardware Architecture (cs.AR)"
keywords: [Open Addressing, Hash Tables, Cache Locality, Tiered Storage, Self-Healing Data Structures]
---

# Repatriatory Tiered Open Addressing (RTOA): A Hierarchy-Aware Extension to Optimal Open Addressing

## 1. Abstract
We present **Repatriatory Tiered Open Addressing (RTOA)**, a dynamic migration model for hash tables designed to reconcile theoretical probe complexity with hierarchical memory latency. While recent work by **Krapivin (2025)** established constant-time $O(1)$ search bounds for open addressing without reordering, those results assume uniform access costs across the address space. RTOA generalizes this model by permitting **Economic Repatriation**—a mechanism that exploits L3 cache locality by migrating elements from a stable overflow tier to a locality-optimized primary tier. We demonstrate that RTOA maintains an empirical latency floor $\sim 17\times$ lower than static Elastic Hashing while preserving $O(1)$ search guarantees.

---

## 2. Model Boundaries and Theoretical Positioning
RTOA is a generalization of tiered open addressing that relaxes the "No Reordering" constraint. 

**Model Distinction:** RTOA does not attempt to improve the asymptotic lower bounds for static open addressing; instead, it achieves a lower physical latency floor by operating in a strictly more expressive model class that permits controlled post-insertion movement. While Krapivin establishes the absolute performance floor for static tables, RTOA utilizes that floor as a **Stability Tier ($T_2$)** while leveraging reordering to achieve higher physical performance in the **Locality Tier ($T_1$)**.

---

## 3. The Formal Model

### 3.1 Hierarchical Memory Cost Function
We assume a memory system $M$ with non-uniform access costs $C(s)$. Memory is fetched in contiguous blocks (cache lines) of size $\beta$.
$$C(s) = \begin{cases} 
C_{local} & \text{if } s \in \text{L1/L2 cache line} \\
C_{remote} & \text{if } s \notin \text{L1/L2 cache line} 
\end{cases}$$
where $C_{remote} \gg C_{local}$.

### 3.2 Assumptions Summary

| Assumption | Justification |
| :--- | :--- |
| **Uniform Hashing** | Standard requirement in open-addressing literature for $O(1)$ expected cost. |
| **Fixed Cache-line Width ($\beta$)** | Hardware constant (e.g., 64-bytes on x86 architectures). |
| **Tier-2 $O(1)$ Guarantee** | Provided by the implementation of Elastic Hashing (Krapivin, 2025). |
| **Bounded Repatriation Cost** | Migration involves a single memory write ($C_{move}$), a constant overhead. |

---

## 4. Economic Repatriation and Entropy

### 4.1 The Repatriation Rule
Let $H \in T_1$ be a vacated slot. Let $V \subset T_2$ be the set of keys currently in the stability tier. Migration $k \in V \to H$ occurs if $H$ lies within the **Hardware Proximity Window** $W_k$ (size $\omega = \beta$) and the **Economic Condition** is met:
$$C_{move} < \sum_{t=1}^{\infty} P(lookup_t) \cdot (C_{T2} - C_{T1})$$

**Amortized Frequency:** Under uniform hashing and typical deletion patterns, the expected number of repatriation events per insertion or deletion is $O(1)$. 

### 4.2 Tier-2 Entropy Behavior
Standard open-addressed tables often accumulate entropy (disorder) over time. In the RTOA model, Tier-2 entropy does not accumulate unboundedly. Because repatriation preferentially removes keys whose locality windows overlap Tier-1 vacancies, Tier-2 entropy remains bounded in expectation. The mechanism acts as a self-correcting filter that actively "drains" the stability tier and restores physical locality.

---

## 5. Complexity and Claims

**Claim 1 (Search Complexity).** The expected search cost $E[S]$ is $O(1)$ for all load factors $\alpha < 1$.  
*Proof Sketch:* $T_2$ lookup is $O(1)$ (Krapivin, 2025). $T_1$ search is strictly bounded by the constant $\omega$. The resulting weighted sum of lookups remains independent of table size $N$.

**Claim 2 (Worst-Case Stability).** Economic Repatriation does not degrade worst-case performance.  
*Proof Sketch:* Repatriation is a one-way promotion from a higher-latency tier to a lower-latency tier. Failed attempts incur zero additional probe costs for subsequent retrievals.

---

## 6. Experimental Methodology and Results

### 6.1 Benchmark Environment
*   **CPU:** Intel Core i7-7700HQ (6MB L3 Cache, 64-byte cache lines).
*   **Implementation:** Contiguous `Uint32Array` saturation to model physical L3 cache behavior.
*   **Adversarial Clustering:** 10% of total keys forced into 0.1% of the available address space.

### 6.2 Results Summary ($\alpha = 0.95$)
| Implementation | Avg. Latency | Complexity Class |
| :--- | :--- | :--- |
| **Pure Greedy** | ~2550.0 ms | $O(N)$ (The Wall) |
| **Elastic (Raw Krapivin)** | ~60.0 ms | $O(1)$ (Theoretical Floor) |
| **RTOA (Hybrid)** | **3.5 ms** | **$O(1)$ (Hardware Optimal)** |

---

## 7. Related Work
*   **Theoretical Foundations:** Yao (1985), Krapivin (2025).
*   **Reordering Variants:** Cuckoo Hashing (2001), Robin Hood Hashing (1986).
*   **Cache-Conscious Systems:** CLHT (Cache-Line Hash Table) and Facebook’s Folly F14. These systems demonstrate the critical importance of cache-line alignment in practical hash tables, reinforcing the motivation for RTOA's tiered, hierarchy-aware design.

---

## 8. Limitations
1.  **Alignment Dependency:** Performance is contingent on tuning $\omega$ to the physical cache-line width of the host CPU.
2.  **Repatriation Idle-Time:** In write-only environments with no deletions, the Vacuum trigger remains inactive, though the $T_2$ constant-time safety net remains functional.

---

## 9. Future Work
Further investigation is required regarding:
1.  **Dynamic Window Sizing:** Adapting $\omega$ at runtime based on detected cache-miss penalties.
2.  **Multi-Tier Scaling:** Extending the model to include NUMA-aware tiers for distributed memory environments.

---

## 10. Conclusion
RTOA bridges the gap between mathematical proofs and physical execution. By treating "No Reordering" as a theoretical lower bound rather than an operational requirement, RTOA achieves a hardware-optimal implementation that maintains L3 cache locality even at extreme occupancy and adversarial clustering.
