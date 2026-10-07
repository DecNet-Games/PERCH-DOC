---
layout: default
title: Quick Start
nav_order: 4
description: "5-minute quick start guide to test the interactive demo scene and set up your first landing creature in Unity."
permalink: /docs/quick-start/
---

# Quick Start Guide
{: .fs-9 }

### From Zero to First Landing in Under 5 Minutes
{: .fs-6 .text-grey-dk-000 }

---

This guide gets you up and running immediately. You can either test the fully configured interactive showcase scene or set up a perch and creature in your own scene in a few steps.

---

## Option A: Explore the Interactive Demo Scene (60 Seconds)

The fastest way to experience PERCH's full capability is to run the primary showcase scene:

1. Open `Assets/Decnet/Perch/Examples/Scenes/06_RealSkinnedBirds.unity`.
2. Press **Play** in the Unity toolbar.
3. Observe the **Garden Finch** and **Willow Wren** navigating the courtyard, executing Hermite approach trajectories, landing on wooden rails, and folding their wings.

![Interactive Demo UI Controls]({{ site.baseurl }}/assets/images/11_Interactive_Demo.png)

### Demo UI Controls
The on-screen control panel allows you to inspect and command creatures in real time:
* **Contact Close-Ups**: Cycles camera focus between the Finch and Wren contact stances to inspect sub-millimeter foot placement and talon locking.
* **Pause / Resume**: Freezes simulation clock, pausing agent motion, timers, and leases synchronously.
* **Take Off**: Commands all perched birds to execute departure sweeps and launch into free flight.
* **Reset**: Returns birds to their initial airspace coordinates and resets slot registrations.
* **Show Status**: Toggles an on-screen HUD displaying active agent states, lease token sequences, and flight speeds.

---

## Option B: Set Up Your Own Scene (5 Minutes)

Follow these steps to add reliable creature perching to any custom scene:

![Hero Two Birds]({{ site.baseurl }}/assets/images/Hero_TwoBirds.png)

### Step 1: Create the `PerchWorld`
1. In the Hierarchy, right-click and select `Create Empty`.
2. Name the GameObject `PerchWorld`.
3. Add the `PerchWorld` component:
   * In the Inspector, click **Add Component** > search for `PerchWorld`.
   * Leave default settings (Spatial Hash Cell Size: `2.0`, Max Physics Queries: `64`).

```csharp
// PerchWorld manages spatial partitioning and atomic leases in the active scene.
```

### Step 2: Create a Perch Support
1. Create a support object (such as a 3D Cube stretched into a wooden fence rail: Scale `(0.1, 0.1, 2.0)`).
2. Ensure the object has a standard Unity **BoxCollider** or **CapsuleCollider**.
3. Name the GameObject `PerchRail`.

### Step 3: Add and Author a `PerchSpot`
1. On `PerchRail`, click **Add Component** > search for `PerchSpot`.
2. In the `PerchSpot` inspector:
   * Drag your `PerchWorld` GameObject into the **World** slot.
   * Under **Slots**, click **Add Slot** (or use Line Layout to generate multiple slots).
   * Set the slot's **Local Position** to the top surface of the rail (e.g., `Y = 0.05`).
   * Verify orientation:
     * **Local Y (Green Gizmo)** represents the surface normal (pointing straight up).
     * **Local Z (Blue Gizmo)** represents the bird's forward landing direction.
   * Assign the rail's `BoxCollider` to the slot's **Support Collider** field.
   * Set **Reserved Footprint Radius** to `0.5` meters.

### Step 4: Drop in a Shipped Bird Prefab
1. In the Project window, navigate to:
   `Assets/Decnet/Perch/Examples/Creatures/Prefabs/`
2. Drag `ConfiguredGardenFinch` (or `ConfiguredWillowWren`) into the Hierarchy.
3. Position the creature in clear airspace above the rail (e.g., `Position = (0, 2.5, 0)`).
4. In the creature's `PerchAgent` component:
   * Drag your `PerchWorld` GameObject into the **World** field.
   * Check **Auto Landing Enabled** (or trigger landing via script).

### Step 5: Press Play
1. Hit **Play**.
2. The bird will cruise in its free-flight roaming volume.
3. After its initial roaming cooldown, the decision engine evaluates the rail slot, acquires an exclusive lease, curves smoothly into the landing corridor, settles onto the collider surface, and holds its perched stance.

> [!TIP]
> **Hotkeys**: Press `Ctrl+Alt+P` at any time to open the **PERCH Studio** window. From Studio, you can author slots with one click, scrub trajectories in Edit Mode, and inspect real-time agent telemetry.

---

## What's Happening Behind the Scenes?

When the bird touched down, PERCH executed an automated sequence:
1. Checked external obstacles along the approach corridor via non-allocating sphere sweeps.
2. Parameterized the braking curve with an Arc-Length LUT to prevent velocity jumps.
3. Locked the creature root to the support collider's local coordinate frame.
4. Preserved physical exclusion space around the bird so no other creature can land within its wingspan.

Continue reading [Slot & Spot Authoring]({{ site.baseurl }}/docs/features/slot-authoring/) to learn about line slots, patch perches, and custom surface alignments.
