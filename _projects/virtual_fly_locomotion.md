---
layout: page
title: Virtual fly locomotion controllers
description: Final project for the EPFL course "Controlling behavior in animals and robots"
img: assets/img/project_covers/cover_fly.png
importance: 2
category: master's
---

<div class="row justify-content-center mt-1 mb-4">
    <div class="col-sm-2 col-4 text-center">
        <a href="{{ '/assets/pdf/COBAR_final_presentation.pdf' | relative_url }}" class="btn btn-sm btn-outline-primary" target="_blank">
            <i class="fa-solid fa-desktop"></i> Slides
        </a>
    </div>
    <div class="col-sm-2 col-4 text-center">
        <a href="{{ '/assets/pdf/COBAR_MiniProject_Report.pdf' | relative_url }}" class="btn btn-sm btn-outline-primary" target="_blank">
            <i class="fa-solid fa-file-pdf"></i> Report
        </a>
    </div>
    <div class="col-sm-2 col-4 text-center">
        <a href="https://github.com/ZuninoLuca/Controlling_behavior_in_animals_and_robots" class="btn btn-sm btn-outline-primary" target="_blank">
            <i class="fab fa-github"></i> Code
        </a>
    </div>
</div>

## At a glance

Final project for the EPFL course **“Controlling behavior in animals and robots”** (Prof. P. Ramdya), completed in **Spring 2023** with **Alaa Aboud** and **Flore Munier-Jolain**.

We developed controllers for **virtual fly locomotion** using **NeuroMechFly**, a biomechanical model of *Drosophila melanogaster* running in the **MuJoCo** physics engine (Lobato-Rios et al., *Nature Methods*, 2022). The project sits at the intersection of **neuroengineering** and **robotics**: using realistic simulation to deconstruct locomotion, test control hypotheses, and study what makes behavior robust across environments.

---

## Goal

Implement and analyze **three distinct locomotion controllers** for a simulated fly:

1. **Rule-based (decentralized) controller** inspired by Cruse-style coordination rules.
2. **CPG-based controller** using coupled oscillators (one per leg), extended with sensory feedback.
3. **Hybrid controller** that uses **reinforcement learning (PPO)** to optimize a *reduced* action space, combining the strengths of hand-engineered structure and learning-based adaptation.

All controllers were evaluated on **three terrains**:
- **Flat** terrain (baseline)
- **Blocks** terrain (rough, uneven heights; challenges stability)
- **Gapped** terrain (deep narrow gaps; challenges leg recovery)

<div class="row">
  <div class="col-sm mt-3 mt-md-0">
    <video autoplay loop muted playsinline class="img-fluid rounded z-depth-1"
           style="height: 300px; object-fit: cover;"
           title="Rule-based controller on blocks terrain">
      <source src="{{ '/assets/video/rule_blocks.mp4' | relative_url }}" type="video/mp4">
      Your browser does not support the video tag.
    </video>
  </div>
  <div class="col-sm mt-3 mt-md-0">
    <video autoplay loop muted playsinline class="img-fluid rounded z-depth-1"
           style="height: 300px; object-fit: cover;"
           title="CPG-based controller on blocks terrain">
      <source src="{{ '/assets/video/cpg_blocks.mp4' | relative_url }}" type="video/mp4">
      Your browser does not support the video tag.
    </video>
  </div>
  <div class="col-sm mt-3 mt-md-0">
    <video autoplay loop muted playsinline class="img-fluid rounded z-depth-1"
           style="height: 300px; object-fit: cover;"
           title="Hybrid controller on blocks terrain">
      <source src="{{ '/assets/video/hybrid_blocks.mp4' | relative_url }}" type="video/mp4">
      Your browser does not support the video tag.
    </video>
  </div>
</div>
<div class="caption">
    Locomotion comparison on blocks terrain: rule-based controller with Cruse-style coordination (left), CPG-based controller with coupled oscillators (center), and hybrid RL-optimized controller combining structured patterns with learned adaptation (right).
</div>

---

## Approach overview

### Biomechanical model and actuation choices
NeuroMechFly includes a detailed morphology (segments + joints). For locomotion we focused on the **7 DoFs per leg** relevant to walking, and used recorded stepping data to define **stance** and **swing** reference trajectories.

---

## Controller 1 — Rule-based (decentralized)

<div class="row justify-content-center">
  <div class="col-sm-8 mx-auto mt-3 mt-md-0">
    <video autoplay loop muted playsinline
           class="img-fluid rounded z-depth-1 d-block mx-auto"
           style="height: 300px; object-fit: cover;"
           title="Rule-based controller on gapped terrain">
      <source src="{{ '/assets/video/rule_gapped.mp4' | relative_url }}" type="video/mp4">
      Your browser does not support the video tag.
    </video>
  </div>
</div>
<div class="caption">
    Rule-based controller with Cruse-Style coordination on gapped terrain.
</div>

### Core idea
A decentralized controller where each leg’s stepping decision emerges from a **weighted sum of simple coordination rules**, reflecting how local sensory-motor circuits can coordinate gait without centralized planning.

We implemented Cruse-inspired rules:
- **Rule 1**: suppress lift-off (inhibitory)
- **Rule 2**: facilitate early protraction (excitatory)
- **Rule 3**: enforce late protraction (excitatory)
- **Rule 5**: distribute propulsive force (balance / load sharing)

Plus a **new rule** to help escape when a leg is trapped in a gap (triggered when contact force stays near zero for a long window).

