---
layout: default
title: "Moving Platforms & Dynamic Perches"
parent: "Core Features"
nav_order: 4
description: "Kinematic sampling, relative contact coordinates, moving platform limits, and takeoff velocity inheritance."
permalink: /docs/features/moving-perches/
---

# Moving Platforms & Dynamic Perches
{: .fs-9 }

### Landing on Rocking Ships, Swaying Branches & Moving Vehicles
{: .fs-6 .text-grey-dk-000 }

---

Landing on a stationary surface is simple; landing on a boat rolling in choppy water, an airship flying through a storm, or a tree branch swaying in high wind is notoriously difficult. Most tools assume world-space static positions, leading to missed landings, feet clipping through decks, or physics explosions.

PERCH handles moving perches as first-class citizens through **Kinematic Target Sampling** and **Relative Contact Coordinate Frames**.

![Windows Unfold Capture on Perch]({{ site.baseurl }}/assets/images/Windows_Unfold.png)

---

## Kinematic Support Sampling

When a `PerchSpot` is parented to a moving transform, it tracks target kinematics across simulation ticks:

```
Platform Transform (Moving)
└── PerchSpot (Samples Delta P and Delta R per tick)
    └── PerchSlot (Relative Contact Coordinate Frame)
```

1. **Stationary Fast-Path**: If the support transform has not moved between ticks, `PerchSpot.SampleKinematics` immediately takes an identity fast-path, skipping quaternion inversions and cross-product math.
2. **Velocity & Acceleration Estimation**: When movement occurs, the spot continuously computes:
   * Instantaneous linear velocity $\mathbf{V}_{\text{platform}}$
   * Angular velocity $\mathbf{\omega}_{\text{platform}}$
   * Linear acceleration $\mathbf{a}_{\text{platform}}$
3. **Trajectory Replanning**: As the creature approaches, the trajectory solver re-evaluates the arrival spline relative to the projected touchdown position of the moving support.

---

## Operating Limits & Teleport Protection

Creatures cannot land on a platform moving at Mach 1. PERCH enforces configurable physical boundaries in the `PerchCreatureProfile`:

| Parameter | Default Threshold | Behavior If Exceeded |
|:---|:---|:---|
| **Max Platform Linear Speed** | $2.0\text{ m/s}$ | Trajectory rejected with `TargetMotionExceeded`. |
| **Max Platform Angular Speed** | $30^\circ/\text{s}$ | Approach aborted; agent recovers to free flight. |
| **Max Platform Acceleration** | $3.0\text{ m/s}^2$ | Approach aborted; prevents landing on jerking platforms. |
| **Teleport Displacement Limit** | $> 0.5\text{ m}$ in 1 tick | Flagged as `TargetTeleported`; immediate emergency abort. |
| **Teleport Angular Limit** | $> 30^\circ$ in 1 tick | Flagged as `TargetTeleported`; immediate emergency abort. |

If a platform suddenly teleports (such as a cinematic cutscene reposition or level streaming warp), PERCH detects the discontinuous delta and instantly commands approaching creatures into `AbortRecovery`, preventing visual artifacts.

---

## Relative Contact Coordinate Frame

Once the creature reaches `Touchdown`, its position and rotation are anchored relative to the support transform:

$$\mathbf{P}_{\text{world}}(t) = \mathbf{T}_{\text{platform}}(t) \cdot \mathbf{P}_{\text{local\_contact}}$$

$$\mathbf{R}_{\text{world}}(t) = \mathbf{R}_{\text{platform}}(t) \cdot \mathbf{R}_{\text{local\_contact}}$$

* **Sub-Millimeter Stability**: Even on rapidly pitching ship decks, the creature's feet remain locked to the surface without jitter or drifting.
* **Non-Uniform Scale Protection**: Moving supports must maintain uniform positive scale (`1, 1, 1`). If a moving parent uses non-uniform or animated scale, PERCH emits `UnsupportedScale` to protect solver stability.

---

## Departure Velocity Inheritance

When a bird departs from a moving vehicle, it does not launch from zero world velocity—it inherits the vehicle's momentum.

During the `TakingOff` phase, the exit velocity returned to the `IPerchFlightProvider` combines the creature's forward flap launch vector with the platform's tangential velocity:

$$\mathbf{V}_{\text{exit}} = \mathbf{V}_{\text{creature\_launch}} + \mathbf{V}_{\text{platform}} + (\mathbf{\omega}_{\text{platform}} \times \mathbf{r}_{\text{offset}})$$

```
        ^ V_creature_launch (Upward & Forward Flap)
        |
        +-----> V_platform (Moving Ship Deck Velocity)
       /
      v Resulting V_exit smoothly handed to FlightProvider
```

This prevents the jarring visual pop where a bird appears to hit an invisible wall immediately upon leaving a moving boat or train.
