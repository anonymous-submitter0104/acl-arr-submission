# OCR Pipeline

This folder accompanies the anonymous ACL ARR submission *MILA (Multilingual Indic Language Archive): A Dataset for Equitable Multilingual LLMs*. It documents the Indic OCR pipeline and its evaluation; see Section 3.3.1 (OCR Pipeline) and Appendices C.4 and C.5 of the paper.

## 📚 Table of Contents

* [Pipeline Overview](#pipeline-overview)
* [Multi-Stage OCR Processing](#multi-stage-ocr-processing)

  * [Pre-Processing](#pre-processing)

    * [Error Identification using VLMs](#error-identification-using-vlms)
    * [Artifact Removal & Enhancement](#artifact-removal--enhancement)
  * [Stage 1: OCR / Parsing](#stage-1-ocr--parsing)
  * [Stage 2: Post-Correction](#stage-2-post-correction)
  * [Final LLM-Judge Verification](#final-llm-judge-verification)
  * [Supplementary: Reconstruction-Based Validation](#supplementary-reconstruction-based-validation)
* [Release](#release)
* [Evaluation](#evaluation)

  * [Metrics](#metrics)
  * [LLM-Assisted Quality Evaluation](#llm-assisted-quality-evaluation)
  * [Experimental Results](#experimental-results)

    * [Without Preprocessing and Postprocessing](#without-preprocessing-and-postprocessing)
    * [Word Error Rate (WER) Evaluation Across Languages](#word-error-rate-wer-evaluation-across-languages)
    * [With Preprocessing and Postprocessing](#with-preprocessing-and-postprocessing)
* [Key Findings](#key-findings)
* [Ablation Experiment: Conventional OCR vs Post-Corrected OCR](#ablation-experiment-conventional-ocr-vs-post-corrected-ocr)

  * [Observations](#observations)
* [Model Architecture (310M Parameters)](#model-architecture-310m-parameters)
* [Validation Loss Comparison](#validation-loss-comparison)

---

# Pipeline Overview

High-quality OCR is a prerequisite for native vocabularies and reliable pretraining corpora. Diverse script families, complex ligatures, heterogeneous typography and degraded scans make Indic OCR significantly more challenging than Latin-script OCR. We therefore use a **two-stage, language-specific OCR pipeline** that combines scalable LLM/VLM-based assessment with frontier models selected through our own benchmarking, and post-correction via high-capacity LLMs.

For each language the pipeline consists of **(i) OCR / parsing** and **(ii) post-correction**, preceded by a unified **pre-processing** module. Layout (block structure and reading order) is preserved throughout, as this improves contextual grounding. For every language we evaluate a pool of state-of-the-art OCR and VLM models on public Indic OCR datasets and on the Indic Synthetic OCR Benchmark (ISOB), yielding a shortlist of top-performing candidates.

**Input:** Digitized or undigitized document images (low-quality, noisy or artifact-rich)
**Output:** Validated OCR text

---

# Multi-Stage OCR Processing

## Pre-Processing

An upfront orientation and noise pre-processing pass is applied before OCR.

### Error Identification using VLMs

**Model:** Lightweight VLM (e.g., Qwen-VL-7B)

**Task:** Detect artifacts, orientation, and readability issues in scanned pages.

**Prompt (VLM):**

```text
You are a document analysis system.  
Given this scanned page, identify the following issues:  
1. Page orientation (normal, rotated, upside-down).  
2. Presence of noise, blur, or watermarks.  
3. Regions of non-text (stamps, illustrations, smudges).  
4. Overall readability score (High, Medium, Low).  

Return the issues as a structured JSON object.
```

---

### Artifact Removal & Enhancement

**Techniques:**

* Orientation correction, noise/blur removal
* Super-Resolution (SRGAN) for low-resolution pages
* Qwen Image Edit for severe degradation

**Prompt (Qwen Image Edit):**

```text
You are an image enhancement system.  
Task: Improve readability of this scanned document.  

Instructions:  
- Sharpen text edges.  
- Remove background noise and smudges.  
- Correct orientation if tilted.  
- Increase resolution while preserving textual structure.  

Return the enhanced page without altering the content.
```

---

## Stage 1: OCR / Parsing

**Approach:**

* A representative page sample is constructed from crawled books and partner-sourced documents, spanning "easy to hard" difficulty as classified by a strong VLM (e.g., Qwen-VL-32B) using visual cues.
* Each page is processed by the **top-*k* candidate OCR/VLM models** for the language.
* Outputs are scored under a **VLM-LLM-as-Judge framework** built on frontier models selected through our Indic benchmarking (see the Indic MMLU folder): a reasoning-capable VLM produces chain-of-thought evaluations on **native-word preservation, spelling accuracy and completeness**, independently verified by a second LLM.
* Scores are aggregated to select **one consensus model per language**.

**Prompt (Generalist VLM for layout):**

```text
You are an OCR layout assistant.  
Given this scanned page, output:  
1. The document layout (columns, tables, figures).  
2. Logical reading order of text blocks.  
3. Any script/language hints detected.  

Return in structured JSON to assist OCR alignment.
```

---

## Stage 2: Post-Correction

Even state-of-the-art OCR/VLM systems exhibit minor but systematic errors (spelling inconsistencies, missing graphemes, mis-segmentation, low-frequency artifacts) that accumulate at scale and are costly to correct manually.

* A post-correction LLM is selected via a **language-specific benchmark suite** covering contextual fidelity, native fluency, factual alignment and hallucination resistance.
* The engine operates at the **page level**, consuming the raw OCR output, other top candidates' outputs, neighboring-page summaries and the VLM's reasoning traces.
* **Hard pages** invoke full reasoning-based correction; **high-quality pages** get a lightweight pass. The correction prioritizes meaning preservation over exact lexical fidelity.
* Rule-based filters (dictionary constraints, script normalization) complement the LLM correction.

**Prompt (LLM Post-Correction):**

```text
You are an expert in correcting OCR output for printed text.  
The OCR output may contain:
- Misrecognized characters (e.g., language-specific confusions)
- Broken, merged, or incomplete words
- Incorrect line or paragraph order due to multi-column layouts
- Stray punctuation, symbols, or other OCR artifacts

Your task is to:
1. Correct all character-level and word-level OCR errors.  
2. Restore proper reading order and logical sentence/paragraph flow.  
3. Remove meaningless OCR noise while preserving all valid content.  
4. Keep the text in the original script.  
5. Output only clean, coherent text in proper writing.

Now correct this OCR output accordingly:
```

---

## Final LLM-Judge Verification

In production, a final verification step evaluates a **stratified sample from all quality tiers** for cultural and contextual sensitivity, tone, factual alignment and hallucination. **Consensus among the judge ensemble** determines batch acceptance.

---

## Supplementary: Reconstruction-Based Validation

_Such efforts are especially needed when dealing with extremely low-resource language corpora._

These scripts provide an additional, optional consistency check by reconstructing a page from its hOCR and comparing it with the original.

### Reconstruction with hOCR + Style Transfer

**Prompt (Qwen Image Edit - Style Transfer):**

```text
You are an image reconstruction system.  
Task: Using the provided hOCR, generate an image that visually resembles the original scanned document.  

Constraints:  
- Preserve layout, fonts, and formatting.  
- Apply natural degradation styles common in manuscripts (faded ink, paper texture).  

Return the synthetic page.
```

### Embedding Similarity Check

* Compute cosine similarity between embeddings of original vs reconstructed images.

### Reasoning-Based Validation with VLMs

**Prompt (Reasoning VLM):**

```text
You are a reasoning OCR evaluator.  
Given this scanned page, describe step by step:  
1. What the page contains (title, paragraphs, tables, etc.).  
2. The key semantic content (entities, topics, structure).  

Return a reasoning trajectory of what you understand from this page.
```

### Trajectory Comparison with LLMs

**Prompt (Text LLM Comparison):**

```text
You are a similarity evaluator.  
Input: Two reasoning trajectories of the same page (original vs reconstructed).  

Task:  
1. Compare their semantic overlap.  
2. Assign a similarity score (0–100).  
3. Provide justification for the score.  
4. Bucket the result into {Good, Acceptable, Bad}.  

Return output as JSON: {score, justification, bucket}.
```

### Score-Reason Consistency Check

**Prompt (Consistency Check LLM):**

```text
You are a validation assistant.  
Input: {score, justification} from similarity evaluator.  

Task: Verify if the justification logically supports the score.  
Return either "Consistent" or "Inconsistent".
```

---

All model interactions are guided by instruction-grade prompts for reproducibility.

---

## Release

* **Image–text pairs:** a large collection of image–text pairs from Indian-language documents is released in this repository for OCR and Document-VLM research.

  - [Image-Text Pairs Dataset](https://github.com/anonymous-submitter0104/acl-arr-submission/tree/main/opensource-release/image-text-pairs)

* **OCR-extracted text:** the pipeline is run across all 22 scheduled languages during corpus construction, but public release of the resulting OCR-extracted text is restricted to **10 languages** (Assamese, Bengali, Hindi, Kannada, Malayalam, Marathi, Sanskrit, Tamil, Telugu and Urdu) because of licensing restrictions on the underlying scanned sources.

* **ISOB:** the synthetic OCR benchmark is unaffected by this restriction and is released for all 22 languages (see the ISOB folder).

> Please cite the accompanying paper if you use the dataset in your research.

---

## Evaluation

### Metrics

- Character Error Rate (CER)
- Word Error Rate (WER)
- Position-Independent WER (PI-WER)
- Char3-gram F1

### LLM-Assisted Quality Evaluation

Conventional OCR metrics measure surface-level string similarity, but they fail to capture whether post-processing (sentence reordering for coherence, ligature repair that changes character sequences, contextual corrections that substitute semantically equivalent terms) preserves or enhances meaning. This is especially acute for quality-enhanced text, where deliberate modifications may increase string distance from the raw OCR while improving linguistic fidelity.

To validate that improvements in traditional metrics translate to better semantic quality, we use LLMs as quality judges: GPT-OSS-120B, DeepSeek and Qwen are prompted to compare the ground truth with both raw OCR and post-processed outputs, producing consistency scores that reflect semantic preservation independent of surface form.

### Experimental Results

#### Without Preprocessing and Postprocessing

Results on two existing Indian OCR benchmarks, Bhashini and Mozhi:

| Model                                    | Bhashini CER | Bhashini WER | Bhashini PI-WER | Bhashini Char3 F1 | Mozhi CER | Mozhi WER | Mozhi PI-WER | Mozhi Char3 F1 |
|------------------------------------------|-------------:|-------------:|----------------:|------------------:|----------:|----------:|-------------:|---------------:|
| DotsOCR                                  | 0.168        | 0.253        | 0.23            | 0.801             | 0.12      | 0.19      | 0.9          | 0.88           |
| Surya                                    | 0.2          | 0.28         | 0.138           | 0.867             | 0.14      | 0.21      | 0.91         | 0.89           |
| Llama-4-Scout-17B-16E-Instruct           | 0.259        | 0.445        | 0.398           | 0.672             | 4.35      | 1.38      | 0.619        | 0.31           |
| NuMarkdown-8B-Thinking                   | 0.361        | 0.537        | 0.508           | 0.556             | 53.31     | 9.21      | 0.677        | 0.168          |
| Llama-4-Maverick-17B-128E-Instruct       | 0.4          | 0.58         | 0.418           | 0.645             | 12        | 4         | 0.72         | 0.22           |
| Qwen2.5-VL-72B-Instruct                  | 0.676        | 0.847        | 0.45            | 0.613             | 18.22     | 4.16      | 0.677        | 0.266          |
| SmolDocling-256M-preview                 | 1.235        | 1.4          | 0.988           | 0.016             | 137.66    | 55.57     | 0.946        | 0.0001         |
| RolmOCR                                  | 1.938        | 2.019        | 0.498           | 0.552             | 986.91    | 263.08    | 0.692        | 0.111          |
| olmOCR-7B-0825                           | 2.068        | 1.842        | 0.516           | 0.531             | 28.53     | 6.48      | 0.704        | 0.126          |
| Nanonets-OCR-s                           | 3.573        | 2.318        | 0.568           | 0.471             | 305.27    | 42.57     | 0.685        | 0.161          |
| GLM-4.1V-9B-Thinking                     | 4.384        | 3.88         | 0.893           | 0.08              | 755.1     | 321.92    | 0.985        | 0.0001         |
| MinerU2.5-2509-1.2B                      | 5.176        | 3.214        | 0.906           | 0.095             | 180       | 55        | 0.91         | 0.1            |
| pixtral-12B                              | 5.847        | 4.86         | 0.893           | 0.163             | 1.47      | 0.999     | 0.941        | 0.0039         |
| InternVL3_5-GPT-OSS-20B-A4B-Preview-HF   | 38.87        | 4.537        | 0.994           | 0.0029            | 195.85    | 240.2     | 0.919        | 0              |


#### Word Error Rate (WER) Evaluation Across Languages

<table>
  <tr>
    <td align="center"><img src="assets/assamese.png" width="170"/><br/>assamese</td>
    <td align="center"><img src="assets/bengali.png" width="170"/><br/>bengali</td>
    <td align="center"><img src="assets/bodo.png" width="170"/><br/>bodo</td>
    <td align="center"><img src="assets/dogri.png" width="170"/><br/>dogri</td>
    <td align="center"><img src="assets/english.png" width="170"/><br/>english</td>
  </tr>
  <tr>
    <td align="center"><img src="assets/gujarati.png" width="170"/><br/>gujarati</td>
    <td align="center"><img src="assets/hindi.png" width="170"/><br/>hindi</td>
    <td align="center"><img src="assets/kannada.png" width="170"/><br/>kannada</td>
    <td align="center"><img src="assets/kashmiri.png" width="170"/><br/>kashmiri</td>
    <td align="center"><img src="assets/konkani.png" width="170"/><br/>konkani</td>
  </tr>
  <tr>
    <td align="center"><img src="assets/maithili.png" width="170"/><br/>maithili</td>
    <td align="center"><img src="assets/malayalam.png" width="170"/><br/>malayalam</td>
    <td align="center"><img src="assets/manipuri.png" width="170"/><br/>manipuri</td>
    <td align="center"><img src="assets/marathi.png" width="170"/><br/>marathi</td>
    <td align="center"><img src="assets/nepali.png" width="170"/><br/>nepali</td>
  </tr>
  <tr>
    <td align="center"><img src="assets/punjabi.png" width="170"/><br/>punjabi</td>
    <td align="center"><img src="assets/sanskrit.png" width="170"/><br/>sanskrit</td>
    <td align="center"><img src="assets/sindhi.png" width="170"/><br/>sindhi</td>
    <td align="center"><img src="assets/tamil.png" width="170"/><br/>tamil</td>
    <td align="center"><img src="assets/telugu.png" width="170"/><br/>telugu</td>
  </tr>
  <tr>
    <td align="center"><img src="assets/urdu.png" width="170"/><br/>urdu</td>
  </tr>
</table>


#### With Preprocessing and Postprocessing

Bhashini, before and after language-specialized post-correction (dictionary-based correction, language-model rescoring, ligature repair and Unicode normalization):

| Model                    | CER   | WER   | PI-WER | Char3 F1 |
|--------------------------|------:|------:|-------:|---------:|
| Dots.OCR                 | 0.168 | 0.253 | 0.23   | 0.801    |
| Dots.OCR - postcorrected | 0.085 | 0.145 | 0.12   | 0.91     |
| Surya                    | 0.2   | 0.28  | 0.138  | 0.867    |
| Surya - postcorrected    | 0.095 | 0.16  | 0.11   | 0.925    |


### Key Findings

- Vision-language models cover Indic scripts but hallucinate extra text, inflating CER and WER (often beyond 1) on these benchmarks.
- Specialized OCR models such as DotsOCR and Surya achieve the lowest error rates, and improve further after post-correction.
- Post-correction roughly halves CER (Dots.OCR 0.168 to 0.085; Surya 0.2 to 0.095) and cuts WER by about 40–45% (0.253 to 0.145; 0.28 to 0.16).
- Pretraining on processed OCR text yields smoother convergence than raw OCR.

## Ablation Experiment: Conventional OCR vs Post-Corrected OCR

We demonstrate the practical impact of OCR quality on downstream model training through controlled experiments on a **310M-parameter dense language model**. Two versions of the same corpus were compared:

1. **Raw OCR output** with typical error rates around **15–20%**.
2. **Quality-enhanced text** after our full post-processing pipeline.

### Observations

- **Raw OCR Data**:
  - Produced unstable loss curves with frequent spikes.
  - Showed irregular perplexity behavior and slow convergence, indicating corpus noise overwhelming the learning signal.

- **Post-processed OCR Data**:
  - Achieved smooth, monotonic loss reduction.
  - Stable perplexity curves, with markedly lower final perplexity.
  - Superior performance on Indic benchmarks.

These results show that careful preprocessing and post-correction directly improve model capability. In low-resource settings, noisy OCR exacerbates data scarcity, whereas robust OCR and post-processing transform limited corpora into high-impact training material.

---

### Model Architecture (310M Parameters)

| **Attribute**                  | **Value**                       |
|--------------------------------|---------------------------------|
| Model Architecture             | causal language model           |
| Hidden size                    | 768                             |
| Intermediate size (FFN)        | 3108                            |
| Max Position Embeddings        | 2048                            |
| Num of Attention Heads         | 12                              |
| Num of Hidden Layers           | 12                              |
| Num of Query Groups            | 12                              |
| Normalization                  | RMSNorm                         |
| Activation Function            | swiglu                          |
| Attention Type                 | Multi-head Attention            |
| Position Embedding Type        | RoPE (rotary)                   |
| Dropout (hidden/attn/ffn)      | 0.0 / 0.0 / 0.0                 |
| Precision                      | bf16 (AMP O2)                   |

---

### Validation Loss Comparison

![Validation loss: conventional vs post-corrected OCR](assets/val-loss-ocr.png)

The post-corrected corpus produces **substantially more stable training dynamics** and **lower final perplexity**, demonstrating the direct impact of OCR quality on model performance.

---

Beyond MILA, the infrastructure and validation framework established here provide reusable tools for equitable multilingual model development.
