---
layout: default
title: Core Features
nav_order: 5
has_children: true
description: "Comprehensive guide to PERCH core systems, algorithms, authoring workflows, and runtime capabilities."
permalink: /docs/features/
---

# Core Features
{: .fs-9 }

### Deep Technical Capabilities of the PERCH Framework
{: .fs-6 .text-grey-dk-000 }

---

PERCH provides an end-to-end toolchain and runtime framework for creature landing, perching, and flight coordination. Each feature is built to solve specific production bottlenecks in game development.

![Studio Overview Hub]({{ site.baseurl }}/assets/images/12_Studio.png)

---

## Feature Matrix & Guide Index

Explore each core system in detail:

### 1. [Slot & Spot Authoring]({{ site.baseurl }}/docs/features/slot-authoring/)
Learn how to define landing locations using `PerchSpot` and `PerchSlot`. Covers single positions, continuous branch lines, patch areas, local contact frames, normal vectors, and support collider associations.

### 2. [Atomic Lease Management]({{ site.baseurl }}/docs/features/lease-management/)
Deep dive into PERCH's deterministic reservation engine. Understand atomic slot claims, lease sequence tokens, heartbeat time-to-live (TTL), and dynamic creature footprint exclusion volumes.

### 3. [Approach Trajectories & Splines]({{ site.baseurl }}/docs/features/approach-trajectories/)
Examine the math behind arc-length look-up table (LUT) parameterized cubic Hermite splines. Covers `ForwardGlide` vs `HoverDescent` motion styles, aerodynamic flare envelopes, and banking calculations.

### 4. [Moving Platforms & Dynamic Perches]({{ site.baseurl }}/docs/features/moving-perches/)
How PERCH lands creatures on moving platforms, swinging branches, vehicles, and ships. Covers kinematic velocity sampling, relative contact coordinate frames, and departure velocity inheritance.

### 5. [Animation Coordination & IK]({{ site.baseurl }}/docs/features/animation-systems/)
Explore the three animation drivers: Mecanim clip transitions (`PerchClipAnimationDriver`), programmatic harmonic wing flappers (`PerchProceduralAnimationDriver`), and the manual Playables mixer (`PerchPoseClipDriver`). Includes the Generic rig `PerchTwoBoneIkSolver`.

### 6. [PERCH Studio Master Hub]({{ site.baseurl }}/docs/features/perch-studio/)
Complete tour of the unified editor hub (`Window > PERCH > PERCH Studio` / `Ctrl+Alt+P`), featuring 5-second onboarding telemetry, 1-click authoring tools, geometry scanners, and 1-click scene switchers.

### 7. [3D Spatial Hash Partitioning]({{ site.baseurl }}/docs/features/spatial-hash/)
How `SpatialHash3D` maintains $O(1)$ slot queries and dynamic footprint checks with zero runtime GC allocations, supporting dense flocks across massive environments.

### 8. [Threats & Panic Sequences]({{ site.baseurl }}/docs/features/panic-waves/)
How `PerchThreatSource` triggers coordinated flock panic waves, enforcing departure clearance checks and holding exclusion space until departing bodies clear the perch.
