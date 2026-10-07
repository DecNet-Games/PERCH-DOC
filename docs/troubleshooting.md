---
layout: default
title: "Troubleshooting Guide"
nav_order: 15
description: "Diagnostic runbook for pink shaders, StartOverlapped, hovering feet, bone jitter, and UPM errors."
permalink: /docs/troubleshooting/
---

# Troubleshooting Guide
{: .fs-9 }

### Rapid Diagnostics & Solutions for Common Integration Issues
{: .fs-6 .text-grey-dk-000 }

---

If you encounter unexpected behavior during setup, consult this diagnostic runbook.

---

## 1. Demo Materials Appear Pink / Magenta

* **Root Cause**: The sample scenes in `Assets/Decnet/Perch/Examples` use shaders built for the **Universal Render Pipeline (URP)**. If your project is using the Built-in Pipeline or lacks an active URP Asset, materials fail to compile.
* **Solution**:
  1. Open `Edit > Project Settings > Graphics`.
  2. Ensure a valid **Universal Render Pipeline Asset** is assigned.
  3. Repeat in `Edit > Project Settings > Quality`.
  *(Note: The core C# runtime code is pipeline-independent and works with custom shaders in any pipeline).*

---

## 2. Creature Wanders but Never Lands

* **Root Cause**: Policy filtering, mismatched references, or unassigned colliders.
* **Diagnostic Steps**:
  1. Open `Window > Perch > Landing Debugger` and select the creature.
  2. Check **Decision Status**: If it says `CooldownActive` or `MinimumFlightActive`, the creature is simply waiting for its policy timer to expire.
  3. Check **Last Failure Reason**:
     * If `InvalidSetup`: Ensure `World`, `Profile`, `Motor`, and slot `Support Collider` fields are not null.
     * If `TagMismatch`: Ensure creature profile tags match slot tags.
     * If `NoCandidates`: Ensure the creature is within 50 meters of the perch.

---

## 3. Immediate `StartOverlapped` Error

* **Root Cause**: At the moment the creature initiates trajectory planning, its physical body sphere collides with external scene geometry (such as an overhanging tree branch, cave roof, or terrain collider).
* **Solution**:
  * Expand creature roaming waypoints away from tight corners.
  * Adjust `BodyRadius` in `PerchCreatureProfile` if the collision sphere is larger than the actual creature mesh.

---

## 4. Feet Hover in Mid-Air or Clip Through Branches

* **Root Cause**: Uncalibrated contact anchor or bone origin offset.
* **Solution**:
  1. Select the creature's `PerchRigBinding` component.
  2. Align the `ContactAnchor` Transform precisely with the lowest vertices of the feet mesh (not the hip bone origin).
  3. Click **Calculate Offset from Anchor** to refresh root offsets.

---

## 5. Wings Snap or Skeletons Jitter Violently

* **Root Cause**: Two competing systems attempting to write to the same skeleton.
* **Solution**:
  * **Single Writer Rule**: Never attach both `PerchProceduralAnimationDriver` and `PerchClipAnimationDriver` to the same character.
  * **Disable Root Motion**: In your Mecanim Animator component, ensure **Apply Root Motion** is **unchecked**. PERCH's motor owns root displacement.

---

## 6. Bird Circles Endlessly Around a Waypoint

* **Root Cause**: Waypoint arrival radius is smaller than the creature's minimum turning circle.
* **Solution**:
  * Increase `ArrivalRadius` on `PerchWaypointFlightProvider` to be $\ge 1.5\text{ m}$.
  * Increase `MaxTurnRate` in `PerchCreatureProfile`.

---

## 7. Package Manager Path Error on Windows

* **Root Cause**: Automated command-line Unity invocations missing environment variables.
* **Solution**: Ensure child processes inherit the `ALLUSERSPROFILE` environment variable (typically `C:/ProgramData`), which Unity Package Manager requires to resolve local global registries.
