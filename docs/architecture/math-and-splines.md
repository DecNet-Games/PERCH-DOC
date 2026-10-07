---
layout: default
title: "Mathematics & Spline Trajectories"
parent: "Architecture"
nav_order: 4
description: "Mathematical formulation of cubic Hermite splines, arc-length numerical integration, banking angles, and flare physics."
permalink: /docs/architecture/math-and-splines/
---

# Mathematics & Spline Trajectories
{: .fs-9 }

### Mathematical Rigor Behind Arc-Length LUTs & Aerodynamic Flares
{: .fs-6 .text-grey-dk-000 }

---

This document provides a mathematical breakdown of the trajectory and flight mechanics implemented in `Decnet.Perch.Runtime.Motion`.

---

## 1. Cubic Hermite Spline Formulation

An approach curve connects the creature's current spatial state $(\mathbf{P}_0, \mathbf{V}_0)$ to the target landing pose $(\mathbf{P}_1, \mathbf{N}_{\text{normal}})$ over parametric domain $u \in [0, 1]$:

$$\mathbf{S}(u) = h_{00}(u)\mathbf{P}_0 + h_{10}(u)\mathbf{M}_0 + h_{01}(u)\mathbf{P}_1 + h_{11}(u)\mathbf{M}_1$$

Where the cubic Hermite basis functions are:
* $h_{00}(u) = 2u^3 - 3u^2 + 1$
* $h_{10}(u) = u^3 - 2u^2 + u$
* $h_{01}(u) = -2u^3 + 3u^2$
* $h_{11}(u) = u^3 - u^2$

Tangents $\mathbf{M}_0$ and $\mathbf{M}_1$ are scaled by curve chord length $\|\mathbf{P}_1 - \mathbf{P}_0\|$ and creature cruise speed to ensure continuous curvature and zero second-derivative discontinuities at boundary conditions.

---

## 2. Numerical Arc-Length Integration (DEC-004)

Because $\|\mathbf{S}'(u)\|$ varies along the curve, evaluating the spline at uniform parameter steps $\Delta u$ results in severe physical velocity fluctuations.

PERCH solves this by constructing a 32-sample cumulative arc-length look-up table:

$$s_k = \sum_{i=1}^{k} \frac{1}{2} \left( \|\mathbf{S}'(u_{i-1})\| + \|\mathbf{S}'(u_i)\| \right) \cdot \Delta u$$

Given a required travel distance $s \in [0, s_{\text{total}}]$:
1. Binary search finds index $k$ such that $s_k \le s < s_{k+1}$.
2. Linear interpolation resolves normalized factor $\alpha = \frac{s - s_k}{s_{k+1} - s_k}$.
3. The true parametric coordinate is resolved as:
   $$u = u_k + \alpha (u_{k+1} - u_k)$$

This guarantees constant, controllable physical deceleration ($a \le a_{\text{max}}$) along the entire flight path.

---

## 3. Dynamic Banking Physics

When a flying creature navigates a curved trajectory, aerodynamic lift must tilt to balance centrifugal force:

$$\tan(\phi) = \frac{v^2}{g \cdot R} = \frac{v^2 \cdot \kappa}{g}$$

Where:
* $\phi$ is the roll banking angle.
* $v$ is the creature's forward airspeed.
* $\kappa$ is the instantaneous spline curvature:
  $$\kappa(u) = \frac{\|\mathbf{S}'(u) \times \mathbf{S}''(u)\|}{\|\mathbf{S}'(u)\|^3}$$
* $g$ is gravitational acceleration ($9.81\text{ m/s}^2$).

PERCH caps $\phi$ against the profile's `MaxBankAngle` (default: $45^\circ$) to prevent excessive or unnatural inverted roll attitudes during sharp approach turns.
