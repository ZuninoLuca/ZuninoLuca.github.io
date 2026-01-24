---
layout: page
title: Poker table cards and chips recognition
description: Final project for the EPFL course "Image Analysis & Pattern Recognition"
img: assets/img/project_covers/IAPR_table.png
importance: 5
category: master's
---

<div class="row justify-content-center mt-1 mb-4">
    <div class="col-sm-2 col-4 text-center">
        <a href="{{ '/assets/pdf/Presentation.pdf' | relative_url }}" class="btn btn-sm btn-outline-primary" target="_blank">
            <i class="fa-solid fa-desktop"></i> Slides
        </a>
    </div>
    <div class="col-sm-2 col-4 text-center">
        <a href="https://github.com/ZuninoLuca/Image_analysis_and_pattern_recognition" class="btn btn-sm btn-outline-primary" target="_blank">
            <i class="fab fa-github"></i> Code
        </a>
    </div>
</div>

## At a glance

Final project for the EPFL course **“Image Analysis and Pattern Recognition”** (Prof. J.-P. Thiran), completed in **Spring 2022** with **Roberto Minini** and **Roberto Ceraolo**.

We implemented **an end-to-end image understanding task** on poker-table scenes. Starting from a single RGB image of a table, our system automatically identifies:
- The **five table cards**,
- The **two cards of each player** (or detects **face-down** / absent players),
- The **number of chips per color** in the pot.

<div class="row justify-content-center">
    <div class="col-sm-6 mx-auto mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/project_covers/IAPR_table.png" title="Poker-table scene to be analyzed" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Our system transforms a single poker-table snapshot into structured game state: every card identified, every chip counted.
</div>

---

## Approach overview

We implemented a robust **classical computer vision pipeline** that turns a complex image into structured predictions through staged geometry + detection + classification:

1. **Detect table corners** in the original image  
2. **Warp** the image to obtain a frontal view of the table  
3. **Split** the warped table into semantic regions (players / table cards / chips)  
4. **Detect + classify chips** (circle detection + color-based k-NN)  
5. **Recognize cards** (template matching with rotation search)  
6. **Return predictions** in the required dictionary format

A key design choice was to increase robustness by combining **redundant preprocessing hypotheses** (multiple thresholding methods + voting) with **simple but reliable classifiers** (k-NN on color features; template matching for symbols).

---

## 1) Table extraction (corner detection)

<div class="row justify-content-center">
    <div class="col-sm-12 mx-auto mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/IAPR/IAPR_1.png" title="Step 1 of our approach (table extraction)" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    The first step of our pipeline localizes the table corners to isolate the playing surface from the surrounding scene.
</div>

### Core idea
Accurate recognition becomes much easier once the table is isolated and rectified. We therefore detect the table as the dominant object and estimate its four corners.

### What we did
- Convert the image to **LAB**, using the **L channel** as grayscale.
- **Downscale** the image to a tractable resolution (e.g., 200×300) for faster processing.
- Compute **five different binary masks** using multiple thresholding methods (**Triangle, Otsu, Mean, Li, Isodata**) to handle lighting variability.
- Use **connected-component labeling** and select the most frequent non-background label (table assumed largest object).
- Clean masks via **erosion + area opening**, then compute a **convex hull** for each candidate table mask.
- Estimate corners using **Harris corner detection**.
  - If Harris fails (not exactly 4 corners), fall back to a **minimum bounding rectangle** on the convex hull.
- Combine the 5 corner candidates using a **majority vote** for robustness.

**Takeaway:** redundancy + voting makes the table detection reliable even when some thresholding methods fail.

---

## 2) Warping and region splitting

<div class="row">
  <div class="col-sm mt-3 mt-md-0 d-flex align-items-center justify-content-center">
    {% include figure.liquid loading="eager"
      path="assets/img/projects/IAPR/IAPR_2.png"
      title="Step 2 of our approach (warping and cutting of the image)"
      class="img-fluid rounded z-depth-1"
      style="height: 300px; object-fit: cover;" %}
  </div>
  <div class="col-sm mt-3 mt-md-0 d-flex align-items-center justify-content-center">
    {% include figure.liquid loading="eager"
      path="assets/img/projects/IAPR/IAPR_3.png"
      title="Step 3 of our approach (splitting the table in sub-regions)"
      class="img-fluid rounded z-depth-1"
      style="height: 300px; object-fit: cover;" %}
  </div>
