---
layout: page
title: Evolving a locomoting robot
description: Final project for the EPFL course "Evolutionary Robotics"
img: assets/img/project_covers/evolrob.png
importance: 6
category: master's
---

<div class="row justify-content-center mt-1 mb-4">
    <div class="col-sm-2 col-4 text-center">
        <a href="{{ '/assets/pdf/Grand_Challenge_Slides.pdf' | relative_url }}" class="btn btn-sm btn-outline-primary" target="_blank">
            <i class="fa-solid fa-desktop"></i> Slides
        </a>
    </div>
    <div class="col-sm-2 col-4 text-center">
        <a href="https://github.com/ZuninoLuca/Evolutionary_robotics" class="btn btn-sm btn-outline-primary" target="_blank">
            <i class="fab fa-github"></i> Code
        </a>
    </div>
</div>

## At a glance

This is the final project for the EPFL course **“Evolutionary Robotics”** (Prof. D. Floreano), completed in **Spring 2022** with **Mariam Hassan, Hadrien Sprumont, and Léo Duggan**.

The goal was to use **evolutionary algorithms** to obtain a robot that can:
- **Locomote efficiently** in a challenging arena,
- **Avoid obstacles**, and
- Remain **as stable as possible** (low roll/pitch/acceleration spikes, robust motion rather than brittle “tricks”).

To assess the **simulation-to-reality gap**, the final robot was **3D-printed** and tested in the real world.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        <video autoplay loop muted playsinline class="img-fluid rounded z-depth-1" style="height: 300px; object-fit: cover;" title="Evolved robot in simulation">
            <source src="{{ '/assets/img/projects/evolutionary_robotics/evolved_sim.mp4' | relative_url }}" type="video/mp4">
            Your browser does not support the video tag.
        </video>
    </div>
    <div class="col-sm mt-3 mt-md-0">
        <video autoplay loop muted playsinline class="img-fluid rounded z-depth-1" style="height: 300px; object-fit: cover;" title="3D-printed robot in real world">
            <source src="{{ '/assets/img/projects/evolutionary_robotics/evolved_real.mp4' | relative_url }}" type="video/mp4">
            Your browser does not support the video tag.
        </video>
    </div>
</div>
<div class="caption">
    From simulation to reality: the evolved robot navigating obstacles in Webots (left) and its 3D-printed counterpart tested with a physical obstacle (right), demonstrating successful transfer across the simulation-to-reality gap.
</div>

---

## Motivation

We framed the task in the spirit of a **Mars exploration rover**: locomotion in imperfect environments, stability of the core module, and obstacle-aware behavior. Furthermore, we aimed to keep the process **scientific and repeatable**.

---

## System in a nutshell

Our work followed an iterative pipeline that progressively increased task difficulty and realism:

**Exploration of parameters → Starting morphologies → Locomotion optimization → Stability optimization → Obstacle avoidance evolution**

Concretely, the project flow was:

1. **Exploration of Robogen parameters** and baseline setups  
2. **Starting robots** (to trade off exploration vs repeatability)
   - **Random configurations** (more exploration, surprising morphologies)
   - **Rolling snake** (oscillator-only control, simple geometry, good racing score)
   - **Starfish** (oscillator-only control, richer geometry, good racing score)
3. **Locomotion (racing) in an arena**
   - Start with simple objectives and progressively refine fitness
4. **Stability** as an explicit optimization target
5. **Obstacle avoidance**, explored via two main tracks:
   - **Brain-only OA** (manual sensor placement, evolve controller)
   - **Body + brain OA** with implicit constraints (co-evolve morphology and control)

<div class="row justify-content-center">
    <div class="col-sm-12 mx-auto mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/evolutionary_robotics/Flowchart.png" title="The flowchart of the project" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    The flowchart of the project.
</div>

---

## Experimental setup

### Evolution strategy
We used a classical evolutionary setup (µ/λ evolution with tournament selection), running **200 generations** with:
- **µ = 25**, **λ = 100**
- **(µ+λ) replacement**
- **Tournament size = 3**
- Controller mutation probability **pBrainMutate = 0.5**
- Brain mutation scale **BrainSigma = 0.9**

