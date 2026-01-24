---
layout: page
title: Thymio autonomous navigation stack
description: Final project for the EPFL course "Basics of Mobile Robotics"
img: assets/img/project_covers/thymio_cover.jpg
importance: 3
category: master's
---

<div class="row justify-content-center mt-1 mb-4">
    <div class="col-sm-2 col-4 text-center">
        <a href="https://github.com/ZuninoLuca/Basics_of_mobile_robotics/blob/main/Report.ipynb" class="btn btn-sm btn-outline-primary" target="_blank">
            <i class="fa-solid fa-file-lines"></i> Report
        </a>
    </div>
    <div class="col-sm-2 col-4 text-center">
        <a href="https://github.com/ZuninoLuca/Basics_of_mobile_robotics" class="btn btn-sm btn-outline-primary" target="_blank">
            <i class="fab fa-github"></i> Code
        </a>
    </div>
</div>

## At a glance

This is the final project for the EPFL course **“Basics of Mobile Robotics”** (Prof. F. Mondada), completed in **Fall 2021** in collaboration with **Nay Abi Akl, Mariam Hassan, and Alaa Abboud**.

The goal was to build a complete navigation pipeline for the **Thymio** mobile robot operating on a physical map observed by an overhead camera. A user defines a target position; the robot must autonomously **reach the goal while avoiding obstacles**, including **dynamic ones**.

Our test map recreates iconic EPFL buildings using LEGO® bricks, with Thymio playing the role of a student navigating campus to attend lectures. The robot and goal positions are detected using ArUco markers.

<div class="row">
  <div class="col-sm mt-3 mt-md-0 media-slot-300 d-flex align-items-center justify-content-center">
    {% include figure.liquid loading="eager"
      path="assets/img/projects/mobile_robotics/epfl_satellite.jpg"
      title="EPFL campus satellite view"
      class="img-fluid rounded z-depth-1" %}
  </div>

  <div class="col-sm mt-3 mt-md-0 media-slot-300 d-flex align-items-center justify-content-center">
    {% include figure.liquid loading="eager"
      path="assets/img/projects/mobile_robotics/thymio_map.jpg"
      title="Physical LEGO® map"
      class="img-fluid rounded z-depth-1" %}
  </div>
</div>
<div class="caption">
  Left: Satellite view of EPFL campus with highlighted buildings used as obstacles. Right: Our physical LEGO® map recreation for the Thymio robot.
</div>

---

## System in a nutshell

The project follows a practical “global + local” navigation structure:

1. **Perception & mapping**
   - Convert the camera view into a consistent **top-down map** (perspective warp).
   - Detect obstacles from the map image via **contours + vertices** (OpenCV).
   - Detect robot and goal using **ArUco markers**.

2. **Planning**
   - Compute a collision-free **global path** using **visibility graphs** and an **A\***-based search (adapted from *ExtremityPathFinder* and integrated into our pipeline).
   - Inflate obstacles to account for the robot footprint and tracking margins.

3. **State estimation**
   - Use an **Extended Kalman Filter (EKF)** with a differential-drive motion model and camera-based pose updates.

4. **Execution**
   - Convert the global path into a smooth, trackable trajectory (**discretization + Bezier smoothing**).
   - Track the trajectory with a **move-to-pose controller**, producing linear/angular velocity commands and converting them to Thymio motor control.
   - When something unexpected appears, switch to a **reactive obstacle avoidance** behavior using proximity sensors, then resume global path following.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/mobile_robotics/thymio_step1.png" title="Initial setup with optimal path" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/mobile_robotics/thymio_step2.png" title="Obstacle detected" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/mobile_robotics/thymio_step3.png" title="Avoiding obstacle" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/mobile_robotics/thymio_step4.png" title="Returning to optimal path" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    A complete navigation sequence: Thymio begins following the computed optimal path (left), detects a dynamic obstacle and switches to local avoidance mode (middle-left), navigates around the obstruction (middle-right), and successfully returns to its optimal trajectory toward the goal (right).
</div>

---

## What we implemented

### Global path planning (visibility graphs + A*)
We compute an *optimal* collision-free route by building a visibility graph from obstacle vertices (after inflation) and running **A\*** to connect start-to-goal. This planning style works well in structured environments because it produces clean, geometrically meaningful paths.

The pipeline includes:
- Obstacle contour extraction
- Polygon/vertex processing
- Obstacle inflation (safety margin)
- Graph construction + search
- Path extraction for downstream tracking

### Camera mapping: perspective warp to top view
A key practical element is the **camera geometry**. We use a perspective warp that transforms an oblique camera view into a top-down representation that is consistent for:
- Obstacle extraction
- Robot/goal detection
- Stable coordinate interpretation across the pipeline

