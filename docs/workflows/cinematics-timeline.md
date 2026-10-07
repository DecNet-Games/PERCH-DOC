---
layout: default
title: "Cinematics & Timeline Integration"
parent: "Workflows & Recipes"
nav_order: 4
description: "Scripted landing sequences, cutscene integration, Timeline signals, and deterministic playback."
permalink: /docs/workflows/cinematics-timeline/
---

# Cinematics & Timeline Integration
{: .fs-9 }

### Directing Precision Narrative Landings in Cutscenes
{: .fs-6 .text-grey-dk-000 }

---

In narrative games, wildlife needs to do more than wander aimlessly: an owl must land on a wizard's shoulder during dialogue; a message hawk must perch on a tavern window as the player enters; or an eagle must take flight at a dramatic cinematic beat.

PERCH provides dedicated mechanisms for deterministic, scripted narrative landings.

---

## 1. Forcing Slot Selection via Script

To command a creature to land on a specific target perch regardless of ambient wandering policy, call `RequestSlot` with `force: true`:

```csharp
using UnityEngine;
using Decnet.Perch;

public class QuestDialogueCutscene : MonoBehaviour
{
    [SerializeField] private PerchAgent questHawk;
    [SerializeField] private PerchSpot tavernWindowSpot;

    public void TriggerHawkArrival()
    {
        var handle = new SlotHandle(
            questHawk.World.Id,
            tavernWindowSpot.Id,
            tavernWindowSpot.Slots[0].Id,
            tavernWindowSpot.Generation
        );

        // force: true bypasses auto-landing cooldowns and candidate scoring
        CommandResult result = questHawk.RequestSlot(handle, force: true);
        
        if (!result.IsAccepted)
        {
            Debug.LogWarning($"Cutscene landing could not start: {result.Reason}");
        }
    }
}
```

> [!NOTE]
> Even with `force: true`, PERCH still executes Hermite trajectory solving, obstacle sweeps, and two-bone IK touchdown solves. The bird will not teleport; it will fly and land naturally along physical curves.

---

## 2. Unity Timeline Integration

To trigger landings from a Unity **Timeline** track:
1. In your Timeline sequence, create a **Signal Track**.
2. Add a Signal Emitter keyframe at the exact second the creature should initiate its landing approach.
3. In the Signal Receiver component on the GameObject, wire the event to invoke your script's `TriggerHawkArrival()` method.
4. Add another Signal Emitter later in the Timeline to invoke `questHawk.Depart()` when the creature should take flight.

---

## 3. Deterministic Playback Guarantees

For cutscenes and automated cinematic recording:
* **Fixed Simulation Seed**: You can assign an explicit integer seed to `PerchWorld.SimulationSeed`.
* **Reproducibility**: With a fixed simulation seed and identical starting transform coordinates, trajectory calculations and approach curves will produce identical frame-by-frame results across recorded takes.
