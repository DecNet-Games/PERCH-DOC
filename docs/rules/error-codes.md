---
layout: default
title: "Failure Codes & Remediation"
parent: "Diagnostics & Tools"
nav_order: 1
description: "Canonical reference for all 24 PERCH FailureReason codes, root cause diagnosis, and remediation steps."
permalink: /docs/rules/error-codes/
---

# Failure Codes & Remediation
{: .fs-9 }

### Complete Taxonomy of Strongly Typed Failure Reasons
{: .fs-6 .text-grey-dk-000 }

---

PERCH rejects the antipattern of silent failures. When an agent cannot land, its `LastFailureReason` property is immediately updated with a strongly typed enum from `Decnet.Perch.Runtime.Core.Enums.FailureReason`.

Use this reference to diagnose issues in your scenes.

---

## The Complete `FailureReason` Matrix

| Failure Reason | Description & Root Cause | Actionable Remediation |
|:---|:---|:---|
| **`None`** | Normal execution. No errors. | N/A |
| **`InvalidSetup`** | Crucial component reference missing (e.g., `World`, `Profile`, `Motor`, `Support Collider`, or `ContactAnchor`). | Inspect `PerchAgent` and `PerchSpot` in the Inspector. Ensure all required fields are assigned. |
| **`UnsupportedScale`** | Creature root transform or moving platform has negative or non-uniform scale (e.g. `(1, 2, 1)`). | Normalize root transform to uniform scale `(1, 1, 1)`. Apply visual scaling to a child model GameObject. |
| **`NoCandidates`** | World has no registered slots, or all slots are beyond the 50m search radius. | Author additional slots in the scene or expand the creature's roaming volume. |
| **`TagMismatch`** | Slot requires specific tags (e.g. `SmallBird`) not present in creature's profile. | Align tags in `PerchCreatureProfile` with the target `PerchSlot` tags. |
| **`StyleMismatch`** | Slot accepts `HoverDescent` only, but creature profile is configured as `ForwardGlide` (or vice versa). | Match the slot's allowed flight styles with the creature profile. |
| **`TooSmall`** | Slot contact patch dimensions are smaller than creature's minimum contact span. | Increase slot patch size or adjust creature profile dimensions. |
| **`UnsupportedSurface`** | Surface normal tilt angle exceeds creature's maximum allowed landing pitch. | Author slots on flatter surfaces or increase creature's allowed slope tolerance. |
| **`Occupied`** | Target slot is currently reserved or occupied by another creature. | Normal contention. Creature will automatically select an alternative slot. |
| **`FootprintConflict`** | Adjacent slot is in use and its reserved wingspan exclusion volume overlaps this slot. | Increase spacing between slots along the branch or reduce `ReservedFootprintRadius`. |
| **`StartOverlapped`** | Creature's starting body envelope is intersecting external scene geometry at planning time. | Relocate creature roaming path away from walls, ceilings, or foliage colliders. |
| **`ApproachBlocked`** | Swept sphere cast detected an obstacle obstructing the planned Hermite approach corridor. | Remove obstructing colliders, reposition the perch, or widen the approach angle. |
| **`RouteInfeasible`** | Required approach curve violates profile kinematic limits (e.g. exceeds max deceleration or turn rate). | Allow creature a longer approach distance or increase max turn rate in profile. |
| **`DepartureBlocked`** | Pre-flight departure sweep detected an obstacle obstructing the takeoff escape corridor. | Clear overhead geometry above the perch. Creature will safely remain perched until path clears. |
| **`QuerySaturated`** | Physics query hit buffer exceeded capacity (`MaxPhysicsQueriesPerTick`). | Increase query buffer size in `PerchWorld` or reduce density of surrounding colliders. |
| **`PlanningTimeout`** | Trajectory planning failed to resolve within the 2.0-second deadline. | Normal protection against pathological setups. Creature returns to free flight. |
| **`LeaseLost`** | Target perch spot was destroyed, rebuilt, or its generation counter changed during approach. | Creature automatically aborts and selects a fresh valid slot. |
| **`SpotUnavailable`** | Target perch GameObject was deactivated (`SetActive(false)`) mid-flight. | Avoid disabling perches while creatures are in approach state. |
| **`TargetMotionExceeded`** | Moving platform exceeded speed ($>2\text{ m/s}$), angular velocity ($>30^\circ/\text{s}$), or acceleration ($>3\text{ m/s}^2$). | Smooth platform movement curves or adjust creature profile limits for fast vehicles. |
| **`TargetTeleported`** | Moving platform displaced $>0.5\text{ m}$ or rotated $>30^\circ$ in a single simulation tick. | Smooth cinematic camera/platform warps. Creature initiates immediate safe abort recovery. |
| **`ContactUnreachable`** | Two-bone IK solver could not reach surface without hyper-extending leg joints. | Calibrate `PerchRigBinding` contact anchor height to match actual mesh vertices. |
| **`AnimationTimeout`** | Animator failed to transition to touchdown/fold state within watchdog duration. | Verify Animator controller transitions and parameter naming (`IsLanding`, `IsPerched`). |
| **`ProviderRejected`** | External flight provider refused to yield control to PERCH motor. | Check custom `IPerchFlightProvider.CanYield()` implementation. |
| **`Cancelled`** | Landing was explicitly cancelled via code by calling `agent.CancelLanding()`. | Normal programmatic response. |
| **`RecoveryBlocked`** | Abort escape path is obstructed by geometry. | Agent enters `InterventionRequired` and holds last safe pose. |

---

## Normal Policy States vs Actual Failures

> [!NOTE]
> Do not confuse normal policy waiting states with failures:
> * **`CooldownActive`**: The creature recently attempted a landing or departed a perch and is waiting for its configured retry cooldown ($2.0\text{s} - 5.0\text{s}$).
> * **`MinimumFlightActive`**: The creature is fulfilling its required free-flight roaming time before being allowed to search for new perches.
> 
> Both of these are reported in the Landing Debugger under **Decision Status**, leaving `LastFailureReason` set to `None`.
