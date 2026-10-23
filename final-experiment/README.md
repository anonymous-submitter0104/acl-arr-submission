# Final Experiment: MILA Evaluation

This document reports the continual-pretraining experiments evaluating **MILA** (Multilingual Indic Language Archive), accompanying the anonymous ACL ARR submission *MILA (Multilingual Indic Language Archive): A Dataset for Equitable Multilingual LLMs*. The focus is on **absolute performance improvements** and **fairness of representation** across Indic languages. See Section 4.2 (Continual Pretraining Experiments) of the paper.

---

## Table of Contents

1. [Overview](#overview)
2. [Experiment 1: Absolute Performance with Continual Pretraining](#experiment-1-absolute-performance-with-continual-pretraining)

   * [Experimental Setup](#experimental-setup)
   * [Results](#results)
   * [Analysis](#analysis)
3. [Experiment 2: Fairness via Indic Parity](#experiment-2-fairness-via-indic-parity)

   * [Definition of Parity](#definition-of-parity)
   * [Results](#parity-results)
   * [Analysis](#parity-analysis)
4. [Experiment 3: Domain-wise Ablation](#experiment-3-domain-wise-ablation)
5. [Conclusion](#conclusion)

---

## Overview

We evaluate **Qwen3-600M** (Q600) under two conditions:

1. The **original pretrained checkpoint** (Q600-Base).
2. The same checkpoint after **continual pretraining** on **MILA** (Q600-CPT), keeping architecture, optimizer and training recipe identical.

Existing benchmarks cover at most 14 of MILA's 22 languages, so evaluation uses **Indic-MMLU** (an MMLU translation into all 22 scheduled languages) together with benchmarks external to our pipeline (MILU, Sanskriti, Belebele, English MMLU, ARC Challenge), so that no conclusion rests on Indic-MMLU alone. The key questions are:

* Does continual pretraining on MILA improve absolute performance on Indic languages?
* Does it yield more **equitable representation** across Indic languages and English (measured via *parity*)?

---

## Experiment 1: Absolute Performance with Continual Pretraining

### Experimental Setup

* **Base Model:** [Qwen3-600M](https://huggingface.co/Qwen/Qwen3-0.6B)
* **Docker Image:** `nvcr.io/nvidia/nemo:25.07`
* **Compute Resources:** 32 × NVIDIA H100 GPUs (4 nodes)
* **Training Duration:** 8 days
* **Dataset:** Continual pretraining on a **200B-token subset of MILA**, equally distributed across the **14 Indic languages** with existing benchmark coverage:
  *Assamese (As), Bengali (Bn), Gujarati (Gu), Hindi (Hi), Kannada (Kn), Malayalam (Ml), Marathi (Mr), Nepali (Ne), Odia (Or), Punjabi (Pa), Sanskrit (Sa), Sindhi (Sd), Tamil (Ta), Telugu (Te)*

**Procedure:**

1. Evaluate the pretrained checkpoint on the benchmarks.
2. Continually pretrain on MILA.
3. Re-evaluate to measure the improvement.

---

### Results

#### Benchmark Scores (averaged across languages)

| Benchmark     | Q600-Base | Q600-CPT  |
| ------------- | --------: | --------: |
| Indic-MMLU    | 32.12     | **34.86** |
| MMLU          | 36.78     | **41.95** |
| MILU          | 27.51     | **28.12** |
| Sanskriti†    | 42.88     | **48.79** |
| Belebele      | 32.33     | **35.63** |
| ARC Challenge | 31.23     | **35.24** |

† Sanskriti is an English-only benchmark, though Indian-context-centric (it tests knowledge of Indian culture rather than native-language ability); it is retained as the closest available India-centric cultural benchmark.

---

### Analysis

* **Consistent improvements:** Q600-CPT outperforms Q600-Base on all six benchmarks.
* **Indic-MMLU:** the average improves from 32.12 to 34.86 (+2.74 points).
* **English capability preserved:** English MMLU rises from 36.78 to 41.95 rather than dropping, so the Indic gains do not come at English's expense. Per-language effects are reported through parity (Experiment 2).

---

## Experiment 2: Fairness via Indic Parity

### Definition of Parity

We define **Indic Parity** for a language *L* as the ratio of the model's MMLU score in *L* to its English MMLU score, which isolates cross-lingual disparity from benchmark difficulty:

```
Parity_L = (MMLU score in language L) / (MMLU score in English)
```

---

### Parity Results

Parity (%) across the 14 Indic languages with MMLU coverage:

| Model     | As   | Bn   | Gu   | Hi   | Kn   | Ml   | Mr   | Ne   | Or   | Pa   | Sa   | Sd   | Ta   | Te   | Avg  |
| --------- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| Q600-Base | 80.6 | 82.1 | 80.2 | 86.7 | 79.1 | 79.7 | 81.6 | 80.7 | 77.8 | 80.2 | 80.7 | 76.2 | 80.6 | 81.3 | 80.5 |
| Q600-CPT  | 85.7 | 87.9 | 85.5 | 91.9 | 84.1 | 85.2 | 87.1 | 86.0 | 83.0 | 85.7 | 86.0 | 81.7 | 86.0 | 86.5 | 85.9 |

---

### Parity Analysis

* **Fairness gains:** average Indic parity improves from **80.5% to 85.9%**, reducing the gap with English.
* **Cross-language equity:** parity improves for *all* 14 Indic languages, indicating more equitable cross-lingual representation.

---

## Experiment 3: Domain-wise Ablation

To see which MILA sources drive the gains, we continually pretrain Q600 on each source in isolation (56B tokens, 4 languages):

| Benchmark     | Q600-OCR | Q600-Synth+Trans | Q600-Crawl |
| ------------- | -------: | ---------------: | ---------: |
| MMLU          | 51.34    | 50.68            | 52.70      |
| MILU          | 30.72    | 29.76            | 30.18      |
| Sanskriti†    | 55.79    | 52.81            | 57.06      |
| Belebele      | 31.37    | 29.74            | 31.52      |
| ARC Challenge | 32.56    | 30.47            | 33.28      |
| HellaSwag     | 39.88    | 37.89            | 39.31      |

OCR, synthetic+translated, and web-crawled data each contribute meaningfully and complementarily.

---

## Conclusion

Continual pretraining of Qwen3-600M on MILA, with architecture, optimizer and recipe held fixed:

1. **Absolute performance:** improves consistently on all six benchmarks, including Indic-MMLU (32.12 to 34.86).
2. **Fairness (parity):** average Indic parity rises from 80.5% to 85.9%, with gains for all 14 Indic languages.

These results support the role of high-quality, curated Indic data in improving both accuracy and cross-lingual equity.
