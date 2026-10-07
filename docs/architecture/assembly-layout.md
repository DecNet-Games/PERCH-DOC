---
layout: default
title: "Assembly Definition Partitioning"
parent: "Architecture"
nav_order: 5
description: "Strict assembly partitioning, zero UnityEditor leakage in runtime builds, and standalone player validation."
permalink: /docs/architecture/assembly-layout/
---

# Assembly Definition Partitioning
{: .fs-9 }

### Strict Assembly Boundaries for Clean Compilation & Zero Editor Leakage
{: .fs-6 .text-grey-dk-000 }

---

A frequent flaw in third-party Unity assets is accidental coupling between editor scripts and runtime code. A single missing `#if UNITY_EDITOR` directive or unpartitioned assembly will break automated CI/CD pipelines and cause compilation failures in standalone player builds.

PERCH enforces **Strict Assembly Definition Partitioning (DEC-001)** across five isolated assemblies.

```
+-----------------------------------------------------------------------------------+
|                            PERCH ASSEMBLY ARCHITECTURE                            |
+-----------------------------------------------------------------------------------+
|  [Decnet.Perch.Editor]          --> References Runtime Assembly Only              |
|  [Decnet.Perch.Examples]        --> Isolated Sample Content & UI Scripts          |
|  [Decnet.Perch.Tests.EditMode]  --> Test Assembly (Stripped from Player Builds)   |
|  [Decnet.Perch.Tests.PlayMode]  --> Test Assembly (Stripped from Player Builds)   |
|  [Decnet.Perch.Runtime]         --> ZERO Editor References / Cross-Platform Ready |
+-----------------------------------------------------------------------------------+
```

---

## The Five Assemblies

| Assembly Name | Target Platforms | Dependencies | Purpose |
|:---|:---|:---|:---|
| **`Decnet.Perch.Runtime`** | All Platforms | None (Pure Engine) | Core runtime engine: state machine, spatial hash, lease manager, trajectories, motors, IK solver. Strictly zero `UnityEditor` references. |
| **`Decnet.Perch.Editor`** | Editor Only | `Decnet.Perch.Runtime` | PERCH Studio, authoring tools, inspectors, candidate scanner, rehearsal scrubbers, scene switchers. |
| **`Decnet.Perch.Examples`** | All Platforms | `Decnet.Perch.Runtime` | Demo scene controllers, camera close-up scripts, UI controls, configured creature prefabs. |
| **`Decnet.Perch.Tests.EditMode`**| Editor Only | Runtime, Editor, UTF | EditMode unit tests for math, splines, and profile immutability. Marked as **Test Assembly**. |
| **`Decnet.Perch.Tests.PlayMode`**| Editor & Players | Runtime, UTF | PlayMode integration tests for physics, contention, and state transitions. Marked as **Test Assembly**. |

---

## Standalone Player Verification Guarantee

* **Zero Editor Assembly Leakage**: `Decnet.Perch.Runtime` compiles cleanly with zero `#if UNITY_EDITOR` branching required for its core types.
* **Automated Build Validation**: Verified via `PerchBuildValidator.cs` in Unity 6 LTS standalone Windows 64-bit release builds.
* **Clean Stripping**: Test assemblies and editor tools are stripped automatically from player builds, resulting in zero unused binary bloat in your final game executable.
