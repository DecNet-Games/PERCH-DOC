---
layout: default
title: "Automated Test Suites"
parent: "Quality Assurance"
nav_order: 1
description: "EditMode and PlayMode automated test suites, contention stress tests, and leaked collider cleanup."
permalink: /docs/qa/test-suites/
---

# Automated Test Suites
{: .fs-9 }

### 100% Green Test Suites Across EditMode & PlayMode
{: .fs-6 .text-grey-dk-000 }

---

PERCH includes an extensive suite of automated tests built using the **Unity Test Framework (UTF 1.7.0)**. These tests guarantee that core mathematics, spatial hashing, lease contention, and lifecycle transitions remain rock-solid across updates.

---

## Test Assemblies

The test suites are partitioned into two isolated assemblies:
* **`Decnet.Perch.Tests.EditMode`**: Validates pure mathematical solvers, arc-length LUT spline parameterization, candidate scoring algorithms, and profile serialization.
* **`Decnet.Perch.Tests.PlayMode`**: Validates active frame simulation, physics sweeps, state machine transitions, moving platform tracking, and two-bone IK solver convergence.

> [!NOTE]
> Test assemblies are explicitly marked as **Test Assemblies** in their `.asmdef` configuration. They are automatically stripped from standalone player builds, ensuring zero test code overhead in shipping games.

---

## Test Coverage Highlights

### 1. Multi-Agent Contention Stress Test (`T-LEASE-01`)
* Executes `1,000` randomized contention batches across 50 simulated agents competing for 10 slots.
* **Assertion**: Exactly one owner acquires a lease; zero conflicting owners; zero deadlocks.

### 2. Arc-Length LUT Spline Accuracy (`T-MATH-04`)
* Verifies that the numerical arc-length LUT parameterization matches analytical curve length within a $0.001\text{m}$ margin of error.

### 3. Steady-State Zero-GC Allocation (`T-PERF-02`)
* Runs 100 simulation ticks with active creatures in PlayMode.
* **Assertion**: `GC.GetTotalMemory()` delta across steady-state ticks is strictly **0 Bytes**.

### 4. Leaked Collider Cleanup Guarantee (DEC-009)
In automated PlayMode testing, dynamically spawned test GameObjects and colliders can leak across sequential tests, causing false positives.
* In `PerchPlayModeTestRunner.cs`, every test is wrapped in strict `try-finally` blocks.
* Calls `PlayModeIntegrationTests.CleanupLeakedObjects()`.
* Calls `Physics.SyncTransforms()` to ensure clean physics state for subsequent test runs.

---

## Running the Tests in Unity Editor

1. Open `Window > General > Test Runner`.
2. To run editor unit tests:
   * Select the **EditMode** tab and click **Run All**.
3. To run integration and physics tests:
   * Select the **PlayMode** tab and click **Run All**.
4. All tests should pass with green checkmarks (22 PASSED / 0 FAILED).
