---
layout: default
title: "Vehicles, Ships & Drones"
parent: "Workflows & Recipes"
nav_order: 3
description: "Authoring landing spots on moving ships, trains, swinging branches, and drone docking stations."
permalink: /docs/workflows/moving-vehicles-drones/
---

# Vehicles, Ships & Drones
{: .fs-9 }

### Dynamic Moving Perches & Autonomous Quadcopter Docking
{: .fs-6 .text-grey-dk-000 }

---

Game worlds are full of dynamic movement: pirate ships rolling on ocean waves, trains speeding across deserts, hanging ropes swinging in breezes, and autonomous drones docking onto mobile command rovers.

PERCH handles moving perches with relative-coordinate stability and momentum inheritance.

---

## 1. Setting Up a Moving Perch

Follow these steps to author a landing spot on any moving object:

```
Moving Ship / Vehicle Transform
├── Rigidbody (Kinematic or Dynamic)
└── Railing GameObject
    ├── BoxCollider (Physical Support Collider)
    └── PerchSpot (Parented to the moving object)
        └── PerchSlot (Configured with local contact coordinates)
```

1. **Parenting**: Attach the `PerchSpot` component directly to the moving transform (or a child transform within its hierarchy).
2. **Assign Support Collider**: Drag the platform's physical collider into the slot's **Support Collider** field.
3. **Verify Uniform Scale**: Ensure the moving parent maintains uniform scale `(1, 1, 1)`. Non-uniform scale can distort contact orientation.

---

## 2. Autonomous Drone & Quadcopter Setup

Drones operate under fundamentally different flight mechanics than winged birds: they do not glide or flare; they descend vertically and spin down their rotors upon touchdown.

### Step 1: Creature Profile Configuration
1. In the drone's `PerchCreatureProfile`:
   * Set **Flight Style** to **`HoverDescent`**.
   * Set **Touchdown Envelope** to vertical descent ($80^\circ - 90^\circ$).
   * Set **Approach Speed** to $1.5\text{ m/s}$.

### Step 2: Contact Pad Configuration
1. On the landing dock, set the `PerchSlot` **Contact Mode** to **`Patch`**.
2. Define the patch dimensions (e.g. $1.0\text{m} \times 1.0\text{m}$) matching the pad surface.

### Step 3: Rotor Spin-Down Setup
1. On the drone root, add `PerchProceduralAnimationDriver`.
2. Assign the propeller transforms to the **Rotor Transforms** array.
3. Set **Max Rotor RPM** (e.g., `1200`).
4. **Touchdown Behavior**: As the drone lands, the driver automatically decelerates rotor RPM to zero over $0.5\text{s}$. Upon takeoff, rotors spool up to full RPM before the drone lifts off.

---

## 3. Platform Velocity Handover

When a creature takes off from a fast-moving vehicle (e.g., a train traveling at $15\text{ m/s}$), standard AI will drop the creature to zero world velocity, creating a jarring visual pop where the bird appears to get thrown backward.

PERCH's departure contract automatically calculates:
$$\mathbf{V}_{\text{exit}} = \mathbf{V}_{\text{launch}} + \mathbf{V}_{\text{platform}}$$

The creature carries the vehicle's momentum smoothly into free flight, creating natural, physically believable motion.
