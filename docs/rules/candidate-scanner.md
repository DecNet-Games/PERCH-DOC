---
layout: default
title: "Surface Candidate Scanner"
parent: "Diagnostics & Tools"
nav_order: 3
description: "Automated raycast-based geometry scanning for discovering and authoring landing perches on complex meshes."
permalink: /docs/rules/candidate-scanner/
---

# Surface Candidate Scanner
{: .fs-9 }

### Rapid Automated Perch Discovery Across Complex 3D Geometry
{: .fs-6 .text-grey-dk-000 }

---

Hand-placing dozens of landing spots along intricate stone battlements, winding tree branches, or rocky cliffs takes hours of repetitive manual alignment. The **Candidate Perch Scanner** automates this by casting downward rays across selected scene colliders and generating valid `PerchSpot` candidates.

Open it via:
* **Menu**: `Window > Perch > Candidate Scanner` (or via the **Scanner** tab in PERCH Studio).

---

## Scanning Workflow

```
[Select Geometry] ──> [Configure Filters] ──> [Run Raycast Scan] ──> [Preview Points] ──> [Generate Spots (Undo-Safe)]
```

### Step 1: Select Environment Geometry
Select the GameObjects containing colliders you want birds to land on (e.g., fence rails, rooftops, trees, or cliff edges).

### Step 2: Configure Sampling Filters
* **Sampling Grid Spacing**: Distance between raycast samples (e.g., `0.5m - 2.0m`).
* **Max Surface Slope**: Maximum allowable deviation from vertical up (e.g., $30^\circ$). Surfaces steeper than this angle are automatically rejected.
* **Overhead Clearance Radius**: Casts an upward sphere check from the hit point to ensure the landing area has sufficient open airspace for wings and approaches.
* **Target Layer Mask**: Restricts raycasts to specific environment layers.

### Step 3: Run the Scan & Preview Candidates
Click **Scan Geometry**. The tool draws interactive gizmos in the Scene View:
* **Cyan Discs**: Valid prospective landing spots with correct surface normals.
* **Red Markers**: Rejected sample points (too steep, obstructed overhead, or insufficient clearance).

### Step 4: Generate Spots
Review the proposed points in the Scene View. Click **Generate Perch Spots**:
* Generates configured `PerchSpot` GameObjects at the confirmed coordinates.
* Automatically links the hit collider as the slot's **Support Collider**.
* Aligns Local Y with the sampled surface normal.
* Fully registered with Unity's `Undo` system (`Ctrl+Z` immediately removes all generated spots).

> [!TIP]
> The scanner provides intelligent starting suggestions. Always inspect generated points on thin wires or complex foliage to verify that the creature's intended approach corridor is not blocked by decorative leaves.
