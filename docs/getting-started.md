---
layout: default
title: Getting Started
nav_order: 2
description: "Core architectural concepts, component relationships, and creature landing lifecycle in PERCH."
permalink: /docs/getting-started/
---

# Getting Started
{: .fs-9 }

### Core Concepts, System Architecture & The 7-Phase Landing Lifecycle
{: .fs-6 .text-grey-dk-000 }

---

PERCH is designed around a single guiding philosophy: **predictable, physically bounded behavior under all runtime conditions.** Rather than relying on fuzzy steering forces or unconstrained raycast snaps, PERCH coordinates flying creatures through a clear contract between spatial registries, atomic reservations, trajectory solvers, and animation drivers.

![Complete Flight-to-Perch Cycle]({{ site.baseurl }}/assets/images/02_Complete_Cycle.png)

---

## Core Components Overview

The PERCH runtime is partitioned into decoupled subsystems. Understanding how these components communicate is key to integrating PERCH into your game:

```
                                  +-------------------+
                                  |    PerchWorld     |
                                  | (Spatial Hash 3D) |
                                  |  (LeaseManager)   |
                                  +---------+---------+
                                            |
                         +------------------+------------------+
                         |                                     |
                         v                                     v
               +-------------------+                 +-------------------+
               |    PerchAgent     |                 |     PerchSpot     |
               | (State Machine)   |                 | (Authored Slots)  |
               +---------+---------+                 +---------+---------+
                         |                                     |
        +----------------+----------------+                    |
        |                |                |                    v
        v                v                v          +-------------------+
+---------------+ +---------------+ +---------------+|     PerchSlot     |
| CreatureProfile| |  PerchMotor  | | FlightProvider|| (Contact Frame)   |
+---------------+ +---------------+ +---------------+ +-------------------+
```

### 1. `PerchWorld`
The master coordinator for a given physical environment. 
* Owns the 3D spatial hash index (`SpatialHash3D`) storing all active perches.
* Runs the atomic `LeaseManager` to prevent simultaneous slot claims.
* Maintains the simulation clock (`SimulationClock`), enabling pause/resume, time scaling, and deterministic integration tests.
* Implements the per-tick physics and candidate scan scheduler.

> [!NOTE]
> You can run multiple independent `PerchWorld` instances in the same Unity project (e.g., in multi-scene or additive scene configurations). Agents and spots assigned to different worlds will never interact or exchange leases.

