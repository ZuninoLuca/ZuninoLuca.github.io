---
layout: page
title: Neural Control Strategies for Energy-Efficient Swimming
description: A zebrafish-inspired robot to study the control and energetic trade-offs of intermittent swimming
img: assets/img/publication_preview/liu2026energy.png
importance: 2
category: research
---

<div class="row justify-content-center mt-1 mb-4">
    <div class="col-sm-2 col-4 text-center">
        <a href="https://www.science.org/doi/10.1126/scirobotics.adw7868" class="btn btn-sm btn-outline-primary" target="_blank">
            <i class="fa-solid fa-file-lines"></i> Article
        </a>
    </div>
</div>

This work was published in *Science Robotics* in January 2026 as **"Energy efficiency and neural control of continuous versus intermittent swimming in a fishlike robot"** (Liu, Longchamp, Zunino, et al.).

<div class="row justify-content-center">
    <div class="col-sm-8 col-md-6 mt-3 mt-md-0">
    {% include figure.liquid loading="eager" path="assets/img/projects/energy_efficiency_neural/ZBot.jpeg" title="ZBot platform" class="img-fluid rounded z-depth-1" %}
  </div>
</div>
<div class="caption">
  ZBot: a zebrafish-inspired robotic swimmer designed for controlled studies of intermittent locomotion. <small>[Source: Xiangxiao Liu, BioRob, EPFL]</small>
</div>

## Why Intermittent Swimming Matters

Many fish don’t swim continuously. Instead, they use **bout-and-glide** (also called *burst-and-coast*): short, active tail undulations (the **bout**) followed by a passive **glide**. This intermittent pattern appears across species and is often hypothesized to improve efficiency; however, the energetic advantage is still debated, partly because it is hard to **control** animal behavior precisely while measuring both kinematics and power.

This research study addresses that gap with a **bio-inspired robotic system**, enabling controlled experiments that disentangle the roles of:
- **Neural control** (how bouts start/stop and how gaits are modulated),
- **Body mechanics & actuation** (how thrust is produced),
- **Hydrodynamics** (how drag and momentum evolve across bout and glide).

---

## The Core Idea: A Robot That Embodies a Neural Hypothesis

Zebrafish larvae are a powerful model for locomotion neuroscience, but most zebrafish-inspired robots have historically focused on kinematics alone, without integrated sensing and explicit neural mechanisms.

We developed **ZBot**, a zebrafish-inspired robotic platform designed to **embody** a neural model of bout-and-glide swimming:
- The robot provides a physical substrate with realistic body dynamics and fluid interaction,
- The controller implements a mechanistic hypothesis of how bouts are initiated, shaped, and terminated,
- Experiments can systematically vary gait parameters while recording both **performance** and **energy consumption**.

<div class="row justify-content-center">
    <div class="col-sm-8 col-md-6 mt-3 mt-md-0">
    {% include figure.liquid loading="eager" path="assets/img/projects/energy_efficiency_neural/ZBot.jpeg" title="ZBot platform, frontal view" class="img-fluid rounded z-depth-1" %}
  </div>
</div>
<div class="caption">
  Frontal view of ZBot, the robotic platform we used in our swimming experiments. <small>[Source: Xiangxiao Liu, BioRob, EPFL]</small>
</div>

---

## ZBot: A Physical Model of a Larval Fish

ZBot is designed to reproduce the qualitative structure of larval swimming while remaining experimentally practical:

- **Articulated tail with multiple joints**  
  The tail is composed of **six hinge joints actuated by servomotors**, enabling undulatory wave generation and modulation.

- **Repeatable, measurable experiments**  
  The platform supports repeated trials under matched conditions, enabling parameter sweeps that are infeasible in animal studies.

- **Field-ready testing**  
  Beyond lab-style tank experiments, the system can be deployed in real environments (e.g., stationary water outdoors) for realistic hydrodynamic conditions.

<div class="row justify-content-center">
    <div class="col-sm-8 col-md-6 mt-3 mt-md-0">
    {% include figure.liquid loading="eager" path="assets/img/projects/bout_glide/field_setup.png" title="Experimental setup (Lake Geneva)" class="img-fluid rounded z-depth-1" %}
  </div>
</div>
<div class="caption">
  Outdoor experiments in stationary water (Lake Geneva harbor). <small>[Source: Xiangxiao Liu, BioRob, EPFL]</small>
</div>

---

## A Neural Model for Bout Initiation, Control, and Termination

A key scientific challenge is that intermittent swimming is not simply “continuous swimming with pauses”.
It requires a controller that can:

1) **Trigger a bout**,  
2) **Shape the bout** (frequency, amplitude, symmetry/asymmetry for turning), and  
3) **Stop the bout** and transition into a passive glide.

Biological work suggests a division of labor:
- **Spinal central pattern generators (CPGs)** generate rhythmic motor patterns,
- **Supraspinal gating mechanisms** regulate when swimming starts/stops.

Inspired by this, we propose a neural controller that can generate a *family* of gaits by manipulating parameters such as:
- Tail-beating **frequency** and **amplitude**,
- **Bout duration** and glide timing,
- Left/right asymmetries to induce **turning**.

---

## What We Measured: Kinematics *and* Energy

To move the debate on energetic efficiency from theory to evidence, we focus on two measurable outputs:

### 1) Behavior and kinematics
We compare robot trajectories and body kinematics against zebrafish-inspired targets:
- Forward bouts vs turning bouts,
- Evolution of head direction,
- Accumulated segment angles through the tail during each bout.

### 2) Power consumption and cost of transport
We quantify how much energy is required to travel a given distance under different strategies, comparing:
- **Intermittent** (bout-and-glide) swimming  
vs.
- **Continuous** tail-beating swimming

This allows a direct test of *when* intermittent swimming is beneficial.

---

## Key Findings

### Zebrafish-like intermittent gaits—generated by a mechanistic controller
By tuning neural-control parameters, ZBot produces **diverse bout-and-glide patterns** that qualitatively match zebrafish-like kinematics, including forward propulsion and controlled turning.

### Intermittent swimming can reduce energetic cost at low speed
By recording both velocity and energy consumption, we show that **bout-and-glide swimming achieves a lower cost of transport than continuous swimming at low velocities**—supporting the idea that intermittency can be advantageous under specific operating regimes.

---

## Why This Platform is Useful (Beyond This Paper)

ZBot turns a difficult biological question into an experimentally controllable one:
- It enables **parameter sweeps** (frequency, amplitude, duty cycle, bout timing) under matched conditions.
- It supports linking neural hypotheses to measurable outcomes: **speed, turning, stability, and energy**.
- It provides a bridge between neuroscience and robotics: testing how “neural design choices” shape embodied behavior.

---

## Acknowledgements

Source of the cover image: Xiangxiao Liu, BioRob, EPFL <br>
Complete author list: Xiangxiao Liu, François A. Longchamp, Luca Zunino, Louis Gevers, Lisa R. Schneider, Selina I. Bothner, André Guignard, Alessandro Crespi, Guillaume Bellegarda, Alexandre Bernardino, Eva A. Naumann, Auke J. Ijspeert.