### What made this non-trivial
- Extracting swing/stance from real data was not always reliable due to **noisy touch forces**. We used **z-position thresholds** where force traces were ambiguous.
- The whole behavior depended strongly on **rule weights**, and manual tuning is inherently approximate.

### Outcome (high level)
- **Flat**: forward motion is possible but gait is **irregular** and not well balanced between stance/swing phases.
- **Blocks**: stability is mostly preserved (force distributions stay near zero mean), but **progress is minimal**.
- **Gapped**: still prone to **getting stuck**, especially when **multiple legs** are trapped; the additional rule helps only in limited cases.

**Takeaway:** rules are a good baseline for coordination, but robustness across terrains likely requires either richer rules (spatial foot placement, recovery behaviors) or automatic tuning.

---

## Controller 2 — CPG-based (coupled oscillators)

<div class="row justify-content-center">
  <div class="col-sm-8 mx-auto mt-3 mt-md-0">
    <video autoplay loop muted playsinline
           class="img-fluid rounded z-depth-1 d-block mx-auto"
           style="height: 300px; object-fit: cover;"
           title="CPG-based controller on gapped terrain">
      <source src="{{ '/assets/video/cpg_gapped.mp4' | relative_url }}" type="video/mp4">
      Your browser does not support the video tag.
    </video>
  </div>
</div>
<div class="caption">
    CPG-based controller with coupled oscillators on gapped terrain.
</div>


### Core idea
We implemented a **CPG network with six oscillators (one per leg)**. The oscillators evolve through standard coupled-oscillator dynamics (phase + amplitude), and we map oscillator outputs to stepping trajectories.

### Closing the loop: force feedback
Because standard CPGs are open-loop, we introduced force feedback to handle rough terrain:

- **Frequency scaling by contact forces**  
  Slow down the leg carrying the most load; speed up the leg carrying the least; interpolate for the rest.

- **Freezing mechanism (thresholded force)**  
  If a leg’s end-effector force exceeds a threshold, freeze its oscillator state temporarily to prevent tipping.

- **Force term inside the phase dynamics**  
  Modify the oscillator phase derivative with a term based on force variation to automatically slow oscillations during destabilizing contacts.

### Gait design: tripod vs caterpillar
We evaluated multiple gaits and focused on:
- **Tripod gait**: fast and classic compromise between speed and stability.
- **Caterpillar gait**: wave-like pattern with **four legs on the ground most of the time**, which proved **more robust** on rough and gapped terrain.

<div class="row justify-content-center">
    <div class="col-sm-8 mx-auto mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/COBAR/Gaits.png" title="Tripod and Caterpillar gait diagrams" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Gait diagrams for the Tripod gait (up) and for the Caterpillar gait (down).
</div>

### Outcome (high level)
- **Flat**: CPG is very effective; caterpillar gait achieved **fast and stable** forward motion (and in our tests outperformed tripod on distance).
- **Blocks / Gapped**: performance drops substantially without feedback; with force feedback the fly can traverse, but robustness is not perfect (occasional tipping and failures when multiple legs are trapped).

**Takeaway:** CPGs are excellent for rhythmic locomotion, but realistic terrains strongly benefit from sensory feedback and/or additional high-level strategies.

---

## Controller 3 — Hybrid RL (PPO) with structured action space

<div class="row justify-content-center">
  <div class="col-sm-8 mx-auto mt-3 mt-md-0">
    <video autoplay loop muted playsinline
           class="img-fluid rounded z-depth-1 d-block mx-auto"
           style="height: 300px; object-fit: cover;"
           title="Hybrid controller on gapped terrain">
      <source src="{{ '/assets/video/hybrid_gapped.mp4' | relative_url }}" type="video/mp4">
      Your browser does not support the video tag.
    </video>
  </div>
</div>
<div class="caption">
    Hybrid RL-optimized controller combining structured patterns with learned adaptation on gapped terrain.
</div>

### Motivation: the capacity problem
A naïve RL policy trying to output **42 continuous joint angles** struggled: the action space was too high-dimensional for a simple policy to learn robustly within the project scope.

### Key design: match RL capacity to the task
We constrained RL to a **higher-level decision**:
- Observation space centered on **leg contact forces** (and, for blocks, also **pitch and roll**).
- Action predicts **which leg should step**, rather than predicting all joint angles directly.
- Joint trajectories still come from recorded stepping data (structured low-level actuation), while RL learns the **coordination policy**.

### Outcome (high level)
This hybrid approach was the most consistently effective across terrains:
- **Flat**: stable locomotion, strong forward progress; multi-objective rewards (speed + stability) traded off speed slightly without clear benefit on flat terrain.
- **Gapped**: strong performance; the policy learns reactive behaviors (e.g., thrust/jump-like recovery) that help escape gaps.
- **Blocks**: initially tipped over; adding **pitch/roll** to the observation improved robustness and enabled consistent traversal.

**Takeaway:** hand-engineered structure (low-level stepping primitives) + learning (high-level coordination) was a strong and practical compromise. It was robust without requiring an unrealistic action space.

---

## Results summary (qualitative)

- **Rule-based:** interpretable, biologically-inspired coordination; limited robustness on complex terrains without better tuning and recovery logic.
- **CPG-based:** strong rhythmic locomotion; best on flat terrain; sensory feedback and gait choice (caterpillar) matter a lot on rough terrain.
- **Hybrid RL:** best overall robustness; learns terrain-adaptive strategies while keeping the control space compact and learnable.

---

## Acknowledgements

Course: EPFL — Controlling behavior in animals and robots (Prof. P. Ramdya)  
Team: Alaa Aboud, Flore Munier-Jolain, Luca Zunino
