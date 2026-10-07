---
layout: default
title: "Animation Coordination & IK"
parent: "Core Features"
nav_order: 5
description: "Mecanim clip drivers, procedural harmonic wing solvers, manual Playables mixers, and Generic rig two-bone IK."
permalink: /docs/features/animation-systems/
---

# Animation Coordination & IK
{: .fs-9 }

### Multi-Paradigm Animation Coordination for Organic & Mechanical Creatures
{: .fs-6 .text-grey-dk-000 }

---

Animation is where most landing systems fail visually. If the wing fold timing does not match the braking trajectory, or if talons hover 10 centimeters above a branch, the entire illusion collapses.

PERCH provides three distinct animation drivers alongside a dedicated **Generic Rig Two-Bone IK Solver**.

| Procedural Wing Fold & Dwell Stance | Inspector Rig Bindings |
|:---:|:---:|
| ![Procedural Wing Fold]({{ site.baseurl }}/assets/images/05_Fold_Rest.png) | ![Rig Bindings Inspector]({{ site.baseurl }}/assets/images/14_Rig_Bindings.png) |
| *Planted foot stance with smooth wing fold hold* | *Auto-detected wing bones and contact anchors* |

---

## The Three Animation Paradigms

Depending on your asset pipeline, performance budget, and creature type, choose the driver that matches your workflow:

### 1. Clip Mode (`PerchClipAnimationDriver`)
For authored characters driven by standard Unity Mecanim Animator controllers:
* Interacts with Animator state machines using configurable parameters:
  * `IsFlying` (bool)
  * `IsGliding` (bool)
  * `IsLanding` (bool)
  * `IsPerched` (bool)
  * `IsTakingOff` (bool)
  * `PhaseProgress` (float, $0.0 \rightarrow 1.0$)
* **Crossfade Timing**: Uses `CrossFadeInFixedTime` ($0.15\text{s} - 0.25\text{s}$) to blend flight into flare and touchdown without pop.
* **Root Motion Invariant**: Disable root motion translation in your Animator; PERCH's motor owns world translation.

### 2. Procedural Mode (`PerchProceduralAnimationDriver`)
For stylized creatures, background flocks, or mechanical drones that lack dedicated landing clips:
* **Harmonic Wing Flapping**: Evaluates continuous sinusoidal oscillations directly on wing bone transforms:
  $$\theta_{\text{wing}}(t) = A \cdot \sin(2\pi f t + \phi)$$
  * Frequency ($f$) and amplitude ($A$) scale dynamically with flight speed and banking angle.
  * Flares outward during approach and folds smoothly against the body upon touchdown.
* **Perched Breathing & Sway**: Applies subtle, low-frequency pitch and yaw balancing while perched, keeping creatures alive without translating locked feet.
* **Rotor Spin-Down**: For mechanical drones, rotates propeller transforms at configurable RPM, automatically decelerating to a stop upon contact and spooling up prior to launch.

### 3. Direct Playables Mixer (`PerchPoseClipDriver`)
Used in the flagship `06_RealSkinnedBirds.unity` scene for maximum fidelity:
* Operates a custom Unity Playables Graph mixing five rig-specific authored clips: Flight, Glide, Landing/Fold, Perched Idle, and Takeoff.
* **Stance Pinning**: Calibrated toe markers and leg stance bones are held completely rigid during the unfold pause, eliminating any foot sliding while the wings prepare for launch.
* Restores Animator state cleanly on disable.

---

## Generic Rig Two-Bone IK Solver

Unity's built-in `OnAnimatorIK` is restricted to **Humanoid** avatars. Because virtually all birds, beasts, dragons, and insects on the Asset Store use **Generic** animation rigs, native Unity IK cannot place bird feet onto branches.

PERCH includes `PerchTwoBoneIkSolver`, an analytical, high-efficiency two-bone inverse kinematics solver built specifically for Generic rigs:

```
[Hip / RootJoint]
      \
       \ (Upper Leg)
        \
      [Knee / MidJoint]
        /
       / (Lower Leg)
      /
[Ankle / EndJoint] ===> Locked to Surface Collider Normal
```

### Characteristics:
* **Zero GC**: Implemented as a lightweight C# struct/serializable helper class—**not a heavy MonoBehaviour**.
* **Analytic Solution**: Computes closed-form trigonometric joint rotations in sub-microsecond time.
* **Joint Limits**: Rejects extreme solutions that would hyper-extend or twist the joint unnaturally (`ContactUnreachable`).
* **Pole Vector Hints**: Supports optional pole targets to control knee bend orientation.

```csharp
// Example IK setup via PerchRigBinding
var ik = new PerchTwoBoneIkSolver(hipBone, kneeBone, footBone);
ik.Weight = 1.0f;
ik.Solve(contactHitPoint, contactSurfaceRotation);
```

---

## Rig Binding & Automated Bone Detection

Configuring bones on dozens of bird species can be tedious. The `PerchRigBinding` component includes **`AutoDetectVisualsAndBones()`**:

1. In the Inspector, click **Auto-Detect Bones**.
2. PERCH parses the child transform hierarchy using intelligent naming heuristics:
   * Left Wing: `Wing_L`, `Wing.L`, `LeftWing`, `wing_l`
   * Right Wing: `Wing_R`, `Wing.R`, `RightWing`, `wing_r`
   * Legs & Talons: `Leg_L`, `Foot_L`, `Talon_L`, `Toe_L`
3. Automatically sets the **Contact Anchor** to the lowest mesh vertex cluster.
4. Generates a non-destructive visual wrapper to preserve clean `(1, 1, 1)` scale on the agent root.

> [!WARNING]
> **Single Pose Writer Invariant**: Exactly one animation driver may write to a skeleton at any time. Never attach `PerchProceduralAnimationDriver` and `PerchClipAnimationDriver` to the same GameObject.
