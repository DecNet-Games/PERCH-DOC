---
layout: default
title: "Quadcopter Drone"
parent: "Shipped Creatures"
nav_order: 3
description: "Mechanical VTOL drone model, procedural rotor spin-down, and hover docking mechanics."
permalink: /docs/creatures/drone-quadcopter/
---

# Quadcopter Drone
{: .fs-9 }

### Mechanical VTOL Flight, Hover Descent & Automated Rotor Spin-Down
{: .fs-6 .text-grey-dk-000 }

---

To prove that PERCH's underlying architecture is fully reusable beyond organic birds, the package includes a mechanical **Quadcopter Drone** fixture. It demonstrates zero-clip procedural rotor animation, vertical hover descents, and flat docking pad alignment.

---

## Technical Specifications

| Property | Value / Specification |
|:---|:---|
| **Prefab Path** | `Assets/Decnet/Perch/Examples/Creatures/Prefabs/ConfiguredQuadcopter.prefab` |
| **Mesh Geometry** | Low-poly mechanical drone chassis with 4 independent rotor meshes (~3,400 triangles) |
| **Flight Style** | `HoverDescent` (VTOL capabilities, zero forward velocity stops) |
| **Animation Driver** | `PerchProceduralAnimationDriver` (Code-driven rotor rotation & spin-down) |

---

## Rotor Spin-Down & Spool-Up Mechanics

The quadcopter does not require complex Animator state machines or baked clips:
* **Cruise State**: All 4 rotor transforms spin at $1,200\text{ RPM}$.
* **Touchdown State**: Upon contact confirmation, rotors decelerate to $0\text{ RPM}$ over $0.4\text{s}$.
* **Takeoff State**: Rotors spool up to $1,200\text{ RPM}$ before upward thrust begins, preventing unrealistic mid-air motor starts.

---

## Docking Profile Settings

```yaml
Dimensions:
  BodyLength: 0.35 m
  BodyRadius: 0.25 m
  ContactSpan: 0.30 m
  ReservedFootprintRadius: 0.45 m

Flight Kinematics:
  FlightStyle: HoverDescent
  CruiseSpeed: 3.5 m/s
  ApproachSpeed: 1.2 m/s
  TouchdownSpeedTolerance: 0.15 m/s
  MaxDeceleration: 6.0 m/s²
  MaxTurnRate: 90 °/s
```
