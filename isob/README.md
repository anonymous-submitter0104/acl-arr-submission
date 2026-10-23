# ISOB-Small: Indic Synthetic OCR Benchmark

ISOB is a synthetic benchmark for evaluating OCR systems on Indic documents. This folder accompanies the anonymous ACL ARR submission *MILA (Multilingual Indic Language Archive): A Dataset for Equitable Multilingual LLMs*; see Appendix C.4 (OCR Pipeline, ISOB) of the paper. The released **ISOB-Small** comprises **110 synthetically generated pages spanning 22 Indian languages**.

## Table of Contents

1. [Motivation](#motivation)
2. [What Makes ISOB Unique](#what-makes-isob-unique)
3. [Benchmark Characteristics](#benchmark-characteristics)
4. [ISOB-Small Construction Recipe](#isob-small-construction-recipe)

   * [Input & Output](#input--output)
   * [Step 0: Initialize Seed Corpus](#step-0-initialize-seed-corpus)
   * [Step 1: Hard Page Identification](#step-1-hard-page-identification)
   * [Step 2: Language Selection](#step-2-language-selection)
   * [Step 3: Artifact Taxonomy Extraction](#step-3-artifact-taxonomy-extraction)
   * [Step 4: Synthetic hOCR Augmentation](#step-4-synthetic-hocr-augmentation)
   * [Step 5: hOCR to Visual Conversion](#step-5-hocr-to-visual-conversion)
   * [Step 6: Style Transformation](#step-6-style-transformation)
   * [Step 7: Image Processing Augmentation](#step-7-image-processing-augmentation)
   * [Step 8: Storage & Annotation](#step-8-storage--annotation)
5. [Download ISOB-Small Dataset](#download-isob-small-dataset)
6. [End Result](#end-result)
7. [License & Data Usage](#license--data-usage)

---

## Motivation

The Indian subcontinent has a wealth of localized textual data spanning multiple languages, dialects, and historical scripts. While large volumes of OCR datasets exist for English and other major languages, **authentic Indian data is fragmented, often offline, and hyper-local**, collected through partnerships with governmental and regional institutions under formal MOUs and legal agreements.

This data includes:

* Remote or hand-digitized archives
* Region-specific dialects
* Manuscripts with non-standard layouts, tables, equations, and mixed scripts

These materials are challenging to OCR due to their variability, rarity, and complexity. Every single token is valuable, and preserving their fidelity is crucial for research and model evaluation.

---

## What Makes ISOB Unique

1. **Reverse-Engineered Synthetic Benchmark:**
   Directly releasing original data is often restricted due to copyright and MOU constraints. To make the research community aware of these challenges, we **reverse-engineered the OCR process**:

   * Constructing ground-truth hOCR by embedding multilingual content
   * Rendering hOCR into images/PDFs resembling authentic documents
   * Applying advanced image transformations to emulate manuscript degradation, complex layouts, and other artifacts

2. **Hard-to-OCR Pages:**
   The benchmark emphasizes pages that are inherently difficult to OCR, including:

   * Multi-column layouts
   * Tables, equations, and figures
   * Mixed scripts and languages
   * Dense or overlapping text

3. **Controlled Ground Truth:**
   Unlike web-scraped datasets, we maintain **precise ground truth** via synthetic reconstruction, enabling reliable evaluation using metrics like CER, WER, PI-WER and character-level 3-gram F1.

---

## Benchmark Characteristics

* **Languages Covered:** 22 Indian languages
* **Size:** 110 synthetically generated pages (ISOB-Small)
* **Artifacts:** multi-column layouts, dense tables and figures, mathematical expressions with mixed scripts, watermarks and stamps, paper folds and shadows, font variations, and controlled blur levels
* **Difficulty:** Small, hard-to-OCR dataset
* **Ground Truth:** Full hOCR with language tags and metadata
* **Synthetic Construction:** Built with the document-centric synthetic generation methodology of Nayana, with additional controlled degradations injected to stress-test OCR robustness
* **Copyright-compliant:** because the pages are synthetic, ISOB can be released for all 22 languages, whereas public release of OCR-extracted production text is limited to 10 languages by licensing restrictions on the underlying scans
* **Metrics:** character error rate (CER), word error rate (WER), position-independent WER (PI-WER) and character-level 3-gram F1

---

## ISOB-Small Construction Recipe

### Input & Output

* **Input:** Seed corpus of OCR’d pages in hOCR format
* **Output:** Synthetic benchmark dataset of hard-to-OCR images with:

  * Ground truth hOCR
  * Language tags
  * Artifact metadata

---

### Step 0: Initialize Seed Corpus

* Use existing OCR’ed pages (JSON/hOCR format) as the starting corpus.

---

### Step 1: Hard Page Identification

* Filter pages based on OCR confidence scores

  * Discard empty or low-text pages
* Use **Qwen-VL-7B** grounded with page hOCR to predict difficulty
* Select pages predicted as **hard-to-OCR**

**Result:** `hOCR_1` (hard page set)

---

### Step 2: Language Selection

* Randomly choose **3–10 languages** from a pool of 22 Indian languages for each page.

---

### Step 3: Artifact Taxonomy Extraction

* Build a taxonomy of “hard artifacts” using LLM guidance:

**Examples of Hard Artifacts:**

* Multi-column layouts
* Dense tables
* Handwriting inserts
* Overlapping scripts
* Reading order complexities
* Equations, figures, pie charts
* Complex tables

---

### Step 4: Synthetic hOCR Augmentation

* Enrich `hOCR_1` using:

  * Selected languages (Step 2)
  * Artifact templates (Step 3)

* Produce `hOCR_2` → serves as ground truth

---

### Step 5: hOCR to Visual Conversion

* Render `hOCR_2` into **PDF/image format**

---

### Step 6: Style Transformation

* Construct a **prompt pool** describing Indian manuscript styles, literature, and domain-specific appearances
* Apply **Qwen image editing** on each page to emulate authentic visual styles

---

### Step 7: Image Processing Augmentation

* Apply low-level transformations to increase OCR difficulty:

  * Orientation changes
  * Contrast/brightness adjustments
  * Noise, blur, distortions

---

### Step 8: Storage & Annotation

* Save final images with metadata:

  * Associated ground truth `hOCR_2`
  * Language tags
  * Style/augmentation metadata

---

## Download ISOB-Small Dataset

ISOB-Small (110 synthetically generated pages spanning 22 Indian languages) is released in this repository.

- [Download ISOB-Small](https://github.com/anonymous-submitter0104/acl-arr-submission/tree/main/opensource-release/isob-small-hard)

> Please cite the accompanying paper if you use the dataset in your research.


---

## End Result

A **structured, multilingual, artifact-rich synthetic OCR benchmark** that captures:

* Hard-to-recognize text cases
* Multi-language content
* Complex visual and structural artifacts

This enables the research community to benchmark OCR models effectively on Indian documents that emulate real-world challenges.

---

## License & Data Usage

* ISOB is synthetically generated rather than sourced from licensed scans, so it is released across all 22 languages (unlike the OCR-extracted production data, which is limited to 10 languages).
* ISOB-Small is published in this repository.

---
