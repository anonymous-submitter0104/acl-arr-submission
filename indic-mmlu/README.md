# Indic MMLU — Benchmarking Open-Source LLMs on Indian Languages

Indic-MMLU is a multilingual adaptation of the CAIS/MMLU test set, covering the **22 scheduled Indian languages and English**. It is designed to evaluate LLMs' ability to understand Indian languages, and to select models for rewriting, distillation and instruction-data generation in the MILA pipeline.

This folder accompanies the anonymous ACL ARR submission *MILA (Multilingual Indic Language Archive): A Dataset for Equitable Multilingual LLMs*; see Appendix D (Indic MMLU) of the paper.

---
# Table of Contents

1. [Why the Need for Indic MMLU](#why-the-need-for-indic-mmlu)
   - [Data Distillation for Indian Languages](#data-distillation-for-indian-languages)
   - [The Role of Indic MMLU](#the-role-of-indic-mmlu)
2. [What This Dataset Has on Offer](#what-this-dataset-has-on-offer)
3. [Coverage](#coverage)
4. [Overview](#overview)
5. [High-Level Pipeline](#high-level-pipeline-what-we-did)
   - [Indic MMLU Construction Algorithm](#indic-mmlu-construction-algorithm)
6. [Translation Algorithm](#translation-algorithm)
7. [Reproducible Commands](#reproducible-commands-examples)
8. [Download Indic MMLU Dataset](#download-indic-mmlu-dataset)
9. [Model Evaluation Methodology](#model-evaluation-methodology)
10. [Indic MMLU Results](#indic-mmlu-results)
11. [How to Reproduce (Quick Checklist)](#how-to-reproduce-quick-checklist)
12. [Best Practices & Caveats](#best-practices--caveats)
13. [Visual Assets](#visual-assets)

---

## Why the Need for Indic MMLU

The rise of large language models (LLMs) has enabled advanced **synthetic data generation**, including:

* Building LLM-based frameworks for **data curation and sandboxing**
* Generating reasoning and coding examples in non-English languages
* Creating SFT datasets, as well as off-policy datasets for **RLHF, DPO, GRPO**-style training

To fully leverage these capabilities for Indian languages, we need LLMs with at least **limited native proficiency** in these languages. Such models allow us to generate high-quality, culturally aligned datasets, create virtual personas to introduce nuanced constructs, and maintain **full control and flexibility** over the synthetic data pipeline.

### Data Distillation for Indian Languages

Data distillation provides a practical approach to achieve this:

* **Easier and cleaner access to world knowledge** from state-of-the-art LLMs
* **Domain control and flexibility** to shape datasets according to cultural and linguistic requirements
* Creating safety datasets and culturally grounded content through **virtual Indian personas**

**Distillation** is the process of generating data from large models to produce **task-specific, high-quality, and culturally aligned datasets**. Using distillation, we can apply synthetic data augmentation strategies such as:

* Chain-of-Thought (CoT) reasoning
* Multi-turn conversational setups for richer data
* Summaries and diverse QA pairs
* Persona-conditioned generation through **virtual Indian personas**
* Other augmentations, including translations and language-specific refinements

### The Role of Indic MMLU

Before deploying LLMs for such Indian-language data generation, we first need to **evaluate their cross-lingual knowledge transfer capabilities**. Current LLM evaluation remains overwhelmingly English-centric, obscuring whether models genuinely reason in Indic languages or rely on hidden translation heuristics. Indic-MMLU provides a standardized benchmark spanning multiple scripts and typologies, enabling assessment of native generalization versus resource-driven collapse arising from limited exposure or tokenization mismatch.

Indic-MMLU is used to:

* Rigorously evaluate multilingual LLMs on Indic languages
* Measure cross-lingual knowledge transfer from English to low-resource scripts
* Select models for rewriting, distillation and instruction-data generation, and for the LLM-as-a-judge roles used in the MILA quality, toxicity and translation-correction pipelines

It is provided with an **open translation recipe and code** and a reproducible evaluation setup.

---

## What this dataset has on offer

The **Indic MMLU dataset**:

* Translates the English **[CAIS/MMLU](https://huggingface.co/datasets/cais/mmlu)** test set into **22 Indian languages**, using the translation methodology of IndicMMLU-Pro.
* Refines the translations with an open-source LLM, with an LLM-based verifier in place of IndicMMLU-Pro's expert human proofreading step.
* Provides a reproducible evaluation setup for measuring open-source models across all 22 languages and English.

If a model's score on an Indic version is close to its score on the original English MMLU, it indicates **genuine language understanding** and strong cross-lingual generalization. This is the idea behind the *parity* metric used in the paper.

---

## Coverage

* **Dataset scope:** 22 scheduled Indic languages, plus English.
* **Evaluation scope:** all 22 languages plus English, covering 26 open-source LLM configurations (open frontier and small-scale models).

---

## Overview

* **Source benchmark:** CAIS/MMLU (general knowledge QA benchmark), English test set.
* **Target:** 22 Indic languages + English (translated test set).
* **Workflow:** machine translation → LLM-based translation refinement → embedding comparison → persona-based LLM-as-judge scores → LLM-based verification → automated evaluation.
* **Deliverables:** translated `.jsonl` datasets, per-language embeddings, similarity reports, LLM-judge ratings, model evaluation scores and visualizations (radar plots).

---

## Key principles applied

1. **Preserve semantic parity:** questions remain semantically the same as English; only the language changes.
2. **Quality control:** model-based automated checks (embeddings, math correctness) and an LLM-based verifier.
3. **Reproducibility:** Dockerized inference environment, scriptable pipeline, and clear CLI commands.
4. **Transparency:** similarity metrics and LLM-judge ratings are provided to document translation fidelity and linguistic quality.

---

## High-level pipeline (what we did)

1. **Translate** the CAIS/MMLU test set to each Indic language using a specialist MT model (*IndicTrans2*, baseline MT).
2. **Refine** those translations using an open-source LLM with task-specific instructions (to correct phrasing, clarify ambiguous translations, preserve math expressions, and maintain answer-choice alignment).
3. **Embed** both the original English items and the refined translated items with the same embedding model.
4. **Compare** embeddings using cosine similarity to measure semantic closeness.
5. **LLM-as-judge verification:** LLM judges under expert personas rate samples for language quality, math correctness and coherence.
6. **Evaluate models:** run open-source LLMs across all languages using the `lm-eval` framework and visualize results (radar plots, top-5 per language).

---

## Indic MMLU Construction Algorithm

The following flowchart gives a **visual overview of the Indic MMLU pipeline**, from the English MMLU dataset through translation, refinement, judging and validation:

* **Translation Stage:** the English MMLU instance is translated into Indic languages using specialist MT.
* **Refinement Stage:** translations are refined with large-scale LLMs for improved fluency and alignment.
* **LLM-based Judging Phase:** three kinds of evaluation — *linguistic quality, subject (math) correctness, and coherence*.
* **Validation Stage:** embedding similarity checks, n-gram metrics (BLEU, ROUGE, chrF++) and discrepancy detection ensure quality.
* **Final Output:** a validated Indic MMLU dataset instance ready for benchmarking.

![Indic MMLU Pipeline](assets/indic-mmlu.png)

# Translation Algorithm

```
Input:
  original_mmlu
  target_languages
  instructions

For each language L in target_languages:
    translated = translate(original_mmlu, tgt=L)
    enhanced = refine_using_llm(translated, instructions)

    eng_emb = embed(original_mmlu)
    trans_emb = embed(enhanced)
    sim_scores = cosine_similarity(eng_emb, trans_emb)

    llm_scores = LLM_as_Judge(original_mmlu, translated)

    consistency = consistency_check(sim_scores, llm_scores)

    If consistency is poor:
        re-verify / re-refine the sample
    Else:
        accept enhanced

Aggregate all scores:
    combine sim_scores, llm_scores
    compute per-language model scores

Produce visualizations:
    generate charts and plots for cross-language comparison

Output:
    per-language evaluation scores
    visualizations
```

---

## Reproducible commands (examples)

> These are the commands used in our pipeline. Replace file paths, ports, and concurrency values as appropriate for your environment.

**Translate**

```bash
python3 trans_mmlu.py \
  --input_file_path test-00000-of-00001_with_ids.jsonl \
  --output_file_path mmlu_kn_in.jsonl \
  --tgt_lang "kn" \
  --src_lang "en"
```

**Refine translations (LLM-based)**

```bash
python3 async_infr.py \
  --input-path mmlu_as_in.jsonl \
  --output-file ds_enhance_eval_hi.jsonl \
  --instruction-path instruction_prompts.yml \
  --task deepseek_enhance_instruct \
  --template-fields 'og_question' 'og_choices' 'question' 'answer' 'Hindi' \
  --max-concurrency 4096 \
  --backend sglang-oai-chat \
  --extra-request-body '{"temperature":0.7,"top_p":0.9,"top_k":50,"max_tokens":1024}' \
  --port 30000
```

**Generate embeddings**

```bash
python3 get_embeddings.py \
  --input-file test-00000-of-00001_with_ids.jsonl \
  --output-file mmlu_qwen3_embd.jsonl \
  --server-url http://127.0.0.1:30000 \
  --concurrency 4096
```

**Cosine similarity (English vs translated)**

```bash
python3 calculate_cosine_similarity.py \
  --input_dir deepseek_enhance \
  --output_dir mmmlu/similarity/cosine/enhance/deepseek_enhance \
  --english_file mmlu_eng.jsonl
```

**LLM-as-judge rating (linguist persona)**

```bash
python3 async_infr.py \
  --input-path mmlu_as_in.jsonl \
  --output-file ds_enhance_eval_as_qwen3_instruct.jsonl \
  --instruction-path instruction_prompts.yml \
  --task rate_translated_text_linguist \
  --template-fields 'og_question' 'og_choices' 'enhanced_text' 'Assamese' \
  --max-concurrency 4096 \
  --backend sglang-oai-chat \
  --extra-request-body '{"temperature":0.7,"top_p":0.9,"top_k":50,"max_tokens":1024}' \
  --port 30000
```

**Model evaluation with `lm-eval`**

```bash
# Example wrapper which runs evaluation for a model snapshot against the Indic MMLU dataset
bash lm-eval-llm.sh /path/to/model_snapshot /benchmark-result-path
```

---

## Download Indic MMLU Dataset

The complete benchmark covering **22 Indic languages + English** is available here:

- [Download Full Benchmark (PARQUET)](https://github.com/anonymous-submitter0104/acl-arr-submission/tree/main/opensource-release/Indic%20MMLU)

> Please cite the accompanying paper if you use the dataset in your research.

---

## Model Evaluation Methodology

We evaluate open-source LLMs across all **22 Indic languages** plus English using the Indic MMLU benchmark.

Workflow:

1. **Environment Setup:** Dockerized `vllm/vllm-openai` for reproducibility.
2. **Framework:** `lm-eval` for standardized evaluation.
3. **Metrics:** per-language accuracy, top-5 rankings per language, aggregate averages.
4. **Visualization:** radar plots for each language and comparative top-model plots.

---

## Indic MMLU Results

Indic MMLU scores (average across languages) for open-source frontier and small-scale models (Appendix D.2 of the paper):

| Model                           | Parameters | Indic MMLU |
| ------------------------------- | ---------: | ---------: |
| DeepSeekR1-0528                 | 685B       | 0.67       |
| DeepSeekV3-0324                 | 685B       | 0.68       |
| DeepSeekV3.1                    | 685B       | 0.68       |
| Gemma-3 27B                     | 27B        | 0.63       |
| gpt-oss-120B (High / Med / Low) | 120B       | 0.48       |
| gpt-oss-20B (High / Med / Low)  | 20B        | 0.52       |
| kimi-k2                         | 1T         | 0.66       |
| Qwen3-235B-A22B-Instruct-2507   | 235B       | **0.72**   |
| Qwen3-235B-A22B-Thinking-2507   | 235B       | 0.70       |
| Qwen3-0.6B                      | 0.6B       | 0.32       |
| GLM 4.5                         | 358B       | 0.66       |
| Gemma-7B                        | 7B         | 0.24       |
| Mistral-7B-v0.1                 | 7B         | 0.28       |
| Mistral-7B-v0.2                 | 7B         | 0.29       |
| Mistral-7B-v0.3                 | 7B         | 0.27       |
| LLaMA-4-Scout                   | 109B       | 0.64       |
| LLaMA-4-Maverik                 | 402B       | **0.72**   |
| Mistral-Small-24B-Instruct-2501 | 24B        | 0.41       |
| LLaMA-3.1-70B                   | 70B        | 0.60       |
| Qwen3-Next-80B-A3B-Thinking     | 80B        | 0.65       |
| Qwen3-Next-80B-A3B-Instruct     | 80B        | 0.67       |
| Mistral-Large-Instruct-2411     | 24B        | 0.54       |

LLaMA-4-Maverik, Qwen3-235B-A22B-Instruct-2507, Qwen3-235B-A22B-Thinking-2507, DeepSeekV3-0324 and DeepSeekV3.1 are the top-5 models by average Indic MMLU score. Per-language top-5 lists are in the paper (Appendix D.2). Only models that place highly here are promoted to the LLM-as-a-judge roles used in the MILA quality, toxicity and translation-correction pipelines.

Per-language radar plots:

| <div align="center">![Assamese](assets/assamese.png)<br/>*Assamese*</div> | <div align="center">![Bengali](assets/bengali.png)<br/>*Bengali*</div> |
|---------------------------------------------------------------------------|------------------------------------------------------------------------|
| <div align="center">![English](assets/English.png)<br/>*English*</div>    | <div align="center">![Gujarati](assets/Gujrati.png)<br/>*Gujarati*</div> |
| <div align="center">![Hindi](assets/Hindi.png)<br/>*Hindi*</div>          | <div align="center">![Kannada](assets/kannada.png)<br/>*Kannada*</div> |
| <div align="center">![Maithili](assets/maithili.png)<br/>*Maithili*</div> | <div align="center">![Malayalam](assets/Malayalam.png)<br/>*Malayalam*</div> |
| <div align="center">![Marathi](assets/marathi.png)<br/>*Marathi*</div>    | <div align="center">![Nepali](assets/Nepali.png)<br/>*Nepali*</div>   |
| <div align="center">![Oriya](assets/oriya.png)<br/>*Oriya*</div>          | <div align="center">![Punjabi](assets/Punjabi.png)<br/>*Punjabi*</div> |
| <div align="center">![Sanskrit](assets/sanskrit.png)<br/>*Sanskrit*</div> | <div align="center">![Sindhi](assets/sindhi.png)<br/>*Sindhi*</div>   |
| <div align="center">![Tamil](assets/Tamil.png)<br/>*Tamil*</div>          | <div align="center">![Telugu](assets/Telugu.png)<br/>*Telugu*</div>   |

---

## How to reproduce (quick checklist)

1. Prepare the English CAIS/MMLU test set (`.jsonl` with IDs).
2. Run `trans_mmlu.py` to produce per-language MT outputs.
3. Run `async_infr.py` with the refinement task to obtain improved translations.
4. Generate embeddings for both English and refined translations.
5. Run `calculate_cosine_similarity.py` to produce similarity reports.
6. Optionally run the LLM-as-judge rating orchestration for additional quality assessment.
7. Evaluate models with `lm-eval` and generate visualizations.

---

## Best practices & caveats

* **Math and formatting:** preserve mathematical expressions and numerals during translation; these are critical for correctness.
* **Choice alignment:** ensure answer choices remain in the same order and map to the same ground-truth labels.
* **LLM-based verification:** the benchmark uses an LLM-based verifier in place of expert human proofreading; periodic sampling by native speakers is recommended for downstream use.
* **Benchmark-construction caveat:** Indic-MMLU is built via a translation pipeline related to the one used for MILA, so models trained on MILA may be evaluated partly on a benchmark built with related machinery. We mitigate this by decontaminating Indic-MMLU against all four MILA sources and by reporting external benchmarks (MILU, Sanskriti, Belebele, Bhashabench, English MMLU) alongside it.
* **Constrained inference:** use deterministic LLM settings for refinement (lower temperature, consistent decoding) to reduce hallucination.

---

## Visual assets

Per-language radar plots and a consolidated results panel are in `indic-mmlu/assets/`. The README in the repo contains thumbnails; click through for full-resolution visuals.
