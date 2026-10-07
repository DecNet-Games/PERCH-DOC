---
layout: default
title: "Upgrade Guide & Changelog"
nav_order: 16
description: "Migration instructions, version history, release notes, and breaking changes for PERCH."
permalink: /docs/upgrade-guide/
---

# Upgrade Guide & Changelog
{: .fs-9 }

### Version History, Migration Notes & Asset Store Updates
{: .fs-6 .text-grey-dk-000 }

---

This document outlines version changes and safe upgrade procedures when updating PERCH from the Unity Asset Store.

---

## Safe Upgrade Procedure

When upgrading to a newer release:
1. **Backup Your Project**: Always create a git commit or project backup before updating third-party assets.
2. **Preserve Custom Profiles**: Keep your custom `PerchCreatureProfile` ScriptableObjects outside the `Assets/Decnet/Perch/` folder (e.g. in your project's `Assets/Game/Profiles/` directory) to avoid accidental overwrites.
3. **Import via Package Manager**:
   * Open `Window > Package Manager` > `My Assets`.
   * Click **Update**, then **Import**.
4. **Recompile & Verify**: Open `Window > General > Test Runner` and run all EditMode tests to confirm that all assemblies recompiled cleanly.

---

## Release Notes & Version History

### Version 2.0.0 (Flagship Commercial Release)
* **Flagship Skinned Birds**: Added original stylized **Garden Finch** and **Willow Wren** 3D models with 5 authored animation clips (Flight, Glide, Landing/Fold, Perched Idle, Takeoff).
* **Arc-Length LUT Hermite Splines**: Replaced raw parametric interpolation with precomputed 32-sample arc-length look-up tables, guaranteeing constant physical braking deceleration along curved flight paths.
* **PERCH Studio Master Window**: Introduced the unified master editor tool (`Window > PERCH > PERCH Studio` / `Ctrl+Alt+P`), unifying 5-second onboarding telemetry, 1-click authoring, geometry scanners, and rehearsal scrubbers.
* **Generic Rig Two-Bone IK Solver**: Shipped `PerchTwoBoneIkSolver`, an analytical inverse kinematics solver built specifically for Generic rigs and bird anatomy.
* **Direct Spatial Hash ID Mapping (DEC-012)**: Replaced linear slot scans with direct dictionary lookups (`_spatialIdToHandle`), improving footprint contention benchmarks by ~8x.
* **Zero-Allocation Struct Buffers (DEC-010)**: Replaced dynamic lambda event queues with fixed-size struct circular buffers (`QueuedAgentEvent[32]`, `PendingCommand[16]`), achieving steady-state 0 B managed GC allocations.
* **Surface Candidate Scanner**: Added automated raycast level geometry scanner with slope and clearance filters for 1-click perch discovery.
* **Moving Platform Kinematics**: Added target velocity sampling and momentum handover contracts for moving ships, swinging ropes, and vehicles.
* **Automated Regression Suite**: Expanded EditMode and PlayMode test suites to 22 verified assertions with leaked collider cleanup guarantees (DEC-009).
