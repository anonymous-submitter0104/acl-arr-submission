# MILA: MULTILINGUAL INDIC LANGUAGE ARCHIVE
_A DATASET FOR EQUITABLE MULTILINGUAL LLMS_

### Metadata for Opensource Data: [MetaData](croissant-file/)
---

 ![Dataset Distribution](/readme-resources/compositional_analysis_with_english.png)

**Anonymous ACL ARR submission.** This repository accompanies the paper *MILA (Multilingual Indic Language Archive): A Dataset for Equitable Multilingual LLMs*.

---

## Overview

MILA is a curated multilingual corpus of **10 trillion tokens** (7T Indian-language tokens and 3T English web-crawl tokens) spanning **22 scheduled Indic languages and English**. **1 trillion high-quality, metadata-mapped tokens** are released openly for research.

The corpus is built through a multi-stage pipeline: large-scale web acquisition, script-aware OCR for under-digitized Indic scripts, LLM-assisted translation post-correction, targeted data distillation via the Indic PersonaHub, and seed-driven synthetic augmentation and rewriting. Curation applies deduplication, toxicity filtering, PII redaction, language identification, quality filtering, and benchmark decontamination, with each major stage supervised by LLM judges selected through our own Indic benchmarking.

For evaluation we use **Indic-MMLU** (22 languages) and introduce a **cross-lingual parity metric**. We also train a 3B-parameter dense model and a 17B-parameter mixture-of-experts model on MILA.

---

## Open Source Release

**We will open source the following:**

* **1 Trillion tokens** of high-quality pretraining data across **22 Scheduled Indian Languages and English**
* **OCR-extracted data** for **10 of the 22 languages** (Assamese, Bengali, Hindi, Kannada, Malayalam, Marathi, Sanskrit, Tamil, Telugu, Urdu). Public release of OCR-extracted text is limited to these languages by licensing restrictions on the underlying scanned sources
* **Image–text pairs** for Indic OCR and VLM development
* **Indic Persona Hub** (200 Million Indian virtual personas) and persona-generated data
* **India-centric parallel translated corpora** across **22 Scheduled Indian Languages**
* **Synthetic rewrites** and augmented data
* **Indic-MMLU** benchmark covering 22 languages
* **ISOB**, the Indic Synthetic OCR Benchmark, across **22 Scheduled Indian Languages** (synthetically generated, hence released for all 22 languages)
* **Domain-specific Indian taxonomies**
* **High-quality web-crawled English** and **Indic web crawl**
* **Crawling, scraping, curation and OCR pipelines**

**_A representative subset, reflecting the committed full open release, is available in this repository. Upon paper acceptance, the full dataset, together with proper legal licensing, will be released on GitHub and Hugging Face, in accordance with open-source best practices._**

---

## 📂 Repository Overview

This repository contains the resources, scripts, and datasets associated with our ACL ARR submission. Each major section of the paper has a corresponding folder, containing **training scripts, ablation study scripts, and detailed READMEs** to support **reproducibility**.

### 🎯 Main Sections

