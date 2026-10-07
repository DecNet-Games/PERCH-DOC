---
layout: default
title: "Atomic Lease Management"
parent: "Core Features"
nav_order: 2
description: "Deterministic atomic slot reservations, heartbeat TTL, exclusion footprints, and multi-agent contention guarantees."
permalink: /docs/features/lease-management/
---

# Atomic Lease Management
{: .fs-9 }

### Deterministic Reservations & Spatial Exclusion for Multi-Agent Flocks
{: .fs-6 .text-grey-dk-000 }

---

In complex game scenes with tens or hundreds of birds, race conditions are the primary cause of broken AI: two agents simultaneously choose the same branch, plan overlapping flight trajectories, and collide or clip through each other on touchdown.

PERCH completely eliminates race conditions through the **`LeaseManager`**, an atomic reservation engine executing on the main simulation tick.

![Spatial Reservation Top View]({{ site.baseurl }}/assets/images/Rig_TopView.png)

---

## The Slot Lease Lifecycle

Every slot in a `PerchWorld` transitions through five formal lifecycle states:

```
    +-------------------------------------------------------+
    |                                                       |
    v                                                       |
+--------+       +------------+       +------------+        |
|  Free  | ----> |  Reserved  | ----> |  Occupied  |        |
+--------+       +------------+       +------------+        |
    ^                   |                   |               |
    |                   v                   v               |
    |            [Abort / Cancel]     +------------+        |
    +-------------------------------- | Departing  | -------+
                                      +------------+
```

| Lifecycle State | Description | Invariants |
|:---|:---|:---|
| **`Free`** | The slot is unreserved and available for candidate selection. | Eligible for scoring and lease acquisition. |
| **`Reserved`** | An agent has claimed the slot and is currently planning or executing its approach. | Blocked from all other agents. Heartbeat TTL active. |
| **`Occupied`** | The creature has successfully touched down and locked its stance on the perch. | Held indefinitely until explicit dwell timeout, departure command, or threat panic. |
| **`Departing`** | The creature has launched into takeoff flight, but its body has not yet cleared the exclusion footprint. | Cannot be claimed by incoming birds until the departing body completely vacates the physical exclusion volume. |

---

## Atomic Claim Architecture

Conventional Unity AI systems often use a two-step check:
```csharp
// THE WRONG WAY (Naive Check-Then-Claim Race Condition)
if (spot.IsAvailable) {
    // If another bird executed this same check in the same frame,
    // both birds think the slot is free and proceed to collide!
    spot.Claim(this); 
}
```

In PERCH, claim evaluation and state transition are strictly atomic:

```csharp
// THE PERCH WAY (Atomic Main-Thread Transition)
public bool TryAcquireLease(SlotHandle handle, PerchAgent agent, out LeaseToken token)
```

Within a single atomic invocation:
1. Validates the slot's generation and current state.
2. Evaluates the spatial hash for neighboring exclusion volume overlaps.
3. If clear, transitions state immediately from `Free` to `Reserved`.
4. Mints a unique `LeaseToken` containing a monotonic sequence number, agent ID, and generation count.
5. Returns `true` with the valid token, or `false` with a descriptive failure code (`Occupied` or `FootprintConflict`).

This design guarantees that even if 50 agents simultaneously request the same branch in the exact same frame, exactly one agent succeeds and 49 agents receive clean, non-blocking failure reasons to immediately select alternative perches.

---

## Token Identity & Integrity

To protect against stale callbacks, destroyed objects, and scene reloads, PERCH uses value-type identity structs:

```csharp
public readonly struct SlotHandle
{
    public readonly int WorldId;
    public readonly int RuntimeSpotId;
    public readonly int SlotId;
    public readonly int Generation; // Incremented every time a spot is edited
}

public readonly struct LeaseToken
{
    public readonly SlotHandle Slot;
    public readonly int AgentId;
    public readonly int AgentGeneration;
    public readonly uint LeaseSequence; // Monotonically increasing counter
}
```

If a spot is destroyed, regenerated, or moved, its `Generation` counter increments. Any incoming agent holding a stale `SlotHandle` is immediately rejected with `SpotUnavailable` or `LeaseLost`, preventing phantom object references.

---

## Heartbeat Time-to-Live (TTL)

During the `Approaching` state, what happens if an agent is destroyed, disabled, or encounters a scripting exception mid-flight?

To prevent orphaned reservations from locking perches indefinitely, PERCH enforces a **Heartbeat TTL**:
* **Default TTL**: `2.0 seconds`.
* **Heartbeat Interval**: The approaching agent automatically sends a heartbeat tick every `0.25 seconds`.
* **Expiry Recovery**: If an agent fails to send a heartbeat within 2.0 seconds (e.g., due to sudden deactivation or external script destruction), the `LeaseManager` revokes the lease, cleanses the spatial hash, and resets the slot to `Free`.

> [!NOTE]
> TTL expiration applies **only** to the `Reserved` state. Once a creature reaches `Occupied`, it cannot expire. The perch remains occupied until the creature explicitly departs or is despawned.

---

## Footprint Exclusion & Spatial Hash Mapping

PERCH prevents wing clipping across adjacent slots using a direct $O(1)$ spatial mapping (`_spatialIdToHandle`):

1. Every slot registers its `ReservedFootprintRadius` in the `PerchWorld` spatial hash grid.
2. During lease acquisition, a spatial sphere query checks neighboring grid cells for existing leases.
3. Rather than iterating linearly through all scene slots ($O(N^2)$), the direct dictionary map resolves cell occupants in bounded $O(1)$ time.
4. This optimization allows PERCH to process multi-agent contention benchmarks with 600 slots and 300 active birds in sub-millisecond frame times.

---

## Contention Verification Evidence

The atomic lease architecture is validated through automated multi-threaded and frame-batch test suites:
* **Stress Test**: `1,000` randomized contention batches executed under maximum agent load.
* **Result**: **0 simultaneous conflicting owners**, **0 deadlocks**, **0 orphan leases**.
