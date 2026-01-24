---
layout: page
title: Student Answer Forecasting
description: Transformer-driven prediction of students’ answer choices in a language-learning ITS
img: assets/img/publication_preview/gado2024student.jpg
importance: 4
category: research
---

<div class="row justify-content-center mt-1 mb-4">
    <div class="col-sm-2 col-4 text-center">
        <a href="https://arxiv.org/abs/2405.20079" class="btn btn-sm btn-outline-primary" target="_blank">
            <i class="ai ai-arxiv"></i> arXiv
        </a>
    </div>
    <div class="col-sm-2 col-4 text-center">
        <a href="https://educationaldatamining.org/edm2024/proceedings/2024.EDM-posters.66/2024.EDM-posters.66.pdf" class="btn btn-sm btn-outline-primary" target="_blank">
            <i class="fas fa-file-pdf"></i> PDF
        </a>
    </div>
    <div class="col-sm-2 col-4 text-center">
        <a href="https://github.com/epfl-ml4ed/answer-forecasting" class="btn btn-sm btn-outline-primary" target="_blank">
            <i class="fab fa-github"></i> Code
        </a>
    </div>
    <div class="col-sm-2 col-4 text-center">
        <a href="https://huggingface.co/collections/epfl-ml4ed/student-answer-forecasting-edm-2024" class="btn btn-sm btn-outline-primary" target="_blank">
            <i class="fas fa-database"></i> Models
        </a>
    </div>
</div>

## Beyond “Correct or Incorrect”: Forecasting *Which* Answer a Student Will Choose

Most student models in Intelligent Tutoring Systems (ITS) aim to predict whether a learner will answer a question correctly. This is useful, but it completely disregards *why* the student might be struggling. In a multiple-choice question (MCQ), the *specific option* a student selects often reveals a misconception, a confusable grammar rule, or a systematic misunderstanding.

This research project introduces **student answer forecasting**: instead of predicting correctness, we predict the **likelihood of choosing each answer option**. That shift unlocks two practical benefits:

- **Richer diagnostics**: recurring selection of the same distractor can signal a stable misconception.
- **Modular answer choices**: forecasting is performed per (question, option) pair, so educators can **add/remove/rewrite options** without needing to retrain a monolithic multi-class model.

---

## A Four-Stage Pipeline: From Raw ITS Logs to Answer-Choice Forecasting

At the core of the work is **MCQStudentBert**, a transformer-based pipeline that combines:

1) the **text** of questions and answer choices, and  
2) a compact **student embedding** distilled from the student’s interaction history.

<div class="row">
  <div class="col-sm mt-3 mt-md-0">
    {% include figure.liquid loading="eager" path="assets/img/projects/answer_forecasting/pipeline.png" title="Student Answer Forecasting pipeline" class="img-fluid rounded z-depth-1" %}
  </div>
</div>
<div class="caption">
  Overview of the pipeline: data processing → student embedding generation → (domain adaptation + correct-answer learning) → student answer forecasting → evaluation.
</div>

### Stage 1 — Data processing (turning interactions into learning signals)
We start from raw ITS logs and reshape them into a format suitable for transformers and sequential models:
- Each student has a time-ordered history of MCQ interactions.
- Each MCQ is expanded into **(question, candidate answer choice)** instances.
- Labels indicate whether the student selected that choice (supporting multi-response settings when applicable).

This “expand-into-binary-instances” view is what later makes answer choices modular: each option becomes its own prediction target.

### Stage 2 — Student embeddings (compressing history into a vector)
A central question is: *what is the best way to represent a student’s past interactions?*  
We compared four families of embedding strategies:

- **MLP autoencoder (feature-based)**  
  Uses engineered mastery-style features and learns a compressed representation.

- **LSTM autoencoder (sequence-based)**  
  Encodes temporal dynamics from ordered interaction sequences.

- **LernnaviBERT (domain-adapted German BERT)**  
  Fine-tunes a German BERT model on the ITS domain to better embed question–answer text.

- **Mistral 7B Instruct (LLM-based embeddings)**  
  Builds embeddings from recent interaction text with longer-context capacity; mean pooling over hidden states yields a compact student representation.

---

## Why Pretrain on Correct Answers First?

