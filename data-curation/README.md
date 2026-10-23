# Data Curation

This folder accompanies the anonymous ACL ARR submission *MILA (Multilingual Indic Language Archive): A Dataset for Equitable Multilingual LLMs*. It describes our **curation pipeline** and the **ablation experiments** conducted to measure its effectiveness. For the full methodology see Section 3.2 (Data Curation) and Appendix B of the paper; the ablations are in Appendix C.1 and C.3.

---

# Table of Contents

1. [Overview](#data-curation)
2. [Folder Structure](#folder-structure)
3. [Curation Pipeline Overview](#curation-pipeline-overview)
   - [Pipeline Diagram](#pipeline-diagram)
   - [Description of Stages](#description-of-stages)
4. [Evaluation Procedure](#evaluation-procedure)
   - [Benchmarks Used](#benchmarks-used)
   - [Evaluation Strategy](#evaluation-strategy)
   - [Key Expectations](#key-expectations)
5. [Ablation Experiment 1: Efficacy of the Curation Pipeline](#ablation-experiment-1-evaluating-the-efficacy-of-the-curation-pipeline)
   - [Model Details](#model-details)
   - [Training Data Composition](#training-data-composition)
   - [Key Design Considerations](#key-design-considerations)
   - [Experiment Replication Procedure](#experiment-replication-procedure)
   - [Results Obtained: Conventional vs Curated](#results-obtained-conventional-vs-curated)
   - [Conclusion](#conclusion)
6. [Ablation Experiment 2: Toxicity Evaluation](#ablation-experiment-2-toxicity-evaluation)
7. [Conventional vs Curated Data Sample](#conventional-vs-curated-data-sample)

---

## Folder Structure

```
acl-arr-submission/
 ├── data_curation/
 │    ├── curation/
 │    │    ├── cc_curator.py
 │    │    ├── language_detector.py
 │    │    ├── nemo_curator.py
 │    │    ├── streaming_regex.py
 │    ├── deduplication/
 │    │    ├── deduplication.py
 │    │    └── deduplication.sh
 │    ├── toxic_filter/
 │    │    ├── sample_toxic_words.txt
 │    │    ├── toxic_filter_inference.py
 │    │    └── toxic_filter_rule.py
 │    ├── quality_filter/
 │    │    ├── quality_filter.py
 │    └── README.md
```

## Curation Pipeline Overview

### Pipeline Diagram
![Curation Pipeline](/readme-resources/data-curation.png)

### Description of Stages

The pipeline is multi-stage and Indic-specific, inspired by general-purpose curation frameworks (NeMo Curator) and augmented with custom language-aware modules. It is applied uniformly to all four MILA sources (OCR, crawl, translation and synthetic data), across the 22 scheduled Indic languages and English.

1. **Modifiers and Heuristic Filters**

   * Modifiers normalize text: boilerplate string removal, HTML tag removal, Unicode reformatting, quotation unification and excess whitespace removal.
   * Heuristic filters remove degenerate content: word count, repeating top-*n*-grams (*n* = 2, 3), URLs, symbols-to-words ratio and numbers filters.
2. **Two-level Deduplication**

   * Exact duplicates are removed via URL and MD5 signatures.
   * Near-duplicates (templated pages, OCR variants, lightly modified copies) are removed via GPU-accelerated fuzzy deduplication.
3. **Multilingual Language Identification**

   * The pretrained fastText language-identification model is used off-the-shelf, combined with regex-driven rules that enforce script–language consistency.
   * This mitigates cross-script contamination, Romanized drift and mixed-language artifacts common in web sources.
4. **Toxicity Filtering**

   * Two stages: rule-based scanning, followed by a fine-tuned IndicBERTv2-MLM-only classifier that recovers false positives and refines borderline cases.
5. **PII Identification and Redaction**

   * A multilingual PII module identifies and redacts personally identifiable information in Indian languages.
6. **Quality Filtering**

   * Language-specific IndicBERTv2-MLM-only classifiers label documents on a five-level ordinal quality scale (High, High–Medium, Medium, Medium–Low, Low), judged on linguistic well-formedness and semantic coherence.
7. **Benchmark Decontamination**

   * $n$-gram overlap filtering (8–13 grams) combined with Infinigram-based cross-domain detection, applied across all four sources.
8. **Final Curated Dataset**

   * High-quality, domain-diverse corpus passed to training.

## Evaluation Procedure

To assess the impact of the **curation pipeline**, we compare models trained on **conventional data** versus **curated data** under otherwise identical conditions.

**Purpose:**
The evaluation quantifies the benefit of data curation on reasoning and knowledge benchmarks, isolating the effect of data quality from scale.

### Benchmarks Used:

* **[ARC Challenge & ARC Easy](https://huggingface.co/datasets/allenai/ai2_arc)** – reasoning-focused multiple-choice questions.
* **[HellaSwag (English & Hindi)](https://huggingface.co/datasets/Rowan/hellaswag)** – commonsense reasoning and next-event prediction.
* **[MMLU (English & Hindi)](https://huggingface.co/datasets/cais/mmlu)** – multitask knowledge evaluation across academic and professional subjects.

### Evaluation Strategy:

* We train two instances of a 3B-parameter dense causal language model **from scratch** on **2 trillion tokens of English and Hindi data**: one on raw web-scraped text processed only with basic cleaning (conventional), and one on data processed with the full curation pipeline (curated).
* Architecture, hyperparameters, training duration, batch size, learning-rate schedule and infrastructure are **identical** across the two runs, so performance differences are attributable to data quality alone.
* Both datasets are **matched in size (2T tokens)**, so differences reflect curation rather than data volume.

### Key Expectations:
We expect the model trained on curated data to show:

1. Higher accuracy across the benchmarks.
2. Larger gains in lower-resource settings (e.g., Hindi variants).
3. Reduced propagation of low-quality, toxic or redundant content.

## Ablation Experiment 1: Evaluating the Efficacy of the Curation Pipeline

### Model Details:

* **Model:** 3B-parameter dense causal language model (grouped-query attention, 32 hidden layers, hidden size 2048, intermediate size 7168, SiLU activations, RoPE).
* **Training:** from scratch (no pretrained checkpoint) on 2T tokens of English and Hindi, under two conditions (conventional vs curated).
* **Architecture details:** see Appendix C.1 (Experiment: 3B Dense Model) of the paper. The curated instance is the 3B model reported in the main-paper results.

### Training Data Composition

1. **Conventional Corpus (Uncurated)**

   * Raw web-scraped text with only basic cleaning.
   * The 2T-token subset was sampled from the [DCLM corpus](https://github.com/mlfoundations/dclm), with **30% of tokens translated into Hindi** to maintain bilingual coverage.
   * No additional quality filters, reflecting typical large-scale pretraining data.

2. **Curated Corpus (Curation Applied)**

   * Derived from the same underlying sources as the conventional corpus.
   * Processed through the curation pipeline described above (deduplication, heuristic filtering, language identification, toxicity filtering, PII redaction, quality filtering).
   * A **2T-token subset** matching the conventional corpus in size.

### Key Design Considerations:

* Both datasets contain **exactly 2T tokens**, eliminating confounding effects of scale.
* All training settings are identical, so observed differences can be attributed to **data quality and curation**.
* The **30% Hindi tokens** ensure the evaluation captures bilingual performance.

### Experiment Replication Procedure

The scripts and codebase needed to replicate this experiment are provided in this repository.

**Curation Scripts and Codebase**

All scripts are provided under [`data_curation/`](data_curation/).

**Key Components**

* `curation/` → Curation pipeline (cleaning, heuristic filters, PII redaction, language identification)
* `deduplication/deduplication.sh` → Bash file for global deduplication
* `quality_filter/quality_filter.py` → Quality filter
* `toxic_filter/toxic_filter_rule.py` → Rule-based toxic filtering (word list included for 1 language)

**Steps to Run**

1. **Setup Environment**

   ```bash
   conda create -n curator python=3.10 -y
   conda activate curator
   pip install nemo_toolkit[all] transformers datasets
   ```

2. **Download Raw Corpus**

   ```bash
   bash scripts/download_raw.sh --languages en,hi --output data/raw/
   ```

3. **Run Curation Pipeline**

   ```bash
   bash scripts/run_curator.sh --config configs/curator.yaml \
                               --input data/raw/ \
                               --output data/curated/
   ```

4. **Inspect Curated Data**

   ```bash
   jupyter notebook notebooks/quality_checks.ipynb
   ```

5. **Use in Pretraining**

   * Curated and non-curated corpora are directly pluggable into the NeMo pretraining recipes.
   * Training scripts are provided under [`experiments/pretraining/`](experiments/pretraining/).


### Results Obtained: Conventional vs Curated

We evaluated models trained on the conventional and curated datasets across six benchmarks (Appendix C.1, Table "Benchmark results: conventional vs. curated 3B training data").

| **Model**      | **ARC Challenge** | **ARC Easy** | **HellaSwag** | **HellaSwag-Hi** | **MMLU** | **MMLU-Hi** |
|----------------|------------------:|--------------:|---------------:|------------------:|---------:|------------:|
| Conventional   | 43.5              | 73.6          | 72.9           | 38.9              | 41.3     | 26.2        |
| Curated        | **46.7**          | **77.31**     | **73.40**      | **44.1**          | **46.2** | **30.7**    |

### Observations

1. **Consistent improvement on all six benchmarks.** The curated model outperforms the conventional model everywhere.

2. **Hindi benefits strongly.** HellaSwag-Hi rises from 38.9 to 44.1 (+5.2) and MMLU-Hi from 26.2 to 30.7 (+4.5). Noisy training data disproportionately degrades performance in the lower-resource language, where script variation, code-mixing and sparse high-quality resources make conventional cleaning insufficient.

3. **English also improves.** ARC Challenge rises from 43.5 to 46.7, ARC Easy from 73.6 to 77.31 (+3.71), and MMLU from 41.3 to 46.2 (+4.9). HellaSwag improves slightly (72.9 to 73.40).

### Conclusion

* The curated model outperforms the conventional model on all six benchmarks under identical architecture, training configuration and token budget.
* The gains are visible in both English and Hindi, and are pronounced for Hindi, which supports the importance of Indic-specific curation for multilingual corpora.

## Ablation Experiment 2: Toxicity Evaluation

### Objective
To evaluate the effect of data curation on model safety, we measure toxicity of the curated 3B model from Experiment 1.

We use the **Toxigen** benchmark via the **LLM360 Safety360** suite, which provides explicit and subtle adversarial prompts spanning identity-based categories (race, religion, gender, nationality) and general offensive content. Model completions from Toxigen templates are classified by a RoBERTa-based detector fine-tuned for nuanced, context-dependent toxic language. We compare the curated 3B model against three multilingual baselines: Sarvam-2B, Llama-3.2-3B and Gemma-2-2B, which were trained on substantial corpora but without our curation pipeline.

![Toxicity Sample](/readme-resources/toxic-comparison)

### Key Findings

* **Lower toxicity than baselines:** across 16 neutral and adversarial evaluation sets, the curated 3B model stays below all three baselines in nearly every condition.
* **Adversarial identity-based prompts:** the most challenging category; the curated model's toxic-output rate is 15–30% lower than the baselines'.
* **No loss of utility:** the same checkpoint remains competitive on the task benchmarks above.
* **Link to the toxicity filter:** the results indicate that the two-stage toxicity filter removes training examples that would otherwise teach models to reproduce harmful patterns, without introducing false positives severe enough to degrade linguistic coverage.

See Appendix C.3 (Ablation Study: Safety and Toxicity Reduction) and Appendix B.3 (Toxicity Classifier) of the paper.

---
## Conventional vs Curated Data Sample

The following is an example of Assamese text before and after curation:
![Curation Sample](/readme-resources/curation.png)

### Observation

After curation, the text is cleaned and standardized to improve model training. HTML and markup tags such as `<div>`, `<br>`, `<i>`, `<b>` and `<span>` are removed, leaving only readable text, and extraneous punctuation and repeated special symbols like `**`, `%%` and `!!` are eliminated to produce more natural sentences. Redundant or repeated phrases are merged or removed, and line breaks and formatting artifacts are smoothed so sentences are coherent and continuous. Non-linguistic metadata, such as authorship markers, instructional notes and bracketed annotations, is stripped. Token normalization unifies encoding, spacing, punctuation and Unicode scripts, minimizing inconsistencies across multilingual text, and potentially toxic content is filtered.
