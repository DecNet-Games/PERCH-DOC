---
layout: default
title: Shipped Creatures
nav_order: 10
has_children: true
description: "Detailed breakdowns of the original skinned models, rigs, and flight profiles included in PERCH."
permalink: /docs/creatures/
---

# Shipped Creatures
{: .fs-9 }

### Original Stylized Skinned Models, Authored Clips & Calibrated Presets
{: .fs-6 .text-grey-dk-000 }

---

Unlike tools that ship with primitive cubes or unrigged placeholders, PERCH includes **production-ready, original stylized 3D skinned creature models** complete with authored animation clips, URP materials, and pre-tuned creature profiles.

| Garden Finch (Male) | Willow Wren |
|:---:|:---:|
| ![Garden Finch]({{ site.baseurl }}/assets/images/08_Garden_Finch.png) | ![Willow Wren]({{ site.baseurl }}/assets/images/09_Willow_Wren.png) |
| *Original CC0 stylized finch model with 5 authored clips* | *Original CC0 stylized wren model with agile flight curves* |

---

## Included Creature Rigs

Explore the technical specifications and setup for each shipped creature:

### 1. [Garden Finch]({{ site.baseurl }}/docs/creatures/garden-finch/)
The flagship hero bird. Stylized passerine anatomy, 5 distinct authored animation clips (Flight, Glide, Landing/Fold, Perched Idle, Takeoff), calibrated toe markers, and balanced flight kinematics.

### 2. [Willow Wren]({{ site.baseurl }}/docs/creatures/willow-wren/)
A secondary agile songbird featuring a different bone hierarchy and smaller physical envelope ($0.3\text{m}$ wingspan). Proves that PERCH handles distinct rig topologies without custom code changes.

### 3. [Quadcopter Drone]({{ site.baseurl }}/docs/creatures/drone-quadcopter/)
A mechanical inspection drone demonstrating the `HoverDescent` motion style, automated rotor spin-down, and flat docking pad contact alignment.
