---
layout: default
title: Architecture
nav_order: 6
has_children: true
description: "Core software engineering principles, assembly boundaries, state machines, and zero-allocation runtime design."
permalink: /docs/architecture/
---

# System Architecture
{: .fs-9 }

### Engineering Principles, Assembly Partitions & Invariant Guarantees
{: .fs-6 .text-grey-dk-000 }

---

PERCH is designed to meet strict commercial game engine performance standards. It adheres to formal software engineering patterns: strict assembly partitioning, single-writer transform ownership, reentrancy guards, and zero managed garbage collection allocations during steady-state ticks.

---

## Architecture Topics

Explore PERCH's low-level engine internals:

### 1. [Deterministic State Machine]({{ site.baseurl }}/docs/architecture/state-machine/)
The complete 10-state lifecycle governing creature states from `Disabled` through `FreeFlight`, `Approaching`, `Perched`, to `TakingOff`. Includes state transition invariants and reentrancy protection.

### 2. [Motor Ownership & Handover]({{ site.baseurl }}/docs/architecture/motor-ownership/)
The single-writer transform invariant. How control smoothly transfers between external AI steering providers (`ExternalProvider`) and PERCH's trajectory motors (`PerchMotor`).

### 3. [Zero-Allocation Engineering]({{ site.baseurl }}/docs/architecture/zero-allocation/)
How PERCH achieves a steady-state 0 B GC allocation profile across 300+ agents using preallocated struct circular buffers, static physics query arrays, and non-allocating delegate patterns.