</div>
<div class="caption">
  The second (left) and third (right) steps of our pipeline: the table image is warped and cut, then it is divided in sub-regions (4 player regions, table cards region, chips region).
</div>

### Warping
Given detected table corners and a canonical destination rectangle, we estimate a **projective transform** and warp the image to obtain a **frontal table view** (both grayscale and color versions). We then crop to keep only the table.

### Splitting into sub-regions
Because the warped table has consistent geometry, we use a **hard-coded split** (proportions of width/height) to extract:
- 4 player regions (`P1..P4`)
- The table cards region (`T`)
- The chips region (`C`)

We then split the table-cards region into **five individual card crops** (`T1..T5`), again using fixed proportional boundaries.

**Takeaway:** warping turns a hard detection problem into a stable “structured crop” problem.

---

## 3) Chips detection and classification

<div class="row justify-content-center">
    <div class="col-sm-10 mx-auto mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/IAPR/IAPR_4.png" title="Step 4 of our approach (detection and classification of chips)" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    The fourth step of our pipeline: the chips are detected and classified.
</div>

### Detection
Chips are detected using the **Hough circle transform**, tuned to be **sensitive to partially occluded circles** (accepting some false positives that we later filter).

We handle chips in two passes:
1. Detect **black/red/green/blue** chips using the L channel thresholding strategy to separate colored chips from the table.
2. Detect **white** chips separately (they blend with the table), using a different preprocessing path (LAB channel choice + masking) and another Hough pass.

### Classification (k-NN)
For each detected circle:
- Build a mask for the chip interior and compute average color features.
- Use a **k-NN classifier (k = 3, distance-weighted)** trained on manually labeled chip samples extracted from the training images.

Features (4D vector):
1. Mean **R** inside chip
2. Mean **G** inside chip
3. Mean **B** inside chip
4. Mean **L** over the whole chips sub-image (captures global lighting)

We also included “negative” examples (white chips / table patches) to reject false detections.

**Takeaway:** circle detection + simple color features + k-NN is surprisingly strong when paired with careful preprocessing and a lighting feature.

---

## 4) Card recognition (table + players)

<div class="row justify-content-center">
    <div class="col-sm-10 mx-auto mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/IAPR/IAPR_5.png" title="Step 5 of our approach (detection and classification of player and table cards)" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    The fifth step of our pipeline: the cards (for each player and for the table) are detected and classified.
</div>

### Single-card classification (table cards)
We implemented a `classify_card` routine based on **template matching**:
- Templates are extracted from provided setup images (`spades_suits`, `kings`, etc.) for:
  - Ranks: A, 2–10, J, Q, K (plus a **back-of-card** template for face-down cases)
  - Suits: D, H, S, C
- Matching metric: `cv.TM_CCOEFF_NORMED`
- For robustness to small rotations, we rotate each single-card crop from **−6° to +6°** (step 2°) and keep the best-confidence result.

### Double-card classification (player hands)
Player crops contain two cards with larger rotations and mutual interference. We therefore:
- Sweep rotations from **−35° to +35°** (step 4°).
- Use `classify_double_card` twice per rotation:
  1. Classify the best card symbol,
  2. **Mask** the matched symbol region and classify again to get the second card.
- Add sanity checks:
  - Swap rank/suit if the two cards get mixed,
  - Return results ordered as **leftmost card first**, matching the expected output format,
  - Return `0` for players not playing / face-down cards.

**Takeaway:** template matching is brittle in general, but becomes workable here thanks to controlled crops (warping) and explicit rotation search.

---

## Results (high level)

On the provided training set, the pipeline achieved strong performance overall, with most errors coming from difficult lighting conditions, partial occlusions, or challenging suit/rank visibility:

- **Chips:** high accuracy (most images in the strong-score range; robust counting even with partial occlusions)  
- **Card ranks:** very high accuracy (template matching is reliable for numbers/letters)  
- **Card suits:** slightly harder than ranks (smaller symbols + color/lighting variability), but still strong on average  

---

## Acknowledgements

Course: EPFL — Image Analysis and Pattern Recognition (Prof. J.-P. Thiran)  
Team: Roberto Ceraolo, Roberto Minini, Luca Zunino