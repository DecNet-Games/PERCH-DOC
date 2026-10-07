---
layout: default
title: "Threading & Determinism"
parent: "Architecture"
nav_order: 6
description: "Deterministic simulation clocks, main-thread atomic locks, seeded randomness, and replayability."
permalink: /docs/architecture/threading-and-determinism/
---

# Threading & Determinism
{: .fs-9 }

### Main-Thread Atomic Coordination & Fixed Simulation Clocks
{: .fs-6 .text-grey-dk-000 }

---

Multithreading in game character AI often introduces more bugs than it solves: race conditions, deadlocks when accessing Unity GameObjects, and non-deterministic playback across machines.

PERCH balances extreme execution speed with total determinism through **Main-Thread Atomic Coordination** and **Pluggable Simulation Clocks**.

---

## 1. Main-Thread Atomic Coordination (DEC-003)

Rather than distributing slot reservation logic across asynchronous worker threads that risk race conditions:
* All lease requests and spatial reservations execute on the **main simulation tick**.
* Because execution is single-threaded and atomic, 1,000 randomized contention batches pass deterministically without needing mutexes, spinlocks, or monitor locks.
* Spatial lookups and candidate scoring are optimized with $O(1)$ spatial hashing and direct dictionaries, executing in sub-millisecond durations without offloading to jobs.

---

## 2. The Pluggable Simulation Clock (`IPerchClock`)

PERCH decouples simulation time from Unity's static `Time.deltaTime` via the `IPerchClock` interface:

```csharp
public interface IPerchClock
{
    float Time { get; }
    float DeltaTime { get; }
    bool IsPaused { get; }
}
```

* **`SimulationClock`**: The default implementation in `PerchWorld`. Supports global pause, slow-motion time dilation, and custom fixed timesteps.
* **Deterministic Automated Tests**: Unit tests inject a mock `TestClock` that advances time in discrete, exact steps, allowing integration tests to run headless without depending on system frame rates.

---

## 3. Seeded Randomness & Cinematics Replay

To ensure identical behavior across test suites and narrative cutscenes:
* Decision jitter, candidate tie-breaking, and dwell timers sample from an internal `System.Random` initialized with `PerchWorld.SimulationSeed`.
* In cutscenes, setting a fixed seed guarantees identical flight paths and perching sequences across repeated takes.
