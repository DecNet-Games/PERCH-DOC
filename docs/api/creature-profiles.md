---
layout: default
title: "Creature Profiles & Presets"
parent: "API Reference"
nav_order: 3
description: "ScriptableObject profile schema, aerodynamic parameters, runtime immutability lock, and shipped presets."
permalink: /docs/api/creature-profiles/
---

# Creature Profiles & Presets
{: .fs-9 }

### Aerodynamic Capabilities, Physical Envelopes & Strict Runtime Immutability
{: .fs-6 .text-grey-dk-000 }

---

A creature's physical dimensions, kinematic capabilities, and decision timings are defined inside a **`PerchCreatureProfile`** ScriptableObject asset. Multiple creatures of the same species can share a single profile asset.

---

## Profile Parameter Schema

### 1. Physical Dimensions & Envelopes
* **`BodyLength`**: Longitudinal length of the creature's torso (default: `0.5m`).
* **`BodyRadius`**: Conservative radial clearance hull enclosing the torso (default: `0.12m`).
* **`ClearanceMargin`**: Safety buffer added to all obstacle collision sweeps (default: `0.04m`).
* **`ContactSpan`**: Lateral span required for foot/gear placement (default: `0.20m`).
* **`ReservedFootprintRadius`**: Radius of the exclusion cylinder reserved when perched (default: `0.50m`).

### 2. Kinematics & Aerodynamics
* **`CruiseSpeed`**: Target free-flight roaming speed (default: `4.0m/s`).
* **`ApproachSpeed`**: Deceleration speed entering the landing corridor (default: `2.0m/s`).
* **`TouchdownRelativeSpeed`**: Maximum allowable contact speed at touchdown (strictly $\le 0.25\text{m/s}$).
* **`MaxAcceleration`**: Maximum forward thrust acceleration (default: `6.0m/s²`).
* **`MaxDeceleration`**: Maximum braking drag deceleration during approach (default: `8.0m/s²`).
* **`MaxTurnRate`**: Maximum angular yaw turning velocity (default: `120°/s`).

### 3. Moving Platform Limits
* **`MaxPlatformLinearSpeed`**: Upper limit for moving platform velocity (default: `2.0m/s`).
* **`MaxPlatformAngularSpeed`**: Upper limit for moving platform rotation (default: `30°/s`).
* **`MaxPlatformAcceleration`**: Upper limit for platform acceleration spikes (default: `3.0m/s²`).

### 4. Dwell & Policy Timings
* **`MinimumFlightTime`**: Mandatory roaming duration before auto-landing policy triggers (default: `5.0s`).
* **`MinDwellTime` / `MaxDwellTime`**: Randomized rest duration spent on a perch (default: `3.0s - 8.0s`).
* **`RetryCooldown`**: Waiting period after a failed landing attempt before trying again (default: `2.0s - 5.0s`).

---

## Strict Runtime Immutability (DEC-011)

In Unity, modifying ScriptableObject fields at runtime is a dangerous antipattern: changes persist across scene reloads, modify asset files on disk, and cause race conditions across multi-agent flocks.

PERCH enforces **Strict Runtime Immutability**:
* When an agent acquires a profile on startup, `profile.LockForRuntime()` is invoked.
* All property setters are locked.
* Any script attempting to mutate profile fields at runtime will immediately throw an `InvalidOperationException`.
* To modify creature capabilities at runtime, create a cloned runtime profile instance via `ScriptableObject.Instantiate(profile)`.

---

## Shipped Presets

PERCH includes four pre-tuned profile presets located in `Assets/Decnet/Perch/Resources/Profiles/`:
1. **`Profile_Songbird`**: Tuned for small agile birds (Finch, Wren, Sparrow). Rapid $180^\circ/\text{s}$ turn rate, small $0.3\text{m}$ footprint.
2. **`Profile_Pigeon`**: Tuned for medium birds. Moderate $120^\circ/\text{s}$ turn rate, steady $4.0\text{m/s}$ cruise.
3. **`Profile_BirdOfPrey`**: Tuned for eagles and hawks. High $7.0\text{m/s}$ cruise, wide $0.8\text{m}$ footprint, long approach flares.
4. **`Profile_Quadcopter`**: Configured for `HoverDescent` mode. Vertical descents, $0\text{m/s}$ mid-air hover capability.
