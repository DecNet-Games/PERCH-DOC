---
layout: default
title: "Zero-Allocation Engineering"
parent: "Architecture"
nav_order: 3
description: "Zero managed GC allocation strategies, preallocated physics buffers, struct ring buffers, and GC profiling."
permalink: /docs/architecture/zero-allocation/
---

# Zero-Allocation Engineering
{: .fs-9 }

### Eliminating GC Spikes for High-Density Flocks at 60+ FPS
{: .fs-6 .text-grey-dk-000 }

---

In mobile, console, and VR titles, steady frame pacing is non-negotiable. Even small garbage collection allocations ($24\text{ B} - 64\text{ B}$ per agent per frame) quickly accumulate across a flock of 50 creatures, triggering frequent, jarring GC collection spikes.

PERCH is engineered from the ground up for **0 B steady-state managed heap allocations**.

---

## The 5 Pillars of Zero-GC Design

```
+-----------------------------------------------------------------------------------+
|                        ZERO-ALLOCATION RUNTIME STRATEGIES                         |
+-----------------------------------------------------------------------------------+
| 1. Non-Alloc Physics Buffers    ===> Physics.SphereCastNonAlloc with Static Arrays|
| 2. Zero-Closure Event Queues    ===> Fixed-Size Struct Ring Buffers (QueuedEvent) |
| 3. Precomputed LUT Trajectories ===> 32-Sample Preallocated Float Arrays          |
| 4. Value-Type Identity Structs  ===> readonly struct SlotHandle, LeaseToken       |
| 5. Spatial Hash Direct Mapping  ===> Pre-Sized Hash Buckets & Direct Int Dictionaries|
+-----------------------------------------------------------------------------------+
```

### 1. Preallocated Physics Buffers (DEC-006)
Unity's standard `Physics.SphereCastAll` allocates a new `RaycastHit[]` array on every single call. In PERCH:
* All spatial sweeps, approach checks, and departure tests use `Physics.SphereCastNonAlloc`.
* Query hit arrays are statically sized during initialization (`RaycastHit[64]` / `RaycastHit[256]`).
* If surrounding geometry saturates buffer capacity, the solver safely flags `QuerySaturated` rather than resizing on the heap.

### 2. Struct Circular Ring Buffers (DEC-010)
Standard C# `System.Action` closures and `List<T>` queues generate managed heap objects whenever lambdas or dynamic array resizes occur.
* Commands use a preallocated `PendingCommand[16]` ring buffer.
* Events use a preallocated `QueuedAgentEvent[32]` ring buffer.
* Adding, inspecting, and draining events operates purely on stack and fixed struct memory with zero heap activity.

### 3. Elimination of Event Closure Allocations (DEC-008)
In `PerchAgent.cs`, all UnityEvent invocations (`OnStateChanged`, `OnLanded`, `OnDeparted`, `OnLandingFailed`) are strictly guarded with null checks before creating invocation delegates.

### 4. Value-Type Data Layout
Core data structures cross subsystem boundaries as unmanaged C# `readonly struct` values:
* `SlotHandle`: 16 bytes.
* `LeaseToken`: 24 bytes.
* `MotionCommand`: 36 bytes.
* `PerchSnapshot`: 32 bytes.
These structs live entirely on the stack, passing between methods by value or `in` reference without creating garbage collector pressure.

### 5. Pre-Allocated Arc-Length LUTs (DEC-004)
Dynamic numerical integration of spline arc lengths per frame causes significant GC churn. PERCH preallocates a fixed 32-sample float array for every trajectory candidate, reusing the buffer across evaluations.

---

## Profiler Verification Evidence

PERCH's zero-allocation design is verified in the Unity Profiler:

| Simulation Workload | Active Agents | Authored Slots | GC Alloc / Frame | Profiler Status |
|:---|:---|:---|:---|:---|
| **Free Flight Roaming** | 100 Birds | 200 Slots | **0 B** | PASSED |
| **Simultaneous Approaches** | 24 Birds | 48 Slots | **0 B** | PASSED |
| **Panic Wave Takeoff** | 50 Birds | 100 Slots | **0 B** | PASSED |
| **Stress Benchmark** | 300 Agents | 600 Slots | **0 B** | PASSED |
