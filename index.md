---
layout: default
title: Home
nav_order: 1
description: "PERCH: Complete creature landing, perching, and flight coordination system for Unity."
permalink: /
---

# PERCH
{: .fs-9 }

### Creature Landing, Perching & Dynamic Flight Coordination for Unity
{: .fs-6 .text-grey-dk-000 }

---

<div style="margin: 1.5rem 0; border-radius: 8px; overflow: hidden; box-shadow: 0 4px 20px rgba(0,0,0,0.4);">
  <video width="100%" controls playsinline poster="{{ site.baseurl }}/assets/images/Cover_1950x1300.png">
    <source src="{{ site.baseurl }}/assets/videos/PERCH_Showcase_1080p.mp4" type="video/mp4">
    Your browser does not support the video tag.
  </video>
</div>
<p align="center" style="font-size: 0.85rem; color: #888;">
  <em>Verified runtime capture: Continuous 32-second Windows player execution featuring original Garden Finch and Willow Wren skinned models, real-time procedural wing fold hold, and dynamic takeoff cycles.</em>
</p>

---

## Stop Fighting Clunky Raycasts and Floaty Snaps

Building believable avian or aerial creature AI in Unity usually hits an infuriating wall:
* **The Teleport Snap**: Creatures fly near a perch, pause unnaturally, and snap their roots into position—ruining immersion.
* **The Multi-Agent Cluster**: Multiple birds choose the exact same branch, clip through each other's wings, or fight for the same physical slot.
* **Rig Rigidity**: Built-in Unity IK (`OnAnimatorIK`) is locked to Humanoid avatars, leaving Generic bird rigs and custom quadrupeds completely stranded.
* **Moving Platforms Break Everything**: The moment a branch sways in the wind or a ship rolls on the water, standard landing logic decouples or shoots the agent into deep space.
* **Silent Failures**: When a landing aborts, the console is dead silent—leaving programmers guessing whether it was a collision sweep, timing desync, or distance failure.

**PERCH solves this with an unshakeable, production-hardened engineering architecture.**

---

## What is PERCH?

**PERCH** is a high-performance creature landing, perching, and dynamic flight coordination framework built specifically for Unity 6 LTS. It replaces brittle scripts with an atomic, reservation-driven pipeline that governs how flying creatures identify perches, reserve exclusive space, execute aerodynamically convincing deceleration and flares, touch down with sub-millimeter foot placement, dwell naturally, and depart without collision.

![Living Courtyard Showcase]({{ site.baseurl }}/assets/images/01_Living_Courtyard.png)

---

## Core Value Pillars

```
+-----------------------------------------------------------------------------------+
|                                  PERCH ARCHITECTURE                                |
+-----------------------------------------------------------------------------------+
|  [ AUTHORING ]     PerchSpot / PerchSlot | Line & Patch Layouts | Studio Scanner   |
|  [ COORDINATION ]  3D Spatial Hash (O(1)) | Atomic LeaseManager | Exclusion Volumes|
|  [ TRAJECTORY ]    Arc-Length Hermite Splines | ForwardGlide & HoverDescent Modes  |
|  [ CONTACT ]       Generic Rig Two-Bone IK | Planted Stance Holds | Surface Normal |
|  [ DYNAMICS ]      Relative Moving Platform Sampling | Velocity Handover Contract |
|  [ RUNTIME ]       Zero Managed GC Allocations | Preallocated Struct Buffers       |
+-----------------------------------------------------------------------------------+
```

### 1. Atomic Lease Management & Spatial Reservations
No two creatures will ever clip or fight for the same perch. PERCH's `LeaseManager` executes main-thread atomic claims backed by a 3D spatial hash grid (`SpatialHash3D`). Reservation tokens, monotonic request IDs, heartbeat TTLs, and dynamic creature footprint radii guarantee conflict-free flock behavior even during high-contention panic waves.

### 2. Arc-Length LUT Hermite Splines
Unlike naive interpolation that causes sudden speed spikes near tight curves, PERCH parameterizes approach curves using a 32-sample arc-length look-up table (LUT). Creatures decelerate along physical braking limits, flare their wings with realistic drag, and align their approach vectors with the perch normal.

### 3. Generic Rig Two-Bone IK Solver
Unity's native `OnAnimatorIK` does not support Generic rigs. PERCH includes an analytical, high-efficiency two-bone IK solver (`PerchTwoBoneIkSolver`) specifically calibrated for bird legs, talons, and landing gear—guaranteeing feet firmly lock to uneven branches without slipping.

