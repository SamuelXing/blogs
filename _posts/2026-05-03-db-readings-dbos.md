---
title: 'Consistency and Correctness in Data-Oriented Workflow Systems'
date: 2026-05-03
published: true
tags:
  - databases
  - workflow
---

The paper extends DBOS's database-oriented OS philosophy to workflow systems, arguing that workflow correctness should be expressed in classical database terms (transactions, atomicity, isolation) rather than ad-hoc retry/compensation logic. It formalizes what "correct" means for workflows via the AC/DC properties (Atomic, Consistent, Durable, Correct) and shows that physical-transaction-based approaches have measurable downsides versus saga-based compensation.


Link: [dbos-consistency.pdf](https://www.vldb.org/cidrdb/papers/2026/p9-stonebraker.pdf)

<details markdown="1">
<summary><b>Detailed Notes</b></summary>

### Why is this paper important? (1-2 sentences)
The paper extends DBOS's database-oriented OS philosophy to workflow systems, arguing that workflow correctness should be expressed in classical database terms (transactions, atomicity, isolation) rather than ad-hoc retry/compensation logic. It formalizes what "correct" means for workflows via the AC/DC properties (Atomic, Consistent, Durable, Correct) and shows that physical-transaction-based approaches have measurable downsides versus saga-based compensation.

### What are the key ideas/main contributions? (2-3 sentences)
The paper's central observation is that "durable execution" frameworks (Temporal, AWS Step Functions, etc.) only handle the happy path — they replay forward after a crash but don't define what correct error handling means. It proposes the AC/DC correctness model and contrasts two implementations: 
(1) Physical Backout, which wraps the entire multi-step workflow in a single long-running database transaction so that aborts roll back automatically — simple but holds locks across the workflow's lifetime, hurting throughput; 
and (2) the Saga pattern, where each step has a programmer-supplied compensation function that semantically undoes its effects, allowing locks to release after each step at the cost of more developer effort. 
The DBOS implementation logs both user operations and workflow state changes in the same database transaction, so workflow durability comes "for free" from existing DB ACID guarantees.


### What system was built/modified, and how was it evaluated? (1-2 sentences)
The system is the commercial DBOS platform with workflow extensions implementing both Physical Backout and Saga modes. The evaluation compares the two on throughput, abort rate, and latency under contention — saga wins broadly because compensation lets locks release between steps.

### Judgement and follow-up ideas or open questions? (1-2 sentences)
The argument that workflow systems should be built into the database rather than layered on top is consistent with Stonebraker's broader thesis (databases as the right abstraction for many systems problems). The interesting tension is that saga's correctness depends on programmer-written compensations being semantically correct — the system can guarantee they run, but not that they undo the right things. So the paper trades a hard system-level guarantee (Physical Backout's transaction) for a softer human-level one (Saga's correctness-of-compensation). Open questions: what about distributed workflows spanning multiple DBOS instances? How does this interact with external side effects (sending emails, calling APIs) that can't be wrapped in DB transactions?

</details>