The transform preserves aspect ratio and produces a top view suitable for planning and localization.

### Localization with an EKF
We implemented an **EKF** that fuses:
- A **differential-drive** motion model for prediction
- Camera-based pose observations for correction

This gives a smoother and more reliable pose estimate than raw detections alone, and helps stabilize control (especially when the camera signal becomes noisy or briefly unreliable).

### Trajectory preparation: discretization + smoothing
A shortest path is not always directly trackable by a real robot. We:
- **Discretize** the planned polyline into a sequence of points
- Smooth the resulting trajectory with a **Bezier curve**, reducing sharp turns and making it easier for the controller to follow

### Control: move-to-pose + motor commands
We used a move-to-pose style controller that computes linear and angular commands to track the planned trajectory. Those commands are then converted into motor actions suitable for Thymio.

### Local obstacle avoidance (static + dynamic)
Global planning alone is not enough in the presence of **unexpected changes**. We implemented reactive avoidance based on proximity sensors:
- Detect nearby obstacles
- Temporarily switch to a local avoidance behavior
- Return to the planned trajectory once safe

This enables handling **dynamic obstacles** (including another moving Thymio entering the path).

<div class="row justify-content-center">
    <div class="col-sm-8 mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/mobile_robotics/thymio_fsm.jpeg" title="Finite state machine" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    High-level finite state machine governing Thymio's navigation behavior, coordinating global path planning with local obstacle avoidance.
</div>

---

## Demo videos

These short demos show the robot operating on different maps and under different conditions:

1. Thymio computes the optimal path to reach the destination, and tracks it while avoiding added obstacles

   <div style="position:relative;padding-bottom:56.25%;height:0;overflow:hidden;border-radius:0.5rem; margin-bottom: 1.25rem;">
     <iframe
       src="https://www.youtube.com/embed/Htc53pRaKMg"
       title="Thymio computes the optimal path"
       style="position:absolute;top:0;left:0;width:100%;height:100%;border:0;"
       allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share"
       allowfullscreen
       loading="lazy">
     </iframe>
   </div>

2. Thymio can reliabily reach its destination even when static obstacles are added along its path

   <div style="position:relative;padding-bottom:56.25%;height:0;overflow:hidden;border-radius:0.5rem; margin-bottom: 1.25rem;">
     <iframe
       src="https://www.youtube.com/embed/Bnllt1kFHOY"
       title="Thymio reaches destination with static obstacles"
       style="position:absolute;top:0;left:0;width:100%;height:100%;border:0;"
       allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share"
       allowfullscreen
       loading="lazy">
     </iframe>
   </div>

3. Thymio can also react to dynamic obstacles, such as another Thymio crossing the planned path

   <div style="position:relative;padding-bottom:56.25%;height:0;overflow:hidden;border-radius:0.5rem; margin-bottom: 1.25rem;">
     <iframe
       src="https://www.youtube.com/embed/Du3KwuTA3x0"
       title="Thymio reacts to dynamic obstacles"
       style="position:absolute;top:0;left:0;width:100%;height:100%;border:0;"
       allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share"
       allowfullscreen
       loading="lazy">
     </iframe>
   </div>

4. Thymio can continue following the computed trajectory even when the camera gets temporarily obstructed

   <div style="position:relative;padding-bottom:56.25%;height:0;overflow:hidden;border-radius:0.5rem; margin-bottom: 1.25rem;">
     <iframe
       src="https://www.youtube.com/embed/J9pF1u8dQ0c"
       title="Thymio continues when camera obstructed"
       style="position:absolute;top:0;left:0;width:100%;height:100%;border:0;"
       allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share"
       allowfullscreen
       loading="lazy">
     </iframe>
   </div>

---

## Notes on robustness and design choices

A few decisions that mattered in practice:

- **Obstacle inflation:** even with a perfect plan, tracking error exists; inflation provides a safety margin for execution.
- **Smoothing:** the Bezier-based smoothing step makes the gap between “geometric shortest path” and “trackable motion” much smaller.
- **Estimator + controller separation:** the EKF reduces noise and makes the controller less reactive to spurious pose jitter.
- **Hybrid navigation:** combining global planning with reactive avoidance made the system more tolerant to dynamic changes and imperfect sensing.

---

## Acknowledgements

Course: EPFL — Basics of Mobile Robotics (Prof. F. Mondada)  
Team: Nay Abi Akl, Mariam Hassan, Alaa Abboud, Luca Zunino<br>
Source of the cover image: [OlivierMichel - Own work | CC BY-SA 4.0](https://commons.wikimedia.org/w/index.php?curid=69693699)