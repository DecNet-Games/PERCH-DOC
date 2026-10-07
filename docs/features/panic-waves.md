---
layout: default
title: "Threats & Panic Sequences"
parent: "Core Features"
nav_order: 8
description: "Coordinated flock panic waves, PerchThreatSource, departure clearance, and safe occupancy release invariants."
permalink: /docs/features/panic-waves/
---

# Threats & Panic Sequences
{: .fs-9 }

### Coordinated Flock Dispersion with Zero Collision or Mesh Clumping
{: .fs-6 .text-grey-dk-000 }

---

Nothing shatters game immersion faster than firing a gunshot near a flock of perched birds only to watch them snap instantly into flight, fly through surrounding tree branches, or have incoming birds land directly on top of fleeing ones.

PERCH provides a dedicated **Threat & Panic Architecture** via `PerchThreatSource` that guarantees cinematic flock dispersion while upholding strict physical invariants.

![Flock Takeoff Sequence]({{ site.baseurl }}/assets/images/06_Takeoff.png)

---

## The `PerchThreatSource` Component

Attach `PerchThreatSource` to any dynamic scene object that should scatter wildlife—such as the player character, predator beasts, vehicles, or explosive barrels:

```csharp
using Decnet.Perch;
using UnityEngine;

public class ExplosiveBarrel : MonoBehaviour
{
    [SerializeField] private PerchThreatSource threatSource;

    public void Detonate()
    {
        // Broadcasts panic wave across all agents within threat radius
        threatSource.TriggerPanicWave();
    }
}
```

### Key Parameters:
* **`ThreatRadius`**: Sphere of influence within which perched creatures will detect the disturbance.
* **`ReactionDelayJitter`**: Random timing variance ($0.05\text{s} - 0.25\text{s}$) applied per creature so the flock disperses in a staggered, organic burst rather than a synchronized robotic flash.
* **`LineOfSightOcclusion`**: Optional raycast check to prevent creatures from panicking through solid stone walls or terrain obstacles.

---

## Panic Wave Lifecycle

When a panic wave triggers, each affected creature executes a deterministic sequence:

```
[Threat Detected] ──> [Staggered Delay] ──> [Departure Sweep] ──> [Flap & Launch] ──> [Clear Footprint & Release Lease]
```

1. **Evaluation**: Perched agents within the threat sphere receive the threat notification.
2. **Clearance Check**: The agent executes a forward-upward sphere sweep to verify that its departure corridor is unobstructed. If an overhead obstacle blocks the immediate exit path, the agent re-evaluates an alternate angled escape vector.
3. **Flap Initiation**: The creature triggers its takeoff animation (wings unfold and flap) before root translation commences.
4. **Footprint Holding**: The agent launches into the air. **Crucially, the slot lease remains held** while the creature's body is within the perching zone.
5. **Clean Handover**: Once the creature's bounding volume completely vacates the slot's `ReservedFootprintRadius`, the lease is atomically transitioned back to `Free`, and free flight roaming resumes.

---

## Core Safety Invariants

> [!IMPORTANT]
> **No Premature Slot Release**: Even in an emergency panic wave, a slot is **never** released while the departing body occupies its exclusion volume. This invariant guarantees that incoming birds searching for safety will never dive into an occupied branch while another creature is in mid-launch.

> [!TIP]
> You can trigger panic waves manually via code, UnityEvents, or the runtime demo control panel in `06_RealSkinnedBirds.unity`.
