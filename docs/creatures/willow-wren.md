---
layout: default
title: "Willow Wren"
parent: "Shipped Creatures"
nav_order: 2
description: "Technical anatomy, agile flight aerodynamics, and profile tuning for the Willow Wren model."
permalink: /docs/creatures/willow-wren/
---

# Willow Wren
{: .fs-9 }

### Agile Songbird Model with High-Rate Aerodynamic Turning
{: .fs-6 .text-grey-dk-000 }

---

The **Willow Wren** is PERCH's secondary skinned bird model. It features a smaller physical silhouette, distinct bone hierarchy, and rapid flight dynamics, proving that PERCH operates seamlessly across varying anatomical proportions.

![Willow Wren Close-Up]({{ site.baseurl }}/assets/images/09_Willow_Wren.png)

---

## Model & Asset Specifications

| Property | Value / Specification |
|:---|:---|
| **Prefab Path** | `Assets/Decnet/Perch/Examples/Creatures/Prefabs/ConfiguredWillowWren.prefab` |
| **Mesh Geometry** | Original CC0 stylized low-poly model (~5,200 triangles) |
| **Material / Shaders** | `URP/Lit` olive-brown plumage texture ($1024 \times 1024$) |
| **Rig Type** | Unity **Generic** Armature (16 bones) |
| **License** | CC0 Public Domain geometry and authored clips |

---

## Agile Flight Characteristics

Compared to the Garden Finch, the Willow Wren is configured for high maneuverability:
* **Turn Rate**: $160^\circ/\text{s}$ (allows tight banking around dense tree foliage).
* **Compact Footprint**: $0.30\text{m}$ reserved radius (enables closer perching density on branches).
* **Higher Flap Rate**: Wing cycle runs at $6.5\text{ Hz}$ with quick burst flutters.

---

## Calibrated Creature Profile Settings

```yaml
Dimensions:
  BodyLength: 0.20 m
  BodyRadius: 0.08 m
  ClearanceMargin: 0.03 m
  ContactSpan: 0.10 m
  ReservedFootprintRadius: 0.30 m

Flight Kinematics:
  CruiseSpeed: 3.8 m/s
  ApproachSpeed: 1.8 m/s
  TouchdownSpeedTolerance: 0.20 m/s
  MaxAcceleration: 7.0 m/s²
  MaxDeceleration: 9.0 m/s²
  MaxTurnRate: 160 °/s

Timings:
  MinimumFlightTime: 4.0 s
  MinDwellTime: 2.5 s
  MaxDwellTime: 6.0 s
  RetryCooldown: 2.0 s
```