1. [**Data Acquisition**](https://github.com/anonymous-submitter0104/acl-arr-submission/tree/main/data-acquisition)
2. [**Data Curation**](https://github.com/anonymous-submitter0104/acl-arr-submission/tree/main/data-curation)
3. [**Data Organisation**](https://github.com/anonymous-submitter0104/acl-arr-submission/tree/main/data-organisation)
4. [**Indic MMLU**](https://github.com/anonymous-submitter0104/acl-arr-submission/tree/main/indic-mmlu)
5. [**OCR Pipeline**](https://github.com/anonymous-submitter0104/acl-arr-submission/tree/main/ocr-pipeline)
6. [**ISOB**](https://github.com/anonymous-submitter0104/acl-arr-submission/tree/main/isob)
7. [**Translation Pipeline**](https://github.com/anonymous-submitter0104/acl-arr-submission/tree/main/translation-pipeline)
8. [**Rewriting & Data Distillation**](https://github.com/anonymous-submitter0104/acl-arr-submission/tree/main/rewriting-data-distillation)
9. [**Final Experiment**](https://github.com/anonymous-submitter0104/acl-arr-submission/tree/main/final-experiment)

---

### 🌐 Open Source Release

For reviewer access, a **representative subset** of the full open-source release is provided in the `opensource-release` folder. Upon **acceptance of the paper**, the complete datasets will be made publicly available with proper **metadata tagging** and under an **open-source license**.

**Available Open Source Subfolders:**

0. [**Open Source Release**](https://github.com/anonymous-submitter0104/acl-arr-submission/tree/main/opensource-release): contains all of the folders below.
1. [**Indic MMLU**](https://github.com/anonymous-submitter0104/acl-arr-submission/tree/main/opensource-release/Indic%20MMLU)
2. [**Image-Text Pairs**](https://github.com/anonymous-submitter0104/acl-arr-submission/tree/main/opensource-release/image-text-pairs)
3. [**Indic Persona Hub**](https://github.com/anonymous-submitter0104/acl-arr-submission/tree/main/opensource-release/indic-personahub)
4. [**ISOB-SMALL-HARD**](https://github.com/anonymous-submitter0104/acl-arr-submission/tree/main/opensource-release/isob-small-hard)
5. [**Training Corpus**](https://github.com/anonymous-submitter0104/acl-arr-submission/tree/main/opensource-release/training-corpus): representative corpora for model training:

   * **Indian English**
   * **Parallel Corpus**
   * **Web Crawl**

Each of the above folders contains its own **README** detailing structure, format, and instructions for usage.

---

## Responsible Data Practices

* **PII:** multilingual PII detection and redaction is applied across OCR, translated and web-sourced text.
* **Toxicity:** a multilingual toxicity classifier filters harmful content before release.
* **Licensing:** every source is tagged with licensing metadata and provenance. Items with restrictive or ambiguous licenses are excluded from the open release, and OCR-derived data is released only for the 10 languages listed above.
* **Known limitations:** language imbalance across the 22 languages, bias in LLM-judge quality labels and persona/synthetic data, and the risk of marginalizing regional dialects are discussed in the paper's Limitations section.

---

## Disclaimer

This repository is part of a research effort submitted to ACL ARR.

Our objective is to **open-source large-scale Indian multilingual datasets** to strengthen the **open-source AI ecosystem** and promote **data sovereignty within the Indic AI community**.

All our pipelines are built entirely on **open-source models from Hugging Face**, ensuring full transparency and reproducibility. We **do not use any closed-source LLMs, VLMs, or proprietary AI systems** at any stage of data creation or processing, a deliberate choice to uphold **data sovereignty and ethical independence** in our work.

If required, we are open to publishing benchmark comparisons against **closed-source or proprietary systems**. However, we have consciously chosen **not to use** such systems in our workflow.

This decision stems from our **commitment to ethical data handling** and our **partnership agreements** with multiple data providers. Using closed or proprietary systems for tasks like OCR, translation, or data processing would involve transferring sensitive data to external servers, which is **beyond the scope of our agreements** and therefore **ethically inappropriate**.

To uphold these standards, we have developed **fully open-source pipelines** that can be **deployed locally on GPU infrastructure**, ensuring both transparency and data integrity throughout the process.

We intend to release not only the **datasets** but also the **data preparation recipes** and the accompanying open-source code, so that the community can reproduce, extend, and build on our methods in a transparent and sovereign manner.

---

## ❤️ A Note from the Authors

This work is a foundational step in what we believe to be one of the most important and underexplored areas of modern AI: **data preparation for training large language models**. Our experiments and ablation studies reaffirm that **careful data preparation is a cornerstone** for building performant and responsible LLMs.

Data preparation for pretraining remains one of the most complex, least standardized, and least openly studied components of the LLM pipeline. By presenting this work in a **peer-reviewed, open-source setting**, we hope to encourage transparency, reproducibility, and community validation, and to inspire other research groups to bring similar efforts into the open.

We believe progress in generative AI should be built on openness, shared validation, and collective growth.
