---
layout: default
title: "Motor Ownership & Handover"
parent: "Architecture"
nav_order: 2
description: "The single-writer transform invariant, MotionOwner states, and seamless handover contracts."
permalink: /docs/architecture/motor-ownership/
---

# Motor Ownership & Handover
{: .fs-9 }

### The Single-Writer Invariant for Flawless Motion Continuity
{: .fs-6 .text-grey-dk-000 }

---

One of the most persistent bugs in character AI is **transform fighting**: an AI steering script and a landing script simultaneously trying to write to the character's `transform.position`. The result is violent visual jitter, erratic physics collisions, and broken navigation.

PERCH enforces a strict **Single-Writer Transform Invariant** governed by the `MotionOwner` state.

---

## The `MotionOwner` Enum

At any given microsecond, exactly one system possesses the authority to displace the creature's root:

```csharp
public enum MotionOwner
{
    ExternalProvider, // Wandering/Navigation AI owns movement
    PerchMotor,       // PERCH owns approach, touchdown, anchoring, takeoff
    InterventionHold  // Safety freeze during entrapment
}
```

```
[Free Roaming]                     [Landing Approach & Dwell]
External Flight AI  ──(Handover)──>  PERCH Motor (Trajectory & IK)
(MotionOwner.ExternalProvider)        (MotionOwner.PerchMotor)
        ^                                      |
        +───────────(Takeoff Handover)─────────+
```

---

## The Handover Contract (`IPerchFlightProvider`)

When PERCH transitions between free flight and landing approaches, it executes a formal handover handshake with the creature's flight provider:

### 1. Acquiring Control (Flight $\rightarrow$ Approach)
1. PERCH queries `provider.CanYield(agent)`. If the creature is in an unyielding state (e.g. playing a death animation or stunned), handover is declined with `ProviderRejected`.
2. PERCH calls `provider.CaptureMotion(agent)` to record the creature's entry position, velocity vector, and forward heading.
3. PERCH calls `provider.OnPerchControlAcquired(agent)`. The provider suspends its internal steering updates.
4. `MotionOwner` switches to `MotionOwner.PerchMotor`.

### 2. Releasing Control (Takeoff $\rightarrow$ Flight)
1. As the creature completes its departure curve and clears the perch footprint:
2. PERCH computes the final exit position $\mathbf{P}_{\text{exit}}$ and departure velocity vector $\mathbf{V}_{\text{exit}}$ (including platform momentum).
3. PERCH calls `provider.OnPerchControlReleased(agent, exitPosition, exitVelocity)`.
4. The external provider initializes its steering vectors from the supplied exit velocity, eliminating any sudden direction snaps.
5. `MotionOwner` reverts to `MotionOwner.ExternalProvider`.

---

## Shipped Motor Implementations

PERCH provides two optimized motor components out of the box:

### 1. `TransformMotor`
* **Target**: Creatures that do not use physical Rigidbody dynamics.
* **Mechanism**: Writes directly to `transform.position` and `transform.rotation` during `Update()`.
* **Performance**: Sub-microsecond execution; ideal for large background flocks of songbirds.

### 2. `KinematicRigidbodyMotor`
* **Target**: Characters interacting with complex physics colliders or moving physics triggers.
* **Mechanism**: Displaces the root using `Rigidbody.MovePosition()` and `Rigidbody.MoveRotation()` during `FixedUpdate()`.
* **Stability**: Eliminates physics penetration tunneling against moving environment colliders.
