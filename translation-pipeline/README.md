# Translation Benchmarking & Pipeline for Indic Languages

> Hybrid MT→LLM translation pipeline for the 22 scheduled Indic languages: specialist MT models for lexical grounding, Indic-capable LLMs for post-correction and long-context consistency, and LLM-judge verification.

This folder accompanies the anonymous ACL ARR submission *MILA (Multilingual Indic Language Archive): A Dataset for Equitable Multilingual LLMs*; see Section 3.3.2 (Translation Pipeline) and Appendix C.6 of the paper.

---
# Table of Contents

1. [Overview](#overview)
2. [Goals](#goals)
3. [Translation Pipeline](#translation-pipeline)

   * [Step 1 — Specialist MT Generation](#step-1--specialist-mt-generation)
   * [Step 2 — LLM-Based Post-Correction](#step-2--llm-based-post-correction)
   * [Step 3 — Long-Context and Overlap Handling](#step-3--long-context-and-overlap-handling)
   * [Step 4 — Quality Classification and Selective Routing](#step-4--quality-classification-and-selective-routing)
   * [Step 5 — Production Deployment and LLM-Judge Verification](#step-5--production-deployment-and-llm-judge-verification)
4. [Evaluation Metrics](#evaluation-metrics)
5. [Model Inventory](#model-inventory)
6. [Reproducible Commands](#reproducible-commands)
7. [Results](#results)
8. [Key Insights](#key-insights)

---

## Overview

Translation-based data generation addresses the structural scarcity of high-quality parallel corpora for Indic languages, producing parallel data that facilitates knowledge transfer from English into the 22 Indic languages that form the backbone of MILA.

Benchmarking open-source MT systems and multilingual LLMs on Indic translation tests shows two consistent patterns:

* **Specialist MT models** achieve low error rates and strong lexical fidelity, but sound rigid.
* **Generalist LLMs** produce more natural translations, at the cost of weaker vocabulary grounding and occasional semantic drift.

No single model is best across languages. This motivates a **hybrid approach**: MT provides lexical grounding, and LLMs supply fluency and long-context consistency. Controlled overlap-based analysis confirms that LLM rewriting of MT output, using small contextual windows from preceding chunks, improves fluency without degrading meaning.

---

## Goals

1. Benchmark open-source specialist MT models and generalist LLMs.
2. Leverage high-scoring Indic-MMLU LLMs for translation refinement and post-correction.
3. Evaluate translation quality with **chrF++**.
4. Define a reproducible specialist-MT + LLM post-correction strategy for high-fidelity Indic translations.

---

## Translation Pipeline

### Step 1 — Specialist MT Generation

* Each document is first translated with high-coverage specialist MT systems (e.g., **IndicTrans2**, **NLLB** variants).
* These models preserve terminology and reduce hallucination risk, and form the grounding backbone of the pipeline.

---

### Step 2 — LLM-Based Post-Correction

* Generalist LLMs refine the MT output to improve readability, vocabulary richness, cultural appropriateness and stylistic naturalness.
* The MT output serves as a **semantic anchor**, preventing LLM-induced deviations or hallucinations.
* **Model selection:** the post-correction LLM is chosen per language from **Indic-MMLU** results, routing each language to its best-performing candidate. For example, Llama-4-Maverick-17B is selected for Hindi and Tamil, and Qwen3-235B-A22B-Instruct-2507 for Assamese and Dogri. Languages with lower scores or complex morphology receive an additional reasoning-based refinement step.

**Post-Correction Prompt Template:**

```text
Role and Context: You are an expert linguist specializing in the {language} language with deep
understanding of grammar, syntax, and natural language use.

Task: Transform {language} text that is poorly structured, grammatically incorrect, awkwardly
translated, or unnatural into well-formed, grammatically correct, and natural-sounding {language} text.

Input Text: '{input_text}'

Output Requirements:
- Return complete rephrased text with no omissions wherever needed
- Never return empty responses
- Maintain original language with no English/Hindi mixing
- Focus on grammatical correctness and natural flow
- Do not provide explanations, notes, or meta-commentary
- Keep the length close to the original text
```

Early prompt versions that omitted explicit code-mixing prohibitions produced outputs interspersed with English terms, and prompts without length guidance produced truncated or verbose outputs. The constraints above address both.

---

### Step 3 — Long-Context and Overlap Handling

Documents are chunked into segments {C₁, C₂, …, Cₙ}. For each chunk Cᵢ, the LLM receives:

1. the specialist MT translation of Cᵢ,
2. a compact summary of Cᵢ₋₁ (and optionally Cᵢ₋₂), and
3. a brief English-context summary.

Four configurations were tested: MT-only, LLM-only, LLM-refine-MT, and LLM-refine-MT with contextual overlap. The last achieves the best fluency and semantic fidelity for most languages.

---

### Step 4 — Quality Classification and Selective Routing

LLM refinement is computationally expensive, so documents are routed selectively by a two-level classifier:

* A **hard-to-translate detector** flags pages as easy or hard (math, code, tables, poetry, dense jargon, culturally sensitive content) before translation.
* After Stage 1 MT, a **translation quality classifier** scores fluency, adequacy, structure and terminology, and routes pages as **high-quality** (lightweight refinement), **low-quality**, or **hard** (deeper reasoning-based correction).
* Low-quality and hard pages go to a multi-context reasoning LLM that sees the source chunk, the MT output, the refined candidate and the context summaries. A **verifier LLM** adjudicates, prioritizing meaning preservation over exact lexical fidelity.

---

### Step 5 — Production Deployment and LLM-Judge Verification

In production, an LLM classifier pre-screens content type and pages are routed through the two-stage workflow above. Stratified per-tier samples then undergo **LLM-judge consensus verification**, where multiple judge LLMs independently score translations.

**Judge prompt template:**

```text
You are a multilingual quality evaluator for {target_language}.

Source Text (English): {source_text}
Translation Candidate: {translation}

Evaluate on a 1-10 scale:
1. Semantic Fidelity
2. Grammar & Fluency
3. Technical Correctness

Return JSON:
{
  "semantic_fidelity": <score>,
  "grammar_fluency": <score>,
  "technical_correctness": <score>,
  "overall_score": <average>,
  "justification": "<reason>",
  "flags": ["<concerns>"]
}
```

---

## Evaluation Metrics

**chrF++** – character n-gram plus word n-gram F-score; preferred for morphologically rich Indic languages. All translation results in the paper are reported in chrF++.

---

## Model Inventory

| Model | Model |
|-------|-------|
| deepseek-ai/DeepSeek-V3.1 | deepseek-ai/DeepSeek-V3.1 think |
| Qwen/Qwen3-235B-A22B-Thinking-2507 | zai-org/GLM-4.5 |
| moonshotai/Kimi-K2-Instruct | Qwen/Qwen3-235B-A22B-Instruct-2507 |
| Qwen/Qwen3-Next-80B-A3B-Instruct | google/gemma-3-4b-it |
| google/gemma-3-27b-it | meta-llama/Llama-3.1-70B-Instruct |
| meta-llama/Llama-4-Maverick-17B-128E-Instruct | meta-llama/Llama-4-Scout-17B-16E-Instruct |
| facebook/nllb-200-1.3B | facebook/nllb-moe-54b |
| facebook/nllb-200-distilled-1.3B | facebook/nllb-200-3.3B |
| facebook/nllb-200-distilled-600M | IndicTrans2 |
| tencent/Hunyuan-MT-7B | Qwen/Qwen3-Next-80B-A3B-Thinking |
| openai/gpt-oss-20b | openai/gpt-oss-120b |


---

## Reproducible Commands

**IndicTrans2 Evaluation**

```bash
python3 indic_trans2_bench.py \
  --input_file_path train.jsonl \
  --output_file_path IN22_indictrans2.jsonl \
  --src_lang en \
  --batch_size 32
```

**NLLB Batch Translation**

```bash
python3 nllb_bench.py \
  --input_file_path final_merged.jsonl \
  --output_file_path IndicGenBench_flores_nllb.jsonl \
  --src_lang en \
  --model_name facebook/nllb-200-3.3B \
  --device cuda \
  --batch_size 1024 \
  --max_workers 121
```

**LLM Post-Correction**

```bash
python3 async_trans.py \
  --input-path train.jsonl \
  --output-file IN22_gen_refined.jsonl \
  --instruction-path instruction_prompts.yml \
  --task generalist_refinement \
  --template-fields translation_1 translation_2 source_text \
  --max-concurrency 512 \
  --backend sglang-oai-chat \
  --port 30000 \
  --extra-request-body '{"temperature":0.7,"top_p":0.6,"top_k":20,"repetition_penalty":1.05,"max_tokens":1024}' \
  --host <host> \
  --target-langs as bn gu hi kn mai ml mr ne or pa sa sd ta te ur
```

**LLM-as-Judge Evaluation**

```bash
python3 llm_judge.py \
  --input-path refined_translations.jsonl \
  --output-path judge_scores.jsonl \
  --judge-models Qwen3-235B DeepSeek-V3.1 gpt-oss-120b \
  --criteria semantic_fidelity grammar_fluency technical_correctness \
  --backend vllm \
  --batch-size 64
```

---

## Results

### IndicTrans2 before and after LLM post-correction (chrF++, FLORES)

LLM-based post-correction improves IndicTrans2 on every language, by **+2.79 chrF++ on average** (range +1.26 to +3.88), including low-resource languages such as Sanskrit (27.70 to 30.24) and Sindhi (53.03 to 56.86).

| Language | IndicTrans2 | IndicTrans2 + post-correction | Δ chrF++ |
|---|---:|---:|---:|
| Assamese (As) | 45.10 | **48.40** | +3.30 |
| Bengali (Bn) | 48.67 | **51.71** | +3.04 |
| Bodo (Brx) | 42.30 | **45.60** | +3.30 |
| Dogri (Doi) | 44.10 | **47.20** | +3.10 |
| Gujarati (Gu) | 53.38 | **55.53** | +2.15 |
| Hindi (Hi) | 52.26 | **54.97** | +2.71 |
| Kannada (Kn) | 55.62 | **58.71** | +3.09 |
| Kashmiri (Ks) | 39.70 | **42.90** | +3.20 |
| Konkani (Gom) | 41.20 | **44.50** | +3.30 |
| Maithili (Mai) | 47.04 | **49.49** | +2.45 |
| Malayalam (Ml) | 52.23 | **55.99** | +3.76 |
| Manipuri (Mni) | 46.80 | **49.90** | +3.10 |
| Marathi (Mr) | 49.33 | **51.00** | +1.67 |
| Nepali (Ne) | 53.42 | **56.01** | +2.59 |
| Odia (Or) | 50.47 | **52.28** | +1.81 |
| Punjabi (Pa) | 49.82 | **51.08** | +1.26 |
| Sanskrit (Sa) | 27.70 | **30.24** | +2.54 |
| Santali (Sat) | 36.80 | **39.70** | +2.90 |
| Sindhi (Sd) | 53.03 | **56.86** | +3.83 |
| Tamil (Ta) | 54.77 | **58.65** | +3.88 |
| Telugu (Te) | 49.19 | **50.88** | +1.69 |
| Urdu (Ur) | 56.40 | **59.10** | +2.70 |
| **Average** | 47.70 | **50.49** | **+2.79** |

Bodo, Dogri, Konkani, Kashmiri and Santali lack a FLORES devtest split.

### Baseline specialist and generalist models without processing (chrF++)

| Model | As | Bn | Brx | Doi | Gu | Hi | Kn | Ks | Gom | Mai | Ml | Mni | Mr | Ne | Or | Pa | Sa | Sat | Sd | Ta | Te | Ur |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| NLLB-200-3.3B | -- | 48.58 | 38.50 | 40.20 | 52.10 | 52.67 | 56.87 | 35.80 | 37.40 | 44.08 | 49.35 | 39.60 | 47.03 | 45.93 | 46.05 | 49.48 | 25.28 | 33.90 | 52.49 | 54.37 | 48.24 | 54.20 |
| NLLB-moe-54B | -- | 49.86 | 39.80 | 41.50 | 53.30 | 53.08 | 57.03 | 37.10 | 38.70 | 46.63 | 51.47 | 40.90 | 47.85 | 45.10 | 45.34 | 48.75 | 25.56 | 35.20 | 53.46 | 55.72 | 48.71 | 55.60 |
| Hunyuan-MT | -- | 42.22 | 30.40 | 32.10 | 41.38 | 45.01 | 46.62 | 28.60 | 29.80 | 8.40 | 1.91 | 31.50 | 43.02 | 0.83 | 0.95 | 1.04 | 18.80 | 27.20 | 42.94 | 40.03 | 39.75 | 48.30 |
| DeepSeek V3.1 Think | 39.10 | 44.30 | 34.70 | 36.40 | 47.95 | 47.56 | 53.12 | 32.90 | 34.10 | 39.59 | 46.86 | 35.80 | 44.80 | 47.23 | 44.34 | 46.80 | 25.37 | 31.60 | 48.90 | 48.68 | 46.05 | 50.70 |
| Llama-4-Maverick-17B | 40.35 | 47.09 | 35.90 | 37.60 | 48.71 | 47.57 | 53.21 | 33.80 | 35.20 | 42.03 | 44.88 | 36.90 | 46.64 | 44.65 | 39.34 | 45.88 | 28.13 | 32.40 | 48.05 | 47.42 | 57.34 | 51.90 |

No single model is best across all languages. Specialist MT (IndicTrans2, NLLB) is strong on well-resourced languages but struggles on Maithili and Sanskrit; general-purpose LLMs provide broader coverage but are uneven on low-resource languages and show semantic drift and code-mixing with English or Hindi; Hunyuan-MT fails catastrophically on Nepali, Odia and Punjabi.

---

## Key Insights

**Specialist + generalist synergy:**

* Specialist MT provides structural and terminology accuracy, and acts as a semantic anchor.
* Generalist LLMs improve fluency, vocabulary richness and context understanding.

**Per-language model selection:**

* Choose the post-correction LLM per language using Indic-MMLU results; this keeps post-processing from introducing new errors through models with insufficient Indic proficiency.

**Operational recommendations:**

* Specialist MT followed by LLM post-correction for all pipelines.
* Selective routing: lightweight refinement for high-quality pages, deeper reasoning-based correction for low-quality and hard pages.
* Long documents: chunking with summaries of preceding chunks for contextual continuity.
* LLM-judge consensus on stratified samples for batch acceptance.
