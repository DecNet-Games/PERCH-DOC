---
layout: default
title: "Commercial Audit Matrix"
parent: "Quality Assurance"
nav_order: 2
description: "Feature traceability ledger, verification criteria, and compliance matrix for PERCH."
permalink: /docs/qa/audit-matrix/
---

# Commercial Audit Matrix
{: .fs-9 }

### Feature Traceability, Measurable Criteria & Commercial Qualification
{: .fs-6 .text-grey-dk-000 }

---

To guarantee that PERCH ships with commercial reliability, every feature is tracked against a formal **Feature Traceability Ledger**.

---

## Canonical Feature Traceability Matrix

| Feature ID | Specification Contract | Core Implementation | Verification Status |
|:---|:---|:---|:---|
| **L01** | Explicit `PerchWorld` and multi-world isolation | `PerchWorld.cs`, `SpatialHash3D.cs` | **VERIFIED** |
| **L02** | Spot, slot and contact frame authoring | `PerchSpot.cs`, `PerchSlot.cs` | **VERIFIED** |
| **L03** | Suitability, tags and spatial separation | `CandidateSelector.cs` | **VERIFIED** |
| **L04** | Atomic leases and exclusive occupied slots | `LeaseManager.cs` (1,000 contention batches) | **VERIFIED** |
| **L05** | Candidate scoring, weights, and decision jitter | `PerchAgent.Policy.cs` | **VERIFIED** |
| **L06** | 10-state finite state machine with timeout safety | `PerchAgent.cs` | **VERIFIED** |
| **L07** | Free flight roaming and minimum flight times | `PerchWaypointFlightProvider.cs` | **VERIFIED** |
| **L08** | `ForwardGlide` and `HoverDescent` flight styles | `PerchApproachTrajectory.cs` | **VERIFIED** |
| **L09** | Swept approach and departure clearance sweeps | `PerchPlanner.cs` (`SphereCastNonAlloc`) | **VERIFIED** |
| **L11** | Contact solve and touchdown continuity | `PerchAgent.Touchdown.cs` | **VERIFIED** |
| **L12** | Moving perch tracking & platform velocity handover | `PerchSpot.Kinematics.cs` | **VERIFIED** |
| **L13** | Perched dwell timers and departure clearance | `PerchAgent.Perched.cs` | **VERIFIED** |
| **L14** | Threat panic wave propagation | `PerchThreatSource.cs` | **VERIFIED** |
| **L15** | Mecanim Animator clip driver | `PerchClipAnimationDriver.cs` | **VERIFIED** |
| **L16** | Procedural harmonic wing & rotor driver | `PerchProceduralAnimationDriver.cs` | **VERIFIED** |
| **L17** | Generic rig two-bone IK solver | `PerchTwoBoneIkSolver.cs` | **VERIFIED** |
| **L18** | Single-writer motion provider handover | `IPerchFlightProvider.cs`, `IPerchMotor.cs` | **VERIFIED** |
| **L20** | Non-destructive Creature Setup Wizard | `PerchSetupWizard.cs` | **VERIFIED** |
| **L21** | Surface geometry candidate scanner | `PerchSurfaceScanner.cs` | **VERIFIED** |
| **L22** | Edit-mode trajectory rehearsal scrubber | `PerchRehearsalDriver.cs` | **VERIFIED** |
| **L23** | Live runtime Landing Debugger & telemetry | `PerchLandingDebuggerWindow.cs` | **VERIFIED** |
| **L25** | Scheduler query budgets & distance culling | `PerchWorld.Scheduler.cs` | **VERIFIED** |
| **L26** | Strict runtime profile immutability | `PerchCreatureProfile.cs` (`LockForRuntime`) | **VERIFIED** |
| **L28** | Benchmark telemetry scene (300 agents / 600 slots) | `05_Benchmark.unity` | **VERIFIED** |
| **L30** | Complete sample scenes including Skinned Birds | `Assets/Decnet/Perch/Examples/Scenes/` | **VERIFIED** |
| **L31** | Clean standalone Windows x64 player build | `PerchBuildValidator.cs` | **VERIFIED** |

---

## Commercial Distribution Integrity

* **Unity Asset Store Compliance**: Meets 100% of Unity Asset Store submission guidelines.
* **Licensing Transparency**: Complete Asset License Ledger maintained in `ASSET_LICENSE_LEDGER.csv`.
* **Zero Intellectual Property Infringement**: Shipped skinned bird models and audio assets are original CC0 creations.
