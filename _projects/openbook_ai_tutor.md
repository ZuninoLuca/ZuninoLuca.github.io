---
layout: page
title: Open-book AI tutor for EPFL course Q&A
description: Final project for the EPFL course "Modern Natural Language Processing"
img: assets/img/project_covers/t5.jpg
importance: 7
category: master's
---

<div class="row justify-content-center mt-1 mb-4">
    <div class="col-sm-2 col-4 text-center">
        <a href="{{ '/assets/pdf/final_report_ModernNLM.pdf' | relative_url }}" class="btn btn-sm btn-outline-primary" target="_blank">
            <i class="fas fa-file-pdf"></i> Report
        </a>
    </div>
    <div class="col-sm-2 col-4 text-center">
        <a href="https://github.com/ZuninoLuca/Modern_natural_language_processing" class="btn btn-sm btn-outline-primary" target="_blank">
            <i class="fab fa-github"></i> Code
        </a>
    </div>
</div>

## At a glance
Final project for the EPFL course **“Modern Natural Language Processing”** (Prof. A. Bosselut), completed in **Spring 2023** with **Nay Abi Akl** and **Mariam Hassan**.  
We built an **academic question-answering chatbot** designed to help students with **university-level course questions**, with a focus on **answer quality, factuality, and robustness** across different domains and languages (English + French).

A central challenge is that course questions often require **domain knowledge** that may not be reliably stored in a compact model’s parameters. To address this, we explored an **open-book** design: the chatbot retrieves **external context from Wikipedia** at inference time, then generates an answer conditioned on that context.

---

## Goal
Develop an AI tutor that can answer academic questions by combining:
1. **A generative QA model** (T5-family) fine-tuned for question answering.
2. **An open-book retrieval mechanism** that fetches relevant Wikipedia summaries in real time.
3. **A learned reward model** that scores answers, enabling quantitative evaluation and experimentation with **RLHF**.

<div class="row justify-content-center">
    <div class="col-sm-12 mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/MNLP/t5.jpg" title="T5 model scheme" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Scheme of the T5 model we considered as starting point for our AI tutor.
    <small>[Source: "Exploring the Limits of Transfer Learning with a Unified Text-to-Text Transformer", C. Raffel et al.]</small>
</div>

---

## System in a nutshell
Our pipeline follows a “retrieve → condition → answer” structure:

1. **Keyword & language prediction (Retriever model)**  
   A fine-tuned **T5 keyword generator** predicts a Wikipedia page title (keyword) and the question language (**EN/FR**).

2. **Context retrieval (Wikipedia)**  
   We query Wikipedia APIs using beam-search candidates and retrieve a **page summary** (skipping disambiguation pages).

3. **Answer generation (QA model)**  
   The chatbot takes **(question + retrieved context)** and generates a concise, tutor-style answer.

4. **Answer scoring (Reward model)**  
   A fine-tuned **XLM-RoBERTa reward model** scores outputs on a **0–5** scale to compare models and settings.

---

## What we implemented

### Open-book context retrieval
We implemented a Wikipedia-based retrieval module to provide **fresh, topic-relevant background** for questions that may be out-of-distribution for the base model.

Key design choices:
- **Wikipedia as source**: free APIs, high-quality curated content, and practical licensing/usage considerations.
- **Summary as context**: a lightweight but effective approximation under project constraints.
- **Robust retrieval**: generate multiple keyword candidates (beam search) and select the first valid, non-disambiguation page.

### Keyword retriever model (T5)
To make retrieval automatic, we fine-tuned a T5 model to map:
- Input: `Keyword and Language of: {question}`
- Output: `{keyword}|{lang}`

This turns retrieval into a learned step and supports **English + French** questions. We found T5 to be more reliable than GPT-2 for keyword generation in this setup.

### QA model fine-tuning (GPT-2 vs T5 vs Flan-T5)
We trained and compared:
- **GPT-2** (decoder-only; next-token training on concatenated inputs)
- **T5** (text-to-text; question+context → answer)
- **Flan-T5** (instruction-tuned; same interface as T5)

Empirically, **T5-family models produced more consistent, concise, and relevant answers**, especially when conditioned on retrieved context. GPT-2 often generated fluent but **overly generic** or **drifty** continuations.

### Reward model (XLM-RoBERTa)
We trained a reward model to score answer quality (0–5), enabling:
- Model comparison beyond pure text overlap metrics
- Evaluation across heterogeneous datasets and formats
- A foundation for RLHF experiments

Implementation highlights:
- Base: **xlm-roberta-base**, with optional **domain-adaptation** via MLM finetuning on course-style data
- Regression head over **CLS** embedding
- MSE loss for stable training; explored pairwise loss but found it unreliable under tight compute constraints

### RLHF exploration
We integrated the reward model into an RL environment and ran **PPO-based RLHF** experiments. In our limited compute setting, RLHF did **not** reliably improve generations; we observed sensitivity to reward quality and hyperparameters.

---

## Datasets and evaluation
We combined course-specific and external datasets to diversify question styles and difficulty:

- **Course dataset (CS-552)**: EPFL question/answer logs (noisy; required heavy preprocessing)
- **Texas University short-answer grading dataset** (expert-scored 0–5)
- **ELI5**: created graded answer variants (5 → 0) using ChatGPT-based augmentation
- **Pairwise preference datasets**: “synthetic-instruct-gptj-pairwise” and “hh-rlhf” (with detoxification filtering for safety)

Evaluation combined:
- **Automatic metrics** where appropriate (e.g., classification-style accuracy for chosen/rejected)
- **Semantic metrics** (e.g., BERTScore for open QA comparisons)
- **Reward-model scoring** to compare average answer quality across models
- Extensive **qualitative analysis** to sanity-check behavior and failure modes

---

## Results summary (high level)
- **Open-book (retrieval-augmented) answering** improved factual grounding when the retrieved context was relevant.
- **T5 and Flan-T5** produced the best overall answers (concise, coherent, and tutor-like), substantially outperforming GPT-2 in this setting.
- **Reward modeling** enabled scalable comparison across models and datasets; performance improved as training data diversity increased.
- **RLHF** was promising in principle but unstable in practice under limited resources, highlighting dependence on reward quality and tuning.

---

## Ethical considerations
We explicitly tested unsafe or biased prompts using the live QA script and observed that the system remained largely **non-discriminatory** in our trials, helped by:
- Wikipedia-based context (generally factual and monitored)
- Dataset curation and **toxicity filtering** (e.g., Detoxify filtering on hh-rlhf samples)

We also documented cases where a model should **refuse** to provide harmful content (e.g., cybersecurity attack code) and treated refusal behavior as desirable.

---

## Acknowledgements
Course: EPFL — Modern Natural Language Processing (Prof. A. Bosselut)  
Team: Nay Abi Akl, Mariam Hassan, Luca Zunino <br>
Source of the cover image: ["Exploring the Limits of Transfer Learning with a Unified Text-to-Text Transformer", C. Raffel et al., 2020, JMLR | CC 4.0](https://jmlr.org/papers/volume21/20-074/20-074.pdf)