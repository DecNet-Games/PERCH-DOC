---
layout: default
title: "Flight Provider API"
parent: "API Reference"
nav_order: 2
description: "Implementation guide for IPerchFlightProvider to integrate custom AI flight and navigation systems."
permalink: /docs/api/flight-provider/
---

# Flight Provider API
{: .fs-9 }

### Integrating Custom Navigation AI, Boids & Spline Paths
{: .fs-6 .text-grey-dk-000 }

---

While PERCH ships with a built-in volume and waypoint flight provider (`PerchWaypointFlightProvider`), games frequently use their own flight algorithms: boids simulation, steering behaviors, NavMesh flying extensions, or authored cutscene paths.

To connect any external navigation system to PERCH, implement **`IPerchFlightProvider`** on your creature root.

---

## The `IPerchFlightProvider` Interface Contract

```csharp
using UnityEngine;
using Decnet.Perch;

public interface IPerchFlightProvider
{
    /// <summary>
    /// Returns true if the external navigation system can safely yield root control.
    /// </summary>
    bool CanYield(PerchAgent agent);

    /// <summary>
    /// Captures the creature's current kinematic motion state for smooth spline blending.
    /// </summary>
    MotionState CaptureMotion(PerchAgent agent);

    /// <summary>
    /// Called when PERCH assumes control of the root for approach/perching.
    /// </summary>
    void OnPerchControlAcquired(PerchAgent agent);

    /// <summary>
    /// Called when PERCH releases control back to the provider following takeoff.
    /// </summary>
    void OnPerchControlReleased(PerchAgent agent, Vector3 exitPosition, Vector3 exitVelocity);

    /// <summary>
    /// Computes movement commands during FreeFlight state.
    /// </summary>
    MotionCommand ComputeFreeFlight(PerchAgent agent, float deltaTime);
}
```

---

## Custom Flight Provider Implementation Template

Here is a clean implementation template connecting a custom wandering behavior:

```csharp
using UnityEngine;
using Decnet.Perch;

public class CustomFlockFlightProvider : MonoBehaviour, IPerchFlightProvider
{
    [SerializeField] private float flightSpeed = 4.5f;
    [SerializeField] private float turnSpeed = 90.0f;

    private bool _isSuspended = false;
    private Vector3 _currentVelocity;

    public bool CanYield(PerchAgent agent)
    {
        // Return false if playing a stun or hit reaction
        return !_isSuspended;
    }

    public MotionState CaptureMotion(PerchAgent agent)
    {
        return new MotionState(
            transform.position,
            transform.rotation,
            _currentVelocity,
            Vector3.zero
        );
    }

    public void OnPerchControlAcquired(PerchAgent agent)
    {
        // Suspend custom steering updates
        _isSuspended = true;
    }

    public void OnPerchControlReleased(PerchAgent agent, Vector3 exitPosition, Vector3 exitVelocity)
    {
        // Re-enable custom steering and resume using departure momentum
        _isSuspended = false;
        transform.position = exitPosition;
        _currentVelocity = exitVelocity;
    }

    public MotionCommand ComputeFreeFlight(PerchAgent agent, float deltaTime)
    {
        if (_isSuspended) return default;

        // Custom steering logic (e.g. wander or follow target)
        Vector3 targetDirection = transform.forward;
        Vector3 newPos = transform.position + targetDirection * (flightSpeed * deltaTime);
        Quaternion newRot = transform.rotation;

        return new MotionCommand(newPos, newRot, targetDirection * flightSpeed);
    }
}
```

---

## Tuning Waypoint Arrival Radius

If you are using the included `PerchWaypointFlightProvider`, tune the **Arrival Radius** relative to the creature's **Turn Rate**:
* **The Orbiting Bug**: If a creature's arrival radius is set to $0.5\text{ m}$, but at its current cruise speed its minimum turning circle radius is $2.0\text{ m}$, the creature will orbit the waypoint endlessly without ever triggering arrival!
* **Formula**:
  $$R_{\text{min}} = \frac{V_{\text{cruise}}}{\omega_{\text{turn\_rad}}}$$
  Ensure `ArrivalRadius` is strictly greater than or equal to $R_{\text{min}}$.
