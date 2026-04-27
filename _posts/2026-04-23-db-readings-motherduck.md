---
title: 'MotherdDuck: DuckDB in the cloud and in the client'
date: 2026-04-23
published: true
tags:
  - databases
  - code generation
---



Link: [motherduck.pdf](https://www.cidrdb.org/cidr2024/papers/p46-atwal.pdf)

<details markdown="1">
<summary><b>Detailed Notes</b></summary>

### Why is this paper important? (1-2 sentences)
MotherDuck challenges the prevailing scale-out orthodoxy in cloud analytics by arguing that most analytical workloads fit on a single beefy node. It introduces a hybrid architecture where queries can execute partially on the client (laptop) and partially in the cloud (a "duckling" container), with seamless data exchange — making it one of the first serious attempts to blur the client/cloud boundary in OLAP.

### What are the key ideas/main contributions? (2-3 sentences)
The paper's core thesis rests on two empirical observations: ~95% of databases are <1TB and ~95% of queries scan <10GB, so scale-out is overkill for most analytics. MotherDuck's architecture has three pieces: (1) ephemeral cloud compute ("ducklings" — short-lived DuckDB containers on demand, no always-on cluster), (2) shared distributed storage (data lives on cloud object storage, accessible to both client and ducklings), and (3) hybrid query execution via bridge operators — placed in the query plan to ship tuple streams between local and remote execution sites. The optimizer chooses the placement (local vs. cloud) per operator based on data location and resource availability, treating the network bridge as just another cost in the plan.


### What system was built/modified, and how was it evaluated? (1-2 sentences)
MotherDuck is a commercial system built on top of DuckDB, evaluated against itself in different configurations and against cloud DWH baselines on TPC-H. The 5-10x speedup on larger scales comes from cloud bandwidth to S3 (faster than typical home internet) plus more cores in the duckling, not from DuckDB algorithmic changes — they're effectively offloading I/O-bound work to where the data is.

### Judgement and follow-up ideas or open questions? (1-2 sentences)
The "small data" thesis is empirically strong and the hybrid model is a genuinely novel architectural point — most cloud DBs assume queries run entirely in the cloud or entirely locally. The biggest open questions are around the optimizer: how to estimate bridge costs accurately, how to handle dynamic placement when network conditions change mid-query, and how this composes with iterative/interactive workloads where intermediate results may need to flow both directions multiple times. The reliance on a single-node engine also caps the upper end — at some point you do need to scale out, and the paper doesn't deeply address that boundary.

</details>
