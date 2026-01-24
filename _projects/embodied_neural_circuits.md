---
layout: page
title: Embodied Neural Circuits in Zebrafish
description: How brain-body-environment integration shapes sensorimotor processing
img: assets/img/project_covers/zbot_above.jpg
importance: 1
category: research
---

<div class="row justify-content-center mt-1 mb-4">
    <div class="col-sm-2 col-4 text-center">
        <a href="https://www.science.org/doi/10.1126/scirobotics.adv4408" class="btn btn-sm btn-outline-primary" target="_blank">
            <i class="fa-solid fa-file-lines"></i> Article
        </a>
    </div>
    <div class="col-sm-2 col-4 text-center">
        <a href="https://www.science.org/toc/scirobotics/10/107" class="btn btn-sm btn-outline-primary" target="_blank">
            <i class="fas fa-book-open"></i> Cover
        </a>
    </div>
    <div class="col-sm-2 col-4 text-center">
        <a href="https://ponyo.epfl.ch/proj/zebrafish/simzfish" class="btn btn-sm btn-outline-primary" target="_blank">
            <i class="fas fa-code"></i> Code
        </a>
    </div>
</div>

## Journal Cover Feature & International Recognition

This research was featured on the **cover of Science Robotics** (October 2025, Volume 10, Issue 107) and received **international media coverage**, including articles in major Italian national newspapers.

> **Project Highlights**
> - **Publication:** Cover article in *Science Robotics* (Impact Factor ~25)
> - **My Role:** Led robotic control software development; conducted real-world river validation experiments
> - **Methods:** Neuromechanical simulation, two-photon calcium imaging, sim-to-real robot validation
> - **Industry Relevance:** Embodied AI, autonomous navigation, sim-to-real transfer, sensor fusion

