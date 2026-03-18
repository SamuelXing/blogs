---
title: 'Improved Query Performance with Variant Indexes(1997)'
date: 2026-03-09
published: true
tags:
  - databases
  - query processing
---

This paper systematically analyzes variant index structures — bitmap indices, projection indices, and bit-sliced indices — for read-heavy data warehouse workloads, providing a cost-model-driven framework for choosing the right index type based on query pattern (selection, aggregation, grouping). It showed that no single index dominates all workloads, establishing the analytical basis for multi-index strategies in OLAP systems.

Link: [Bitmap_Indices.pdf](https://www.cs.cmu.edu/~15721-f25/papers/Bitmap_Indices.pdf)

<details markdown="1">
<summary><b>Detailed Notes</b></summary>

### Why is this paper important? (1-2 sentences)
This paper systematically analyzes variant index structures — bitmap indices, projection indices, and bit-sliced indices — for read-heavy data warehouse workloads, providing a cost-model-driven framework for choosing the right index type based on query pattern (selection, aggregation, grouping). It showed that no single index dominates all workloads, establishing the analytical basis for multi-index strategies in OLAP systems.

### What are the key ideas/main contributions? (2-3 sentences)
The paper defines three index types beyond B-trees: **bitmap indices** (one bitvector per distinct value, fast for low-cardinality predicates via bitwise AND/OR), **projection indices** (store raw column values in row-order for fast sequential scans), and **bit-sliced indices** (decompose column values bit-by-bit, enabling SUM via popcount and range predicates via bitwise arithmetic without touching every distinct value). The key insight is that different query types have different optimal indices: bit-sliced wins for SUM aggregation (O(bits) operations vs O(rows)), bitmap wins for low-cardinality GROUP BY, and projection wins for high-cardinality retrieval. The paper also introduces **Groupset indices** to accelerate joins between fact and dimension tables by pre-materializing grouping information.

![Aggregation Performance](/images/2026-03-17-readings-variant-indices/aggregation-index.png)
![Range Evaluation Performance](/images/2026-03-17-readings-variant-indices/range-index.png)


### What system was built/modified, and how was it evaluated? (1-2 sentences)
The authors built analytical cost models (counting I/O pages and CPU operations) and validated them against Sybase IQ, a commercial column-store that supported these index types. The evaluation used synthetic warehouse schemas to measure performance across varying selectivities and cardinalities.

### Judgement and follow-up ideas or open questions? (1-2 sentences)
The bit-level decomposition idea in bit-sliced indices anticipated modern techniques — SIMD bitwise operations and column-oriented compression (e.g., bit-packing in Parquet/Arrow). However, the cost models assume single-threaded, disk-based I/O patterns; modern in-memory columnar engines change the tradeoffs significantly. The framework of "match index type to query pattern" remains the right mental model, but the specific breakeven points have shifted.

### Evaluate my original writeup

| Aspect | Original | How to improve |
|---|---|---|
| **Importance** | "in data warehouse where workload dominant by read, building variant indexes can speed up OLAP queries" — too generic. Any index speeds up queries. | Name the *specific* contribution: a systematic comparison of three index types (bitmap, projection, bit-sliced) with cost models showing which wins for which query pattern. |
| **Key ideas** | Lists some results (bit-sliced good for SUM, range) but doesn't explain *why*. "I/O CPU cost" is a fragment, not a sentence. | Explain the mechanism: bit-sliced wins for SUM because it computes aggregates via popcount over N bit-planes instead of scanning all rows. Always answer *why*, not just *what*. |
| **System** | "no actual system was built" — partly correct but misleading. They built cost models and validated on Sybase IQ. | Say: "Built analytical cost models for I/O and CPU, validated against Sybase IQ." The cost models *are* the contribution. |
| **Judgement** | "bit slice index is largely replaced by SIMD" — conflates two things. SIMD is a hardware feature, not an index type. | Be precise: bit-sliced decomposition *anticipated* SIMD-friendly computation (popcount, bitwise ops). Modern column stores (Parquet bit-packing, Arrow dictionary encoding) inherit these ideas. |

**Recurring patterns:**
1. **Incomplete sentences and fragments** — "I/O CPU cost" isn't a thought. Write complete sentences even in notes.
2. **Listing results without mechanism** — You note bit-sliced is good for SUM but don't say *how* (popcount over bit-planes).
3. **Vague judgement** — "something still preserved" doesn't tell the reader anything. Name specific modern systems or techniques.

</details>
