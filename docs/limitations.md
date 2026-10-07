---
layout: default
title: "Operating Envelope & Limits"
nav_order: 14
description: "Explicit operational boundaries, physical envelopes, and non-supported architectural scopes."
permalink: /docs/limitations/
---

# Operating Envelope & Limits
{: .fs-9 }

### Clear Engineering Boundaries & Operating Constraints
{: .fs-6 .text-grey-dk-000 }

---

Professional game development requires clear boundaries. This document outlines exactly what PERCH does—and what lies outside its designed operational envelope.

---

## What PERCH Is vs What It Is Not

| Capability | Supported by PERCH | Outside Designed Scope |
|:---|:---:|:---:|
| **Local Approach & Landing** | **YES** | Global 3D maze pathfinding |
| **Exclusive Slot Reservations** | **YES** | Multi-server network replication |
| **Moving Platform Compensation** | **YES** | Deforming skeletal mesh perches |
| **Generic Rig Two-Bone IK** | **YES** | Universal auto-retargeting across arbitrary rigs |
| **Perched Dwell & Takeoff** | **YES** | Ground walking locomotion |
| **Zero-GC Simulation Tick** | **YES** | Unity DOTS / ECS architecture |

---

## Core Operational Constraints

### 1. Local Bounded Flight vs Global Pathfinding
PERCH specializes in local landing approaches, obstacle sweeps, spatial reservations, and smooth touchdowns. It is **not** a global volumetric 3D navigation mesh solver. For navigating complex indoor labyrinths, integrate an external navigation system via `IPerchFlightProvider`.

### 2. Uniform Root Transform Scale
The creature root GameObject must maintain uniform positive scale (`1, 1, 1`). If imported models require scaling, place the mesh inside a child `VisualWrapper` GameObject. Non-uniform or negative root scales are rejected with `UnsupportedScale`.

### 3. Moving Support Constraints
Moving perches must be rigid transforms. The supported baseline envelope is:
* Linear speed $\le 2.0\text{ m/s}$
* Angular speed $\le 30^\circ/\text{s}$
* Linear acceleration $\le 3.0\text{ m/s}^2$
Platforms that teleport ($>0.5\text{m}$ or $>30^\circ$ in one tick) will cause approaching agents to abort with `TargetTeleported`. Deforming skinned meshes (e.g. landing on a walking giant's shoulder) are not supported as rigid perch supports.

### 4. Physical Geometry Requires Colliders
PERCH's swept obstacle detection relies on standard Unity physics. Visual-only vegetation or alpha-card leaves without colliders cannot be detected by collision sweeps.

### 5. Qualified Baseline
PERCH is officially qualified on **Unity 6000.5.4f1 LTS**, **URP 17.5.0**, and **Windows x64**. While the core C# runtime is pipeline-independent, other operating systems and Unity releases have not undergone formal automated verification suites.
