---
layout: default
title: API Reference
nav_order: 8
has_children: true
description: "Public C# API reference, method signatures, events, and integration contracts for PERCH."
permalink: /docs/api/
---

# Public API Reference
{: .fs-9 }

### Clean, Strongly Typed C# Interfaces and Event Hooks
{: .fs-6 .text-grey-dk-000 }

---

PERCH exposes a concise, fully documented public API under the `Decnet.Perch` namespace. All public classes, structs, and methods include complete XML doc comments and avoid unnecessary boilerplate.

---

## API Modules

Explore the core public interfaces:

### 1. [PerchAgent API]({{ site.baseurl }}/docs/api/perch-agent/)
The primary controller attached to creature GameObjects. Details on commands (`RequestLanding`, `RequestSlot`, `Depart`, `CancelLanding`, `PauseAgent`) and UnityEvents (`OnStateChanged`, `OnLanded`, `OnDeparted`, `OnLandingFailed`).

### 2. [Flight Provider API]({{ site.baseurl }}/docs/api/flight-provider/)
The contract for bridging external flight AI systems to PERCH. Implementation details for `IPerchFlightProvider`, motion state capture, and handover methods.

### 3. [Creature Profiles & Presets]({{ site.baseurl }}/docs/api/creature-profiles/)
The `PerchCreatureProfile` ScriptableObject schema, kinematic parameters, physical tolerances, and the `LockForRuntime()` immutability guarantee.