**Media Coverage:**
- [ANSA: "From a robot with a fish brain, a new approach to AI" [in Italian]](https://www.ansa.it/canale_scienza/notizie/frontiere/2025/11/06/da-un-robot-con-il-cervello-di-pesce-un-nuovo-approccio-allia_20cc2205-5ddc-439d-a896-d2f4d6c8c5d9.html)
- [Repubblica: "A fish-robot reveals the secrets of the brain and movement" [in Italian]](https://www.repubblica.it/tecnologia/2025/10/30/news/zebrafisch_scienza_pesce-robot_svela_i_segreti_del_cervello_e_del_movimento-424948228/)
- [La Stampa: "A fish-robot reveals the secrets of the brain and movement" [in Italian]](https://www.lastampa.it/tecnologia/2025/10/30/news/zebrafisch_scienza_pesce-robot_svela_i_segreti_del_cervello_e_del_movimento-424948228/)
- [IVG: "A young researcher from Bergeggi among the authors of a study published in Science Robotics" [in Italian]](https://www.ivg.it/2025/11/un-giovane-ricercatore-di-bergeggi-tra-gli-autori-di-uno-studio-pubblicato-su-science-robotics/)
- [EPFL Press Release: "Roboticists reverse engineer zebrafish navigation"](https://actu.epfl.ch/news/roboticists-reverse-engineer-zebrafish-navigatio-2/)
- [Duke University Press Release: "Robotic fish unlocks secrets of the brain-body connection"](https://medschool.duke.edu/news/robotic-fish-unlocks-secrets-brain-body-connection)

---

## The Challenge: Understanding Brains Beyond Isolation

Brains don't evolve in a vacuum. They develop within bodies that move through physical environments, yet neuroscience has traditionally studied neural circuits in isolation. This approach misses something important: to truly understand how neural circuits work, we need to study them as part of a complete brain-body-environment system.

This project tackled that challenge by building a full neuromechanical simulation of the zebrafish **optomotor response**, a behavior where fish swim against visual motion to hold their position. Larval zebrafish are ideal for this because their transparent bodies let us observe neural activity directly. By recreating everything from body mechanics to water physics to the actual neural networks found in real fish, we could replicate real zebrafish behavior and test ideas that would be impossible to explore in biological experiments alone.

---

## The Approach: Iterating Between Simulation and Reality

The power of this work came from constantly moving between computational models, biological experiments, and physical robots. We developed **simZFish**, a neuromechanical simulation that captured:

- **Neural circuits** based on real zebrafish brain data
- **Body mechanics** with accurate physical structure and movement
- **Hydrodynamics** modeling how the fish body interacts with water
- **Visual processing** including lens properties and retinal wiring
- **Environmental context** from simple visual patterns to turbulent river flows

This simulation became a discovery platform. We could tweak aspects impossible to change in living animals (lens properties, retinal connections, virtual environments) and then check our predictions with real experiments. In parallel, we developed **ZBot**, a physical robotic platform that let us test the same principles in real-world conditions.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/project_covers/zbot_above.jpeg" title="ZBot in river environment" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/project_covers/simzfish.png" title="Neuromechanical simulation" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Left: The ZBot robotic platform tested in a real river with complex fluid dynamics. Right: The simZFish neuromechanical simulation in a virtual river environment.
    <small>[Sources - Left: Olivier Porchet, EPFL; Right: "Enhancing Robotic Behavior through Bio-Inspired Artificial Neural Networks: A Study of Optomotor Response and Rheotaxis in Robotic Fish", Luca Zunino]</small>
</div>

---

## Key Discoveries: How Embodiment Shapes Neural Function

The close link between simulation and experiment led to real discoveries about how embodiment shapes neural activity and behavior:

### 1. Why the Lower Posterior Visual Field Dominates

By systematically changing lens properties and retinal wiring in simulation, we figured out why the lower posterior visual field drives the strongest optomotor responses in zebrafish. The simulation predicted specific receptive field properties that were then confirmed in real fish. This explains a long-standing observation about how zebrafish process visual motion, and shows how the physical structure of the eye shapes what the brain computes.

### 2. Predicting New Neural Response Types

When we tested the simZFish with new visual stimuli, it predicted neural response patterns nobody had seen before. Working with experimentalists, we used **two-photon calcium imaging** to look for these responses in live zebrafish brains, and we found them. We then added these new response types back into the simulation, completing the loop between prediction and validation.

### 3. Autonomous Navigation in Complex Environments

We placed the simZFish in virtual rivers where it had to hold position against water flow, a behavior called rheotaxis. Without any explicit instructions for this task, the embodied sensorimotor circuits figured out how to use flow-induced visual patterns as navigation cues, compensating for the current on their own. This shows how complex navigation can emerge from the interaction between brain, body, and environment.

---

## Validation: Physical Robots in Real Rivers

Simulations are powerful, but they're still approximations. To really validate our findings, we built a physical robot called **ZBot** and tested it in a real river with all the messiness simulations can't fully capture: turbulent water, changing light, unpredictable noise.

The robot successfully held its position in the river using the same control principles we developed in simulation. This wasn't just a cool demo. It proved that the principles we discovered about embodied neural function actually work in the real world.

<div class="row justify-content-center">
    <div class="col-sm-12 mx-auto mt-3 mt-md-0">
        {% include video.liquid path="assets/video/ZBot_in_River.mp4" class="img-fluid rounded z-depth-1" controls=true autoplay=false %}
</div>
</div>
<div class="caption">
    ZBot holding its position in a real river with turbulent flow and natural lighting.
<small>[Source: Olivier Porchet, EPFL]</small>
</div>

---

## Impact: A New Paradigm for Integrative Neuroscience

This work shows a different way to study neural circuits, one that embraces brain-body-environment complexity rather than trying to eliminate it. By iterating between simulations, behavioral observations, neural imaging, and robot testing, we gained insights that no single approach could provide alone.

The framework goes beyond neuroscience. The same ideas (realistic simulation, experimental validation, bridging the sim-to-real gap) apply directly to challenges in **autonomous systems**, **embodied AI**, and **adaptive robotics**, where real-world performance depends on tight integration of sensing, computation, and physical dynamics.

---

## My Contributions

I worked at EPFL's BioRob Laboratory for over two years across two semester projects, my master's thesis, and an additional summer, contributing directly to this publication.

**Software Development**
- Developed the robotic control software for the ZBot platform
- Contributed to the simZFish neuromechanical simulation
- Built experimental frameworks for systematic testing and data collection

**Hardware & Experimental Work**
- Contributed to the construction of the ZBot robotic platform
- Conducted river validation experiments in real-world conditions

**Data Analysis & Validation**
- Analyzed behavioral data across simulation, robot, and biological experiments
- Validated simulation predictions against experimental results

---

## Acknowledgements

Source of the cover image: Olivier Porchet, EPFL