### 2. `PerchSpot` and `PerchSlot`
* **`PerchSpot`**: A MonoBehaviour attached to scene geometry (such as a tree branch, telephone wire, stone fence, or ship railing). It tracks whether the support is stationary or moving and manages a collection of slots.
* **`PerchSlot`**: A lightweight serializable struct representing an individual landing position. Each slot defines:
  * Local contact position and rotation (where Y is the surface normal and Z is the creature's facing forward direction).
  * Associated physical support collider.
  * Contact mode (`Point`, `Line`, or `Patch`).
  * Reserved exclusion footprint radius (to prevent wings overlapping neighbors).
  * Accepted creature styles and tags.

### 3. `PerchAgent`
The brain attached to the creature root.
* Manages the 10-state deterministic finite state machine (FSM).
* Processes landing commands (`RequestLanding`, `RequestSlot`, `Depart`, `CancelLanding`).
* Governs motor ownership handover between free roaming and approach curves.
* Emits strongly typed events (`OnStateChanged`, `OnLanded`, `OnDeparted`, `OnLandingFailed`).

### 4. `PerchCreatureProfile`
An immutable `ScriptableObject` defining physical and aerodynamic capabilities:
* Physical dimensions: body length, body radius, contact span, clearance margins.
* Flight kinematics: cruise speed, approach speed, touchdown speed tolerance, max acceleration, max deceleration, turn rate.
* Moving perch limits: maximum platform linear speed, angular speed, and acceleration.
* Dwell and retry timings: min/max perch dwell duration, retry cooldowns, recent-slot penalties.

### 5. `IPerchFlightProvider` & `IPerchMotor`
* **`IPerchFlightProvider`**: Drives the agent during free flight roaming. PERCH ships with `PerchWaypointFlightProvider`, but you can plug in any custom steering, boids, or navigation AI.
* **`IPerchMotor`**: The single component responsible for writing root transform movements. Built-in implementations include `TransformMotor` and `KinematicRigidbodyMotor`.

---

## The 7-Phase Creature Landing Lifecycle

Every perch-and-depart cycle executes through seven deterministic stages:

```
[1. Free Flight] ──> [2. Selection] ──> [3. Planning] ──> [4. Approach]
                                                               │
[7. Takeoff]     <── [6. Dwell & Fold] <── [5. Touchdown] <────┘
```

### Phase 1: Free Flight Roaming (`FreeFlight`)
The creature moves freely through the environment under the control of its `IPerchFlightProvider`. During this phase, PERCH's motor is inactive (`MotionOwner.ExternalProvider`). The agent enforces a configurable minimum flight duration before auto-landing policy triggers.

### Phase 2: Candidate Selection (`Selecting`)
When an automatic decision interval elapses (or an explicit API call occurs), the agent queries the `PerchWorld` spatial hash:
1. **Hard Filtering**: Discards spots with mismatched creature styles, tags, or dimension limits.
2. **Coarse Distance Culling**: Immediately rejects spots beyond the 50-meter operating envelope.
3. **Suitability Scoring**: Evaluates reachability, distance, approach alignment, and recent-use penalties.
4. **Tie-Breaking**: Uses seeded, deterministic evaluation to select the best candidate.

### Phase 3: Trajectory Planning & Lease Reservation (`Planning`)
Before moving toward the spot, PERCH guarantees safety:
* **Atomic Claim**: The `LeaseManager` checks whether the slot and its neighboring footprint are free. If so, a lease token is granted with a monotonic request ID.
* **Hermite Spline Generation**: An approach spline is calculated using arc-length look-up tables.
* **Swept Clearance Check**: SphereCast sweeps verify that external obstacles do not obstruct the approach corridor or the initial departure path.
* If any check fails, the lease is released cleanly and the agent returns to `FreeFlight` on cooldown.

### Phase 4: Swept Approach & Physical Deceleration (`Approaching`)
* PERCH acquires motor control (`MotionOwner.PerchMotor`).
* The creature follows the arc-length parameterization, decelerating smoothly from cruise speed to approach speed.
* Periodic heartbeats maintain the lease.
* If a dynamic obstacle moves into the swept corridor or the target platform accelerates beyond safe limits, the agent initiates an `AbortRecovery`.

### Phase 5: Contact Solve, Touchdown & Stance Lock (`Touchdown`)
* As the creature crosses the final arrival boundary, the solver aligns the root to the slot's contact frame.
* The optional `PerchTwoBoneIkSolver` or leg stance markers engage, pinning talons/feet firmly to the collider geometry.
* Relative contact speed drops below the profile's touchdown tolerance (default $\le 0.25\text{ m/s}$).
* The slot transitions from `Reserved` to `Occupied`.

### Phase 6: Perched Dwell & Wing Fold Hold (`Perched`)
* The creature holds its stance on the perch.
* The animation system triggers wing folding (via Mecanim crossfades, Playables stance holds, or procedural harmonic damping).
* Subtle breathing and balance bobbing can be applied while feet remain anchored.
* If the perch is on a moving platform (e.g., a boat railing or swinging branch), the agent tracks the platform's local coordinate frame.

### Phase 7: Safe Takeoff & Momentum Handover (`TakingOff`)
* When the dwell timer expires, or upon receiving a manual `Depart()` or panic threat wave:
* Departure sweeps verify that the takeoff corridor is free of obstacles.
* Wings unfold and flapping initiates before root translation begins.
* The agent moves forward/upward, holding its exclusion footprint until the body completely clears the perch area.
* Once clear, the slot is released back to `Free`, and motion control is cleanly handed back to `IPerchFlightProvider` with inherited departure velocity.

---

## Next Steps

* **Ready to install?** Head to [Installation & Requirements]({{ site.baseurl }}/docs/installation/).
* **Want your first bird flying in 5 minutes?** Check out the [Quick Start Guide]({{ site.baseurl }}/docs/quick-start/).
* **Deep dive into features?** Explore [Core Features]({{ site.baseurl }}/docs/features/).