Before forecasting *students’* choices, we first train a model (**MCQBert**) to predict the **correct answer(s)** from text alone. This step matters because it separates two failure modes:

- The model doesn’t understand the MCQ content (poor “correct answer” competence), vs.
- The model understands the content, but needs student context to predict *which distractor* a given learner will pick.

Only after learning correct-answer structure, we inject student embeddings and fine-tune for answer-choice forecasting.

---

## MCQStudentBert: Fusing Student History with Question Understanding

We explore two integration strategies that inject student context into a BERT-based answer predictor:

### 1) Concatenation at the decision stage (MCQStudentBertCat)
Student embedding and MCQ representation are kept distinct until the end:
- BERT encodes the (question, option) text into a [CLS] representation
- The student embedding is projected to the right dimensionality
- **Concatenate → classify**

This approach preserves the base language processing and lets the classifier “reason” jointly over both sources of information.

### 2) Addition at the input stage (MCQStudentBertSum)
Student embedding is projected and **added** into the input embedding space before transformer encoding:
- Student context acts like a “bias” shaping how the MCQ text is interpreted
- Useful as an early fusion baseline inspired by multimodal embedding fusion

<div class="row justify-content-center">
  <div class="col-sm-8 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/projects/answer_forecasting/architecture_cat_sum.png" title="MCQStudentBert fusion strategies" class="img-fluid rounded z-depth-1" %}
  </div>
</div>
<div class="caption">
  Two fusion strategies: (A) summation at the input embedding level (Sum) vs. (B) concatenation before classification (Cat).
</div>

---

## Real-World Evaluation on a Large-Scale German ITS Dataset

We evaluate on language-learning MCQs from **Lernnavi**, a real-world ITS used at scale:
- **10,499 students**
- **138,149 interactions (transactions)**
- **237 unique MCQs**

Beyond accuracy, we emphasize metrics robust to class imbalance:
- **Matthews Correlation Coefficient (MCC)**
- **F1 score**
- **Accuracy**

---

## Results: Student Context Helps, and LLM Embeddings Work Best

Across embeddings and fusion strategies, injecting student context consistently improves answer forecasting over:
- A dummy majority baseline
- A text-only model that ignores student history.

A representative summary of our trained models:

