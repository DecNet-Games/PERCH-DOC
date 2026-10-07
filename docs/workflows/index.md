---
layout: default
title: Workflows & Recipes
nav_order: 9
has_children: true
description: "Production workflows, step-by-step recipes, custom creatures, high-density flocks, and vehicle perches."
permalink: /docs/workflows/
---

# Workflows & Recipes
{: .fs-9 }

### Production Solutions for Real-World Game Development Challenges
{: .fs-6 .text-grey-dk-000 }

---

Integrating complex creature AI into a production title requires more than reading API docs. This section provides battle-tested production guides for bringing your own assets, scaling simulations, and coordinating complex cinematics.

---

## Workflow Guides

### 1. [Importing Custom Creatures]({{ site.baseurl }}/docs/workflows/custom-creatures/)
Comprehensive guide to bringing third-party 3D bird and flying creature models into PERCH. Covers FBX scale normalization, visual wrapper generation, contact anchor calibration, and bone auto-detection.

### 2. [High-Density Flocks (300+ Birds)]({{ site.baseurl }}/docs/workflows/high-density-flocks/)
How to optimize PERCH for large ambient environments with hundreds of active agents. Explains scheduler query budgets (`MaxPhysicsQueriesPerTick`), coarse distance culling, and staggered decision jitter.

### 3. [Vehicles, Ships & Drones]({{ site.baseurl }}/docs/workflows/moving-vehicles-drones/)
Authoring moving perches on ships, airships, swinging ropes, trains, and mechanical quadcopters with rotor spin-down.

### 4. [Cinematics & Timeline Integration]({{ site.baseurl }}/docs/workflows/cinematics-timeline/)
How to coordinate scripted landings for narrative sequences, cutscenes, and Timeline tracks while preserving physical clearance guarantees.

### 5. [Procedural Environment Generation]({{ site.baseurl }}/docs/workflows/procedural-environment-generation/)
Authoring and generating perches dynamically at runtime in procedural levels, dungeon crawlers, or dynamic spline-based roads and wires.
