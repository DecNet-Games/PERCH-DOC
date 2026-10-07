---
layout: default
title: "PerchAgent API"
parent: "API Reference"
nav_order: 1
description: "Complete C# method signatures, events, and properties for the PerchAgent component."
permalink: /docs/api/perch-agent/
---

# PerchAgent API
{: .fs-9 }

### The Core Runtime Controller for Flying Creatures
{: .fs-6 .text-grey-dk-000 }

---

`PerchAgent` is the primary MonoBehaviour component attached to creature roots. It manages the finite state machine, executes commands, and emits lifecycle events.

**Namespace**: `Decnet.Perch`  
**Inherits from**: `MonoBehaviour`

---

## Public Methods

### `RequestLanding()`
Commands the agent to search for the best available perch in the world and initiate a landing approach.
```csharp
public CommandResult RequestLanding()
```
* **Returns**: `CommandResult.Accepted` if the request was successfully queued, or `CommandResult.Rejected` with an immediate failure reason (e.g. if already perched or disabled).

### `RequestSlot(SlotHandle slot, bool force = false)`
Commands the agent to target a specific authored slot.
```csharp
public CommandResult RequestSlot(SlotHandle slot, bool force = false)
```
* **`slot`**: The targeted `SlotHandle`.
* **`force`**: If `true`, bypasses normal decision interval cooldowns and suitability scoring. (Note: physical clearance and occupancy checks are still strictly enforced).

### `Depart()`
Commands an occupied/perched agent to initiate its departure sequence and launch into free flight.
```csharp
public CommandResult Depart()
```

### `CancelLanding()`
Cancels an ongoing landing approach or candidate planning phase, commanding the agent to execute an abort curve and return to free flight.
```csharp
public CommandResult CancelLanding()
```

### `SetAutoLanding(bool enabled)`
Enables or disables autonomous background landing decisions.
```csharp
public void SetAutoLanding(bool enabled)
```

### `PauseAgent(bool isPaused)`
Freezes or unfreezes the individual agent. When paused, active leases and occupancy remain intact, but motion and dwell timers are frozen.
```csharp
public void PauseAgent(bool isPaused)
```

---

## Public Properties

```csharp
public PerchState CurrentState { get; }
public MotionOwner MotionOwner { get; }
public SlotHandle CurrentSlot { get; }
public FailureReason LastFailureReason { get; }
public DecisionStatus DecisionStatus { get; }
public bool IsPerched { get; }
public bool IsApproaching { get; }
public bool AutoLandingEnabled { get; set; }
```

---

## Public UnityEvents

`PerchAgent` exposes strongly typed UnityEvents that can be wired via Inspector or registered in code:

### `OnStateChanged`
Fires whenever the agent transitions between states.
```csharp
public UnityEvent<PerchAgent, PerchState, PerchState> OnStateChanged;
// Parameters: (agent, previousState, newState)
```

### `OnLanded`
Fires when the agent completes touchdown and locks its perched stance.
```csharp
public UnityEvent<PerchAgent, SlotHandle> OnLanded;
// Parameters: (agent, landedSlot)
```

### `OnDeparted`
Fires when the creature completes its departure curve and releases its slot.
```csharp
public UnityEvent<PerchAgent, SlotHandle> OnDeparted;
// Parameters: (agent, vacatedSlot)
```

### `OnLandingFailed`
Fires whenever a landing request aborts or fails.
```csharp
public UnityEvent<PerchAgent, FailureReason, uint> OnLandingFailed;
// Parameters: (agent, failureReason, requestId)
```

---

## Complete Integration Example

```csharp
using UnityEngine;
using Decnet.Perch;

public class BirdGameplayController : MonoBehaviour
{
    [SerializeField] private PerchAgent agent;
    [SerializeField] private PerchSpot targetSpot;

    private void OnEnable()
    {
        // Subscribe to lifecycle events
        agent.OnStateChanged.AddListener(HandleStateChanged);
        agent.OnLanded.AddListener(HandleLanded);
        agent.OnLandingFailed.AddListener(HandleLandingFailed);
    }

    private void OnDisable()
    {
        // Always clean up listeners
        agent.OnStateChanged.RemoveListener(HandleStateChanged);
        agent.OnLanded.RemoveListener(HandleLanded);
        agent.OnLandingFailed.RemoveListener(HandleLandingFailed);
    }

    public void CommandLandOnRail()
    {
        if (targetSpot != null && targetSpot.Slots.Count > 0)
        {
            // Build a valid slot handle
            var handle = new SlotHandle(
                agent.World.Id,
                targetSpot.Id,
                targetSpot.Slots[0].Id,
                targetSpot.Generation
            );

            CommandResult result = agent.RequestSlot(handle);
            if (!result.IsAccepted)
            {
                Debug.LogWarning($"Landing rejected: {result.Reason}");
            }
        }
    }

    private void HandleStateChanged(PerchAgent a, PerchState before, PerchState after)
    {
        Debug.Log($"Bird transitioned: {before} -> {after}");
    }

    private void HandleLanded(PerchAgent a, SlotHandle slot)
    {
        Debug.Log($"Bird successfully perched on slot {slot.SlotId}!");
    }

    private void HandleLandingFailed(PerchAgent a, FailureReason reason, uint requestId)
    {
        Debug.LogWarning($"Bird landing failed: {reason} (Request #{requestId})");
    }
}
```