### 4. Moving Perch Kinematic Compensation
Whether your perches are stationary tree branches, swinging ropes, rocking boat railings, or moving vehicle landing pads, `PerchSpot` continuously samples target linear and angular velocities. Approach curves dynamically adapt relative to the target's frame of reference, and departures seamlessly inherit platform momentum.

### 5. Master PERCH Studio Hub
An all-in-one developer environment (`Window > PERCH > PERCH Studio` / `Ctrl+Alt+P`) featuring live 10Hz telemetry, 1-click creature and slot authoring wizards, automated surface geometry raycast scanners, and non-destructive edit-mode trajectory rehearsal scrubbers.

### 6. Zero Managed GC Allocations
Designed for mission-critical production frame rates. All physics query buffers (`RaycastHit[]`, `Collider[]`), event flushes, and command queues use fixed-size preallocated struct circular buffers. The steady-state simulation tick produces **0 B managed heap allocations**.

---

## Visual Demonstration

PERCH ships with fully configured, stylized **Garden Finch** and **Willow Wren** skinned models featuring 5 distinct motion clips (Flight, Glide, Landing/Fold, Perched Idle, Takeoff) alongside full URP shaders and interactive camera inspection tools.

| Complete Flight-to-Perch Cycle | Procedural Rig & Anchor Setup |
|:---:|:---:|
| ![Complete Cycle]({{ site.baseurl }}/assets/images/02_Complete_Cycle.png) | ![Rig Bindings]({{ site.baseurl }}/assets/images/14_Rig_Bindings.png) |
| *Smooth deceleration, flare, touchdown, and wing folding* | *Calibrated contact anchors, visual wrappers, and bone heuristics* |

---

## Quick Navigation

<div class="cards-grid" style="display: grid; grid-template-columns: repeat(auto-fit, minmax(280px, 1fr)); gap: 1rem; margin-top: 1.5rem;">

  <div style="border: 1px solid #333; border-radius: 6px; padding: 1.2rem; background: rgba(255,255,255,0.02);">
    <h3><a href="{{ site.baseurl }}/docs/getting-started/">Getting Started</a></h3>
    <p>Understand the core architecture, the 7-phase state machine, and fundamental system concepts.</p>
  </div>

  <div style="border: 1px solid #333; border-radius: 6px; padding: 1.2rem; background: rgba(255,255,255,0.02);">
    <h3><a href="{{ site.baseurl }}/docs/installation/">Installation & Requirements</a></h3>
    <p>Import requirements, Unity Asset Store procurement details, and URP/UGUI pipeline configuration.</p>
  </div>

  <div style="border: 1px solid #333; border-radius: 6px; padding: 1.2rem; background: rgba(255,255,255,0.02);">
    <h3><a href="{{ site.baseurl }}/docs/quick-start/">Quick Start (5 Mins)</a></h3>
    <p>Step-by-step walkthrough: loading the sample scene, placing your first bird, and landing on an authored rail.</p>
  </div>

  <div style="border: 1px solid #333; border-radius: 6px; padding: 1.2rem; background: rgba(255,255,255,0.02);">
    <h3><a href="{{ site.baseurl }}/docs/features/">Core Features</a></h3>
    <p>Deep-dive technical guides on slot authoring, reservations, Hermite splines, moving platforms, and Studio.</p>
  </div>

  <div style="border: 1px solid #333; border-radius: 6px; padding: 1.2rem; background: rgba(255,255,255,0.02);">
    <h3><a href="{{ site.baseurl }}/docs/rules/">Diagnostics & Error Codes</a></h3>
    <p>Complete dictionary of all 24 failure codes, runtime Landing Debugger, and geometry candidate scanner.</p>
  </div>

  <div style="border: 1px solid #333; border-radius: 6px; padding: 1.2rem; background: rgba(255,255,255,0.02);">
    <h3><a href="{{ site.baseurl }}/docs/architecture/">System Architecture</a></h3>
    <p>Assembly isolation, single-writer motor ownership invariants, and zero-allocation runtime design.</p>
  </div>

</div>

---

> [!NOTE]
> **Commercial Asset Notice**: PERCH is a proprietary commercial package distributed exclusively via the [Unity Asset Store](https://assetstore.unity.com/). It is not available as free open-source software on Git. All source code in the commercial distribution is fully editable and dependency-free.
