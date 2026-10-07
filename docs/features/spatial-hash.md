---
layout: default
title: "3D Spatial Hash Partitioning"
parent: "Core Features"
nav_order: 7
description: "Bounded O(1) spatial queries, 3D hash grids, coarse distance culling, and zero-allocation radius checks."
permalink: /docs/features/spatial-hash/
---

# 3D Spatial Hash Partitioning
{: .fs-9 }

### High-Speed O(1) Spatial Lookups for Large-Scale Flocks
{: .fs-6 .text-grey-dk-000 }

---

When managing dozens of creatures searching for hundreds of potential landing perches across a massive level, spatial lookup efficiency is paramount. A linear scan across 600 slots for 300 birds requires up to 1.8 million distance checks per frame—triggering severe CPU frame drops. Octrees eliminate some comparisons but require dynamic pointer structures that trigger GC allocations when perches move.

PERCH solves spatial indexing with **`SpatialHash3D`**, a flat, high-speed 3D spatial hashing structure optimized for zero-allocation query cycles.

---

## The Spatial Hash Architecture

`SpatialHash3D` partitions 3D world space into discrete volumetric cells of uniform size (default: $2.0\text{ m}$):

```
       +-------+-------+-------+
      /       /       /       /|
     +-------+-------+-------+ |
    /       / Cell  /       /| +
   +-------+ [2.0m]+-------+ |/|
   |       |   *   |       | + |
   |       | (Slot)|       |/| +
   +-------+-------+-------+ |/
   |       |       |       | +
   +-------+-------+-------+
```

### The Hash Function
Given a 3D coordinate $(x, y, z)$ and cell size $C$:

$$\text{Index} = \left( \lfloor x/C \rfloor \cdot p_1 \oplus \lfloor y/C \rfloor \cdot p_2 \oplus \lfloor z/C \rfloor \cdot p_3 \right) \pmod{\text{TableCapacity}}$$

Where $p_1 = 73856093$, $p_2 = 19349663$, and $p_3 = 83492791$ are large prime constants that produce uniform distribution across the hash table with near-zero hash collisions.

---

## O(1) Direct ID-to-Handle Mapping (DEC-012)

In high-density scenes, checking whether an adjacent slot's exclusion footprint overlaps a prospective landing spot can easily degenerate into an $O(N^2)$ bottleneck if handles must be looked up by searching an array.

PERCH implements a direct **`_spatialIdToHandle` mapping** in `LeaseManager`:
* Every spatial entry in the grid stores an integer ID.
* The dictionary resolves the corresponding `SlotHandle` in guaranteed $O(1)$ time.
* In benchmark scenes with 600 slots, this optimization reduced footprint contention evaluation time by **8x**.

---

## Two-Tiered Coarse Culling (DEC-013)

Before running expensive mathematical evaluations (such as Hermite spline feasibility, angle-axis math, and surface clearance checks), PERCH applies a two-tiered culling filter:

1. **Coarse Distance Culling**: Rejects any slot whose squared distance from the creature exceeds $50^2 = 2,500\text{ m}^2$ using a single subtraction and dot product.
2. **Spatial Hash Bucket Query**: Queries only the immediate 27 neighboring cells ($3 \times 3 \times 3$ grid) around the creature's query sphere.

```csharp
// High-efficiency radius query without heap allocations
int hitCount = world.SpatialHash.QueryRadiusNonAlloc(agentPosition, searchRadius, queryResultsBuffer);
```

---

## Memory & Allocation Guarantees

* **Preallocated Internal Buffers**: All bucket arrays and hit list buffers are sized during `PerchWorld.Initialize()`.
* **Zero GC Heap Allocations**: Querying spots, testing footprint overlaps, and updating moving spot positions generate **0 B** managed heap allocations during steady-state gameplay ticks.
* **Benchmark Evidence**: 300 concurrent agents querying 600 active slots maintain a 95th-percentile frame processing time of **2.67 ms**—comfortably within the 6.0 ms budget for 60 FPS titles.
