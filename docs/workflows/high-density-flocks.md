---
layout: default
title: "High-Density Flocks (300+ Birds)"
parent: "Workflows & Recipes"
nav_order: 2
description: "Optimization guidelines, scheduler budgets, decision jitter, and LOD strategies for large-scale flocks."
permalink: /docs/workflows/high-density-flocks/
---

# High-Density Flocks (300+ Birds)
{: .fs-9 }

### Architecting Massive Ambient Flocks Without Frame Rate Drops
{: .fs-6 .text-grey-dk-000 }

---

Simulating a flock of 5 birds is trivial; simulating 100 to 300 birds navigating a dense medieval city or fantasy forest requires strict budgeting and performance isolation. If 300 birds all fire physics raycasts on the exact same frame, frame time can spike past 30 ms.

PERCH is designed to handle high-density flock simulations smoothly at 60+ FPS.

---

## 1. Tuning the `PerchWorld` Scheduler

The master scheduler in `PerchWorld` enforces bounded CPU workloads:

```csharp
// PerchWorld Scheduler Settings
MaxPhysicsQueriesPerTick = 64;   // Hard cap on SphereCasts / tick
MaxCandidateScansPerTick = 16;   // Max slots scored / tick
SpatialHashCellSize = 3.0f;      // Grid cell size in meters
```

* **Query Budgeting**: By capping `MaxPhysicsQueriesPerTick`, PERCH distributes physics sweeps across consecutive frames rather than executing a burst on a single tick.
* **Non-Blocking Slices**: If the query budget is reached during a frame, pending evaluations safely pause and resume on the next tick without resetting agent states.

---

## 2. Staggered Decision Jitter

If 100 birds are spawned simultaneously, their internal timers would normally fire on the exact same frame interval.

PERCH applies **Decision Interval Jitter**:
$$\Delta t_{\text{decision}} = T_{\text{base}} \cdot (1.0 \pm 0.2)$$
* With a 1.0s base interval, decisions are staggered uniformly between $0.8\text{s}$ and $1.2\text{s}$.
* This flattens the CPU workload curve into a near-constant, smooth line across all frames.

---

## 3. Distance Culling & LOD Strategies

In open-world games, birds 100 meters away do not need the same level of simulation fidelity as a bird landing 2 meters in front of the camera:

| Distance Tier | Distance Range | Optimization Applied |
|:---|:---|:---|
| **Tier 1 (Near)** | $0\text{m} - 25\text{m}$ | Full Hermite spline, swept obstacle checks, two-bone IK feet alignment. |
| **Tier 2 (Mid)** | $25\text{m} - 60\text{m}$ | Full spline trajectory; disable two-bone IK solver (mesh foot placement imperceptible). |
| **Tier 3 (Far)** | $> 60\text{m}$ | Coarse distance culling rejects slots immediately ($>50\text{m}$); creatures maintain free-flight roaming. |

> [!IMPORTANT]
> **Off-Screen Occupancy Guarantee**: Even if a perched creature is culled visually or off-screen, PERCH **never releases its slot lease or exclusion volume**. When the player turns the camera back, the bird is still resting firmly on its branch, preserving world persistence.

---

## Benchmark Evidence: 300 Agents / 600 Slots

Using the included `05_Benchmark.unity` scene:
* **Active Agents**: 300
* **Registered Slots**: 600
* **95th-Percentile Frame Time**: **2.67 ms**
* **Managed GC Allocations**: **0 B / frame**
