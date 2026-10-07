---
layout: default
title: "Approach Trajectories & Splines"
parent: "Core Features"
nav_order: 3
description: "Arc-length LUT parameterized Hermite splines, ForwardGlide vs HoverDescent modes, aerodynamic flare, and swept clearance."
permalink: /docs/features/approach-trajectories/
---

# Approach Trajectories & Splines
{: .fs-9 }

### Physics-Bounded Curves, Arc-Length Parameterization & Swept Clearance
{: .fs-6 .text-grey-dk-000 }

---

Standard Bezier curves evaluated at naive normalized time $t \in [0, 1]$ suffer from a fatal flaw: when a curve bends sharply, equal steps in $t$ cover vastly different physical distances. This causes flying creatures to accelerate violently through turns and decelerate unnaturally on straights.

PERCH solves this with **Arc-Length Parameterized Hermite Splines** backed by pre-computed Look-Up Tables (LUTs).

![Approach Trajectory Flare]({{ site.baseurl }}/assets/images/03_Approach.png)

---

## The Arc-Length LUT Hermite Spline Solver

To guarantee constant, controllable deceleration along physical profile limits:
1. When a landing route is generated, PERCH computes a cubic Hermite spline connecting the creature's current flight state $(\mathbf{P}_0, \mathbf{V}_0)$ to the slot's contact pose $(\mathbf{P}_{\text{contact}}, \mathbf{N}_{\text{normal}})$.
2. The curve is numerically integrated into a **32-sample Arc-Length Look-Up Table (LUT)**:
   $$s(t_i) = \int_{0}^{t_i} \|\mathbf{S}'(\tau)\| \, d\tau$$
3. During runtime ticks, given the creature's current distance traveled along the path $s$, the exact parametric coordinate $t$ is resolved via binary search and linear interpolation within the LUT:
   $$t = \text{LUT}^{-1}(s)$$
4. This ensures that physical braking profiles (deceleration up to $8\text{ m/s}^2$) produce perfectly smooth velocities regardless of how sharply the trajectory curves.

```
+-----------------------------------------------------------------------------------+
| Parametric Spline S(t)  -->  Arc-Length LUT (32 Samples)  -->  Uniform Trajectory |
+-----------------------------------------------------------------------------------+
|  [HIGH CURVATURE]   === Dense Arc-Length Samples ===> Controlled Physical Speed   |
|  [LOW CURVATURE]    === Sparse Arc-Length Samples ==> Smooth Braking Profile      |
+-----------------------------------------------------------------------------------+
```

---

## Flight Styles: ForwardGlide vs HoverDescent

PERCH provides two fundamentally different flight profiles depending on creature biology or mechanics:

| Metric | `ForwardGlide` (Birds, Winged Creatures) | `HoverDescent` (Drones, Hummingbirds, VTOL) |
|:---|:---|:---|
| **Minimum Airspeed** | Strictly $> 0$ until touchdown. Cannot stop mid-air. | $0\text{ m/s}$. Can stop, hold position, and hover. |
| **Approach Path** | Sweeping aerodynamic arc aligning with perch axis. | Direct descent vector onto the landing patch. |
| **Flare Dynamics** | Pitches upward to convert kinetic energy into drag. | Level attitude or slight counter-thrust pitch. |
| **Banking** | Banks dynamically into turns ($\phi = f(\text{speed}, \text{curvature})$). | Yaw-in-place with independent tilt limits. |
| **Touchdown Envelope** | Shallow tangential angle of attack ($15^\circ - 35^\circ$). | Vertical or steep descent ($45^\circ - 90^\circ$). |

![Garden Finch Banking and Flare]({{ site.baseurl }}/assets/images/08_Garden_Finch.png)

---

## Aerodynamic Flare & Kinematic Limits

In nature, birds do not fly at full speed into a branch and stop instantly. They flare their wings, pitch their breast upward, and bleed velocity right at the contact threshold.

PERCH replicates this with profile-driven kinematic limits:
* **Cruise Speed**: Nominal roaming speed (default: $4.0\text{ m/s}$).
* **Approach Speed**: Speed entering the approach corridor (default: $2.0\text{ m/s}$).
* **Touchdown Relative Speed**: Speed at the moment of contact (strictly $\le 0.25\text{ m/s}$).
* **Max Deceleration**: Maximum allowable braking force (default: $8.0\text{ m/s}^2$).
* **Max Turn Rate**: Maximum angular yaw velocity (default: $120^\circ/\text{s}$).

If a prospective landing route requires a turn or deceleration that exceeds these limits, the candidate is discarded with `RouteInfeasible` before wasting CPU cycles on physics sweeps.

---

## Swept Corridor Clearance Verification

Before an agent commits to an approach curve, the swept flight corridor must be proven clear of external obstacles:

```
[Start Envelope]   ======== Swept SphereCast Corridor ========   [Touchdown Funnel]
  (Agent Radius)   -------------------------------------------      (Slot Patch)
```

1. **Initial Overlap Check**: A sphere check at the creature's current position verifies that the agent is not already inside scene geometry. If obstructed by an external collider, it fails with `StartOverlapped`.
2. **Path Segment Sweeps**: The spline is segmented (up to 32 segments) and evaluated using non-allocating sphere casts (`SphereCastNonAlloc`). If an obstacle intersects the corridor, planning fails with `ApproachBlocked`.
3. **Touchdown Exemption**: In the final arrival segment, the slot's assigned **Support Collider** is granted an exclusive collision exemption so the perch itself is not registered as an obstacle.
4. **Departure Feasibility**: In addition to the approach, PERCH verifies that an initial departure escape vector is clear before committing to touchdown.

> [!TIP]
> All collision sweeps use statically preallocated hit arrays. If dense geometry saturates the buffer capacity, PERCH safely rejects the path with `QuerySaturated` rather than risking an unverified path passing through an obstacle.
