---
layout: default
title: "Deterministic State Machine"
parent: "Architecture"
nav_order: 1
description: "The 10-state deterministic finite state machine, state transition rules, and reentrancy guarantees."
permalink: /docs/architecture/state-machine/
---

# Deterministic State Machine
{: .fs-9 }

### Formal State Transitions, Timing Deadlines & Reentrancy Guards
{: .fs-6 .text-grey-dk-000 }

---

At the heart of every `PerchAgent` is a formal, deterministic finite state machine (FSM). Rather than loosely polling boolean flags across multiple components, all creature transitions are governed by strict lifecycle invariants.

---

## The 10 States of `PerchState`

```
  +--------------+
  |   Disabled   |
  +-------+------+
          | (OnEnable)
          v
  +--------------+
  |  FreeFlight  | <======================================================+
  +-------+------+                                                        |
          | (Decision / Request)                                          |
          v                                                               |
  +--------------+                                                        |
  |  Selecting   | ──(No Candidates)──> [Return to FreeFlight on Cooldown]|
  +-------+------+                                                        |
          | (Candidate Selected)                                          |
          v                                                               |
  +--------------+                                                        |
  |   Planning   | ──(Obstacle / Failure)──> [Release Lease & Cooldown]   |
  +-------+------+                                                        |
          | (Lease Claimed & Corridor Clear)                              |
          v                                                               |
  +--------------+                                                        |
  | Approaching  | ──(Obstacle / Lease Lost)──> +---------------+         |
  +-------+------+                             | AbortRecovery | ─────────+
          | (Contact Threshold)                +-------+-------+          |
          v                                            | (Trapped)        |
  +--------------+                                     v                  |
  |  Touchdown   |                             +----------------------+   |
  +-------+------+                             | InterventionRequired |   |
          | (Stance Locked & Occupied)         +----------------------+   |
          v                                                               |
  +--------------+                                                        |
  |   Perched    |                                                        |
  +-------+------+                                                        |
          | (Dwell Expired / Depart Command / Panic Wave)                 |
          v                                                               |
  +--------------+                                                        |
  |  TakingOff   | ──(Body Clears Footprint & Releases Slot)──────────────+
  +--------------+
```

### Detailed State Invariants

| State | Motion Owner | Primary Invariant & Behavior |
|:---|:---|:---|
| **`Disabled`** | None | Agent is unregistered from `PerchWorld`. All leases released, motors detached. |
| **`FreeFlight`** | `ExternalProvider` | Driven by external flight AI. Enforces minimum flight roaming time before auto-landing triggers. |
| **`Selecting`** | `ExternalProvider` | Filters slots by tags, size, and distance. Evaluates candidate suitability scores. |
| **`Planning`** | `ExternalProvider` | Atomically claims lease token. Computes Hermite spline and performs swept collision checks. |
| **`Approaching`** | `PerchMotor` | PERCH motor assumes exclusive control. Decelerates along arc-length LUT spline. Sends 0.25s heartbeats. |
| **`Touchdown`** | `PerchMotor` | Final contact alignment. Solves two-bone IK leg placement. Transitions slot to `Occupied`. |
| **`Perched`** | `PerchMotor` | Creature rests on perch. Evaluates wing fold, breathing sway, dwell timers, and moving platform deltas. |
| **`TakingOff`** | `PerchMotor` | Pre-flight departure sweeps verify exit path. Launches into air. Holds exclusion footprint until clear. |
| **`AbortRecovery`** | `PerchMotor` | Safely steers away from unexpected obstacles before returning control to `ExternalProvider`. |
| **`InterventionRequired`** | `PerchMotor` | Triggered if an agent is completely entrapped without any escape route. Holds safe pose without crashing. |

---

## Reentrancy Safety (DEC-010)

A classic vulnerability in game architecture is **event callback reentrancy**: a user attaches an event listener to `OnStateChanged`, and inside that listener immediately calls `agent.CancelLanding()` or `agent.RequestLanding()`. In poorly architected systems, this mutates internal collections while the state machine is iterating, causing collection modified exceptions or infinite recursive loops.

PERCH solves this using **Preallocated Struct Circular Buffers**:
* Commands are queued into a fixed-size `PendingCommand[16]` ring buffer.
* Events are buffered into a `QueuedAgentEvent[32]` ring buffer.
* The state machine drains events using a frozen snapshot count.
* Any reentrant commands issued during an event callback are safely enqueued and processed on the subsequent tick, ensuring complete reentrancy safety and zero heap allocations.
