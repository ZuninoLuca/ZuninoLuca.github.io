---
layout: page
title: Crazyflie autonomous mission
description: Final project for the EPFL course "Aerial Robotics"
img: assets/img/project_covers/crazyflie.jpg
importance: 4
category: master's
---

<div class="row justify-content-center mt-1 mb-4">
    <div class="col-sm-2 col-4 text-center">
        <a href="{{ '/assets/pdf/CrazyPractical_Project_Slides.pdf' | relative_url }}" class="btn btn-sm btn-outline-primary" target="_blank">
            <i class="fa-solid fa-desktop"></i> Slides
        </a>
    </div>
    <div class="col-sm-2 col-4 text-center">
        <a href="https://github.com/ZuninoLuca/Aerial_robotics/" class="btn btn-sm btn-outline-primary" target="_blank">
            <i class="fab fa-github"></i> Code
        </a>
    </div>
</div>

## At a glance

This is the final project for the EPFL course **“Aerial Robotics”** (Prof. D. Floreano), completed in **Spring 2022** in collaboration with **Nay Abi Akl, Mariam Hassan, and Yehya El Hassan**.

The project, **“CrazyPractical,”** consisted of implementing the onboard software for the **Crazyflie** quadrotor to autonomously:

1. **Take off** from a starting pad  
2. **Navigate through an arena** while **avoiding obstacles**  
3. **Find and localize a landing pad**, then **land safely**  
4. **Take off again** from the landing pad  
5. **Return to the initial pad** and land, all within a strict time limit

This required a full autonomy stack combining **planning, reactive avoidance, and robust localization** under changing arena configurations.

---

## System in a nutshell

The pipeline follows a pragmatic “search → avoid → confirm → act” structure, split into two mission phases.

### Phase 1: Start pad → Landing pad
1. **Take-off** from the take-off pad  
2. **Sweep** to find a free corridor / direction of motion  
3. **Avoid obstacles** during navigation (reactive layer)  
4. **Sweep** to detect the landing pad  
5. **Landing-pad localization manoeuvre**, then **controlled landing**

### Phase 2: Landing pad → Start pad (return)
1. **Take-off** from the landing pad  
2. Choose a return-planning strategy:
   - **A\*** (global planning), or
   - Follow a **forward path** heuristic, or
   - **Re-discretize** a new path based on current conditions  
3. **Take-off pad localization manoeuvre**, then **landing**

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/aerial_robotics/CF_Strategy_1.png" title="Scheme of Phase 1 of the pipeline" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/aerial_robotics/CF_Strategy_2.png" title="Scheme of Phase 2 of the pipeline" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Left: Phase 1 of the pipeline: Start → Sweep → Avoid → Locate → Land. Right: Phase 2 of the pipeline: Take-off → Plan return (A* or heuristic) → Locate start → Land.
</div>

---

## What we implemented

### Autonomous navigation with obstacle avoidance
A key requirement was safe navigation in cluttered environments. The solution combines:
- A **high-level navigation strategy** (sweep-based exploration to progress and to search for pads)
- A **reactive obstacle avoidance** layer to handle nearby obstacles and last-moment corrections

This hybrid approach kept the drone moving toward objectives while remaining safe when the environment changed.

### Pad search and robust localization manoeuvres
Landing is only possible if the pad pose is estimated reliably. We implemented dedicated manoeuvres to:
- **Confirm pad detection**
- Reduce ambiguity through controlled motion (rather than trusting a single noisy observation)
- Align for **stable descent and touchdown**

A similar manoeuvre is used again during the return phase to localize the original take-off pad before landing.

### Return-path planning options (A* / heuristics / re-discretization)
For the second half of the mission, we included alternative strategies to plan the way back:
- **A\*** for a more globally consistent path when conditions allow it
- Simpler forward/heuristic approaches when speed or reliability is preferable
- **Re-discretization** to recompute a feasible path from the current state

This makes the system more flexible across different obstacle/pad configurations.

---

## Results (from our trials)

From the test summary we performed:
- **Obstacle avoidance:** 10/10 trials  
- **Landing on landing pad:** 9/10 trials  
- **Landing back on take-off pad:** 8/10 trials  
- **Time limit respected:** 8/10 trials  
- **Average completion time:** **1:51 minutes**

These numbers reflect end-to-end performance, including both landings and the full round trip.

---

## Demo video

This demo video shows the full mission: take-off → obstacle-aware navigation → landing-pad acquisition & landing → re-takeoff → return planning → final landing, completed in under two minutes on successful runs.

<div style="position:relative;padding-bottom:56.25%;height:0;overflow:hidden;border-radius:0.5rem; margin-bottom: 1.25rem;">
  <iframe
    src="https://www.youtube.com/embed/7H17Nvg9MPc"
    title="Crazyflie autonomous mission | Demo video"
    style="position:absolute;top:0;left:0;width:100%;height:100%;border:0;"
    allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share"
    allowfullscreen
    loading="lazy">
  </iframe>
</div>

---

## Acknowledgements

Course: EPFL — Aerial Robotics (Prof. D. Floreano)  
Team: Nay Abi Akl, Mariam Hassan, Yehya El Hassan, Luca Zunino<br>
Source of the cover image: [Matin, drone detection Dataset, Roboflow | CC BY 4.0](https://universe.roboflow.com/matin-pxex7/drone-detection-xre4f)