Simulation settings included sensor/motor noise (0.03), a simulation horizon of ~12s, and a bounded actuation update rate (16 direction shifts/s), encouraging solutions that remain functional under noise.

---

## Fitness design (what we optimized)

A key learning was that **fitness design dominates outcomes**: evolution is opportunistic and will exploit loopholes if they exist.

### Locomotion (racing)
We tested multiple progressively refined fitness functions, including:
- **Racing distance** (final distance of the closest element)
- **Core displacement integrals** (encouraging sustained progress, not just end position)
- **Final core position + stability terms**
- **Minimum distance constraints + stability**

### Stability
Stability was introduced explicitly with terms related to **roll/pitch** and **core acceleration**, pushing the evolved gaits toward smoother, more controllable motion.

### Obstacle avoidance (OA)
We explored OA in two ways:

**1) Brain-only OA + manual sensor placement**
- Keep the body fixed, place sensors deliberately,
- Evolve the controller to use sensory feedback for OA.

**2) Body + brain OA (implicit OA)**
- Co-evolve morphology and controller,
- Enforce constraints such as a minimum number of sensors and preventing “cheating” behaviors (e.g., obstacle despawning, lucky initializations, or exploiting arena artifacts).

This stage required repeated adjustments because evolution will reliably discover unintended exploits unless the environment and constraints are carefully designed.

---

## Key outcomes

### Emergent morphologies and controllers
We observed distinct locomotion “families” emerging from different initial conditions:
- **Rolling-snake** solutions: simple, effective, highly regular oscillatory motion.
- **Starfish-like** solutions: more complex geometry enabling richer gait patterns.
- A repeated high-performing morphology we nicknamed **“GorillaBot”**, which appeared multiple times under non-random initializations, illustrating the trade-off between **repeatability** and **novel exploration**.

<div class="row">
  <div class="col-sm mt-3 mt-md-0 d-flex align-items-center justify-content-center">
    {% include figure.liquid loading="eager"
      path="assets/img/project_covers/evolrob.png"
      title="Starfish-like robot in simulation"
      class="img-fluid rounded z-depth-1" %}
  </div>

  <div class="col-sm mt-3 mt-md-0 d-flex align-items-center justify-content-center">
    {% include figure.liquid loading="eager"
      path="assets/img/projects/evolutionary_robotics/snake.png"
      title="Snake-like robot in simulation"
      class="img-fluid rounded z-depth-1" %}
  </div>

  <div class="col-sm mt-3 mt-md-0 d-flex align-items-center justify-content-center">
    {% include figure.liquid loading="eager"
      path="assets/img/projects/evolutionary_robotics/Sim.png"
      title="GorillaBot in simulation"
      class="img-fluid rounded z-depth-1" %}
  </div>
</div>
<div class="caption">
  An example of starfish-like (left) and rolling-snake (center) solutions, and our GorillaBot (right) in simulation.
</div>

### Sim-to-real validation
The final evolved design was **3D-printed** and tested physically to compare behavior with simulation, giving direct insight into:
- Which behaviors transfer robustly,
- And which are sensitive to real-world friction, compliance, and actuator imperfections.

<div class="row justify-content-center">
    <div class="col-sm-10 mx-auto mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/evolutionary_robotics/Real.png" title="3D-printed GorillaBot" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Our 3D-printed GorillaBot.
</div>

---

## Main takeaways

The main takeaways of this project were:
- **Simpler fitness functions worked better** (and were harder to exploit).
- **Random starts** produced more diverse and surprising robots, but with larger variability.
- **Non-random starts** improved repeatability (e.g., GorillaBot emerged multiple times).
- Evolution is inherently **opportunistic**: careful arena design + constraints are essential.

---

## Acknowledgements

Course: EPFL — Evolutionary Robotics (Prof. D. Floreano)  
Team: Mariam Hassan, Hadrien Sprumont, Léo Duggan, Luca Zunino