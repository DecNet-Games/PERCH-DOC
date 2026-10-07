---
layout: default
title: "Performance & Profiling"
nav_order: 13
description: "Workload telemetry, scheduler budgets, GC profiling, profiler markers, and benchmark methodology."
permalink: /docs/performance/
---

# Performance & Profiling
{: .fs-9 }

### Hard CPU Budgets, Zero Managed GC Allocations & Profiler Markers
{: .fs-6 .text-grey-dk-000 }

---

Performance in PERCH is treated as an engineering contract rather than an afterthought. The system is architected to guarantee stable frame pacing and zero garbage collection spikes across high-density agent workloads.

---

## Performance Targets & Results

| Workload Metric | Target Limit | Measured Result | Status |
|:---|:---|:---|:---|
| **Steady-State GC Allocations** | **0 B / tick** | **0 B / tick** | **MET** |
| **300 Agents / 600 Slots CPU (p95)** | $\le 6.0\text{ ms}$ | **2.67 ms** | **MET** |
| **Lease Contention Batch Throughput** | $\le 1.0\text{ ms}$ | **0.18 ms** | **MET** |
| **Physics Query Budgeting** | Bounded Cap | Enforced via `MaxPhysicsQueriesPerTick` | **MET** |

---

## Profiler Markers & Deep Telemetry

PERCH instruments its internal systems with strongly typed `ProfilerMarker` scopes, allowing you to isolate exact timings in the Unity Profiler:

```
[Main Thread]
└── PerchWorld.Tick
    ├── SpatialHash.UpdateMovingPerches
    ├── LeaseManager.HeartbeatTick
    ├── CandidateSelector.ScoreCandidates
    │   ├── SpatialHash.QueryRadiusNonAlloc
    │   └── CandidateSelector.CoarseDistanceCull
    ├── PerchPlanner.EvaluateTrajectories
    │   ├── PerchTrajectory.ComputeHermiteLUT
    │   └── Physics.SphereCastNonAlloc
    └── PerchAgent.DrainEventQueue
```

---

## Recommended Benchmark Methodology

When measuring performance in your own project:
1. Always build and run a **Standalone Player** (Release configuration) rather than profiling inside the Unity Editor with deep profiling overhead.
2. Measure using actual production meshes, collision layers, and target hardware.
3. Record test environment metadata: Unity version, platform OS, CPU model, agent count, slot count, fixed timestep, warm-up period, and 95th-percentile frame time.
