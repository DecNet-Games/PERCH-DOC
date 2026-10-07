---
layout: default
title: "Procedural Environment Generation"
parent: "Workflows & Recipes"
nav_order: 5
description: "Authoring and registering perches programmatically at runtime in procedural levels and infinite worlds."
permalink: /docs/workflows/procedural-environment-generation/
---

# Procedural Environment Generation
{: .fs-9 }

### Registering Dynamic Perches at Runtime in Procedural Worlds
{: .fs-6 .text-grey-dk-000 }

---

In rogue-likes, endless runners, or procedural open-world games, levels are generated at runtime. Perches cannot be hand-placed in the editor; they must be created, registered, and destroyed dynamically as level chunks stream in and out.

PERCH provides an unconstrained runtime API for programmatic spot registration.

---

## 1. Runtime Spot Creation Recipe

The following C# recipe demonstrates how to create a `PerchSpot` and register it with the active `PerchWorld`:

```csharp
using UnityEngine;
using Decnet.Perch;

public class ProceduralBranchGenerator : MonoBehaviour
{
    [SerializeField] private PerchWorld world;
    [SerializeField] private Collider branchCollider;

    public void GenerateRuntimePerch(Vector3 branchStart, Vector3 branchEnd)
    {
        // 1. Create support GameObject
        GameObject perchGO = new GameObject("Procedural_PerchSpot");
        perchGO.transform.position = branchStart;
        perchGO.transform.rotation = Quaternion.LookRotation(branchEnd - branchStart);

        // 2. Attach PerchSpot
        PerchSpot spot = perchGO.AddComponent<PerchSpot>();
        spot.Initialize(world);

        // 3. Define and add slot records
        var slot = new PerchSlot
        {
            Id = 1,
            LocalPosition = Vector3.up * 0.1f,
            LocalRotation = Quaternion.identity, // Local Y is normal, Z is forward
            SupportCollider = branchCollider,
            ContactMode = ContactMode.Line,
            ReservedFootprintRadius = 0.5f,
            AllowedStyles = FlightStyle.ForwardGlide
        };

        spot.AddSlot(slot);

        // 4. Register with the world spatial hash
        world.RegisterSpot(spot);
    }
}
```

---

## 2. Managing Procedural Chunk Despawning

When a procedural world chunk streams out, its perches must be unregistered cleanly to prevent dangling references in the spatial hash:

```csharp
private void OnDestroy()
{
    if (world != null && spot != null)
    {
        // Unregisters from spatial hash and invalidates generation tokens
        world.UnregisterSpot(spot);
    }
}
```

### Safety Invariants During Despawn:
* If an incoming agent is currently approaching a slot on a despawning chunk, the unregister call increments the spot's `Generation` counter.
* The agent's heartbeat immediately detects that the lease was lost (`LeaseLost`) and triggers a safe abort recovery back to free flight.
* No null-reference exceptions or memory leaks are generated.
