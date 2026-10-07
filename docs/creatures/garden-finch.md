---
layout: default
title: "Garden Finch"
parent: "Shipped Creatures"
nav_order: 1
description: "Technical anatomy, rig hierarchy, 5 authored clips, and creature profile for the Garden Finch."
permalink: /docs/creatures/garden-finch/
---

# Garden Finch
{: .fs-9 }

### Flagship Stylized Skinned Bird Model with 5 Authored Clips
{: .fs-6 .text-grey-dk-000 }

---

The **Garden Finch** is PERCH's primary hero bird model. It demonstrates full end-to-end integration: custom generic armature, 5 authored motion clips, calibrated contact markers, and balanced flight aerodynamics.

![Garden Finch Close-Up]({{ site.baseurl }}/assets/images/08_Garden_Finch.png)

---

## Model & Asset Specifications

| Property | Value / Specification |
|:---|:---|
| **Prefab Path** | `Assets/Decnet/Perch/Examples/Creatures/Prefabs/ConfiguredGardenFinch.prefab` |
| **Mesh Geometry** | Original CC0 stylized low-poly model (~6,800 triangles) |
| **Material / Shaders** | `URP/Lit` stylized hand-painted feather atlas ($1024 \times 1024$) |
| **Rig Type** | Unity **Generic** Armature (18 bones) |
| **License** | CC0 Public Domain geometry and authored clips |

---

## The 5 Authored Animation Clips

The Garden Finch uses `PerchPoseClipDriver` with five rig-specific clips:

1. **`Finch_Flight`**: Continuous rhythmic wing flapping with sinusoidal body bobbing.
2. **`Finch_Glide`**: Extended wing banking posture used during straightaway cruising.
3. **`Finch_LandFold`**: Aerodynamic flare posture transitioning into progressive wing folding against the body.
4. **`Finch_PerchedIdle`**: Perched stance featuring subtle head tilts and breathing balance shifts while feet remain pinned.
5. **`Finch_Takeoff`**: Rapid power flap launch transitioning smoothly back into forward flight.

---

## Calibrated Creature Profile Settings

The shipped `Profile_GardenFinch.asset` is pre-configured with the following values:

```yaml
Dimensions:
  BodyLength: 0.28 m
  BodyRadius: 0.10 m
  ClearanceMargin: 0.04 m
  ContactSpan: 0.14 m
  ReservedFootprintRadius: 0.40 m

Flight Kinematics:
  CruiseSpeed: 4.2 m/s
  ApproachSpeed: 2.1 m/s
  TouchdownSpeedTolerance: 0.22 m/s
  MaxAcceleration: 6.5 m/s²
  MaxDeceleration: 8.5 m/s²
  MaxTurnRate: 130 °/s

Timings:
  MinimumFlightTime: 5.0 s
  MinDwellTime: 4.0 s
  MaxDwellTime: 9.0 s
  RetryCooldown: 2.5 s
```

> [!TIP]
> Drag `ConfiguredGardenFinch` directly into any scene with a `PerchWorld` and a rail to watch it operate with zero configuration required.