<table style="border-collapse:collapse; width:100%; border-top:1px solid #000; border-bottom:1px solid #000; margin-bottom:1.5rem;">
  <thead>
    <tr>
      <th rowspan="2" style="padding:6px 10px;"></th>
      <th rowspan="2" style="padding:6px 10px;"></th>
      <th colspan="6" style="padding:6px 10px; text-align:center; border-bottom:1px solid #000;">
        Embedding
      </th>
    </tr>
    <tr>
      <th style="padding:6px 10px; text-align:center;">Dummy</th>
      <th style="padding:6px 10px; text-align:center;">MCQBert</th>
      <th style="padding:6px 10px; text-align:center;">MLP</th>
      <th style="padding:6px 10px; text-align:center;">LSTM</th>
      <th style="padding:6px 10px; text-align:center;">LernnaviBERT</th>
      <th style="padding:6px 10px; text-align:center;">Mistral 7B</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td rowspan="3" style="padding:6px 10px; font-weight:700; border-top:1px solid #000;">MCQStudentBertCat</td>
      <td style="padding:6px 10px; border-top:1px solid #000;">MCC</td>
      <td style="padding:6px 10px; text-align:right; border-top:1px solid #000;">0</td>
      <td style="padding:6px 10px; text-align:right; border-top:1px solid #000;">0.518</td>
      <td style="padding:6px 10px; text-align:right; border-top:1px solid #000;">0.557</td>
      <td style="padding:6px 10px; text-align:right; border-top:1px solid #000;">0.567</td>
      <td style="padding:6px 10px; text-align:right; border-top:1px solid #000;">0.575</td>
      <td style="padding:6px 10px; text-align:right; border-top:1px solid #000;"><b>0.579</b></td>
    </tr>
    <tr>
      <td style="padding:6px 10px;">F1 Score</td>
      <td style="padding:6px 10px; text-align:right;">0.305</td>
      <td style="padding:6px 10px; text-align:right;">0.740</td>
      <td style="padding:6px 10px; text-align:right;">0.772</td>
      <td style="padding:6px 10px; text-align:right;">0.777</td>
      <td style="padding:6px 10px; text-align:right;">0.780</td>
      <td style="padding:6px 10px; text-align:right;"><b>0.782</b></td>
    </tr>
    <tr>
      <td style="padding:6px 10px;">Accuracy</td>
      <td style="padding:6px 10px; text-align:right;">0.590</td>
      <td style="padding:6px 10px; text-align:right;">0.771</td>
      <td style="padding:6px 10px; text-align:right;">0.785</td>
      <td style="padding:6px 10px; text-align:right;">0.790</td>
      <td style="padding:6px 10px; text-align:right;">0.795</td>
      <td style="padding:6px 10px; text-align:right;"><b>0.797</b></td>
    </tr>
    <tr>
      <td rowspan="3" style="padding:6px 10px; font-weight:700; border-top:1px solid #000;">MCQStudentBertSum</td>
      <td style="padding:6px 10px; border-top:1px solid #000;">MCC</td>
      <td style="padding:6px 10px; text-align:right; border-top:1px solid #000;">0</td>
      <td style="padding:6px 10px; text-align:right; border-top:1px solid #000;">0.518</td>
      <td style="padding:6px 10px; text-align:right; border-top:1px solid #000;">0.552</td>
      <td style="padding:6px 10px; text-align:right; border-top:1px solid #000;">0.564</td>
      <td style="padding:6px 10px; text-align:right; border-top:1px solid #000;">0.568</td>
      <td style="padding:6px 10px; text-align:right; border-top:1px solid #000;"><b>0.569</b></td>
    </tr>
    <tr>
      <td style="padding:6px 10px;">F1 Score</td>
      <td style="padding:6px 10px; text-align:right;">0.305</td>
      <td style="padding:6px 10px; text-align:right;">0.740</td>
      <td style="padding:6px 10px; text-align:right;">0.767</td>
      <td style="padding:6px 10px; text-align:right;">0.774</td>
      <td style="padding:6px 10px; text-align:right;">0.777</td>
      <td style="padding:6px 10px; text-align:right;"><b>0.778</b></td>
    </tr>
    <tr>
      <td style="padding:6px 10px;">Accuracy</td>
      <td style="padding:6px 10px; text-align:right;">0.590</td>
      <td style="padding:6px 10px; text-align:right;">0.771</td>
      <td style="padding:6px 10px; text-align:right;">0.785</td>
      <td style="padding:6px 10px; text-align:right;"><b>0.790</b></td>
      <td style="padding:6px 10px; text-align:right;">0.789</td>
      <td style="padding:6px 10px; text-align:right;">0.789</td>
    </tr>
  </tbody>
</table>



Two practical takeaways:
- **Concatenation** is slightly more reliable than summation across embeddings.
- **Mistral 7B embeddings** perform best overall, suggesting that longer-context LLM representations capture meaningful structure in students’ interaction histories.

---

## What the Embedding Space Reveals: Learning as a Trajectory

To make the student representations more interpretable, we visualize embeddings over time using t-SNE. One striking pattern is that a single student’s embedding often evolves smoothly as they answer more questions, suggesting that the representation captures a moving “state” rather than a static profile.

<div class="row justify-content-center">
  <div class="col-sm-8 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/projects/answer_forecasting/tsne_trajectory.png" title="Student embedding trajectory (t-SNE)" class="img-fluid rounded z-depth-1" %}
  </div>
</div>
<div class="caption">
  A student’s embedding trajectory over time (lighter → earlier, darker → later), illustrating how representations shift as interaction history grows.
</div>

---

## Why This Matters for Intelligent Tutoring Systems

Answer-choice forecasting enables interventions that correctness-only prediction can’t support:

- **Misconception-aware hints**: target the *specific distractor* a student is likely to pick.
- **Dynamic distractors**: personalize which options are shown to increase diagnostic value.
- **Safe content updates**: add a new option (e.g., a new misconception) without retraining a multi-class model.
- **Granular analytics**: track how distractor likelihood changes after an explanation or practice session.

---

## Acknowledgements

Source of all images and table: "Student Answer Forecasting: Transformer-Driven Answer Choice Prediction for Language Learning", E.G. Gado et al.