# Data Distillation & Persona-based Knowledge Rewriting

This folder accompanies the anonymous ACL ARR submission *MILA (Multilingual Indic Language Archive): A Dataset for Equitable Multilingual LLMs*. It demonstrates methods for **distilling knowledgeable data** using **Virtual Indian Personas** (the Indic PersonaHub); see Section 3.3.3 (Indic PersonaHub), Section 3.3.4 (Synthetic Augmentation and Rewriting) and Appendices C.7 and C.8 of the paper.

The Indic PersonaHub is a large-scale synthetic population of **over 200 million virtual Indian "citizens" spanning 1,400+ domains**, derived from Indian-language web crawls, regional literature and localized digital footprints. The approach leverages personas with diverse cultural and professional backgrounds to **extract, expand and rewrite domain-specific knowledge** in a more contextual, culturally grounded way.

---

## Methods for Persona Synthesis

Personas are generated with a dual-strategy protocol that maximizes sociodemographic coverage and relational depth.

### 1. Text-to-Persona

* A bottom-up strategy that captures the long tail of Indian society.
* Diverse texts (regional blogs, niche technical forums, village records) are processed by a reasoning-capable LLM, which infers detailed sociodemographic profiles of the people likely to write or engage with that content.
* Profiles are grounded in observed distributions, so that they align with real-world linguistic, occupational and cultural patterns and avoid mode collapse.

### 2. Persona-to-Persona

* Text-to-Persona may miss groups with **low online visibility** (e.g., the elderly, informal laborers, rural homemakers).
* Persona-to-Persona addresses this with a top-down relational expansion: high-confidence seed personas generate additional profiles via social-graph modeling, inferring plausible family, occupational and community connections into a cohesive virtual society.

### 3. Filtration and Refinement

* Raw profiles are mapped against a comprehensive **Indian Demographic Taxonomy** (state, language, profession, urban–rural) to monitor coverage and to trigger targeted generation for underrepresented subgroups.
* Semantic deduplication ensures that retained personas represent distinct viewpoints.

---

## Knowledge Distillation Workflow

Each Virtual Indian Persona has a **specific knowledge domain**. We use this structure to **ask thought-provoking questions** and obtain distilled, domain-rich knowledge.

**Steps:**

1. **Persona Population** – Synthesize diverse personas across knowledge domains (above).
2. **Task Assignment** – Assign each persona a task tailored to its expertise and background, framed within Indian contexts.
3. **Persona-Guided Generation** – LLMs generate responses consistent with the persona's characteristics.
4. **Translation** – Outputs pass through the hybrid MT→LLM translation pipeline to produce aligned translations across all 22 scheduled languages.
5. **Quality Judging** – Two LLM-based judges enforce quality: a **cultural compliance judge** verifies that persona behavior adheres to Indian cultural norms, and a **task relevance judge** evaluates whether the assigned task is appropriate for that persona.

Persona-conditioned generation is also used to diversify QA data: for a subset of text chunks, the question is posed through a selected persona profile, varying tone, intent, curiosity, formality and reasoning style while keeping facts grounded in the chunk.

> 💡 Personas can also be grouped by domain to perform **major knowledge distillation** (e.g., multiple personas in *biomedical ethics*).

Example prompt templates are available in **`dd_instruction.yml`**.

---

## Setup

* Use [SGLang](https://github.com/sglang) or [vLLM](https://github.com/vllm-project/vllm) to load the model locally.
* Run inference with asynchronous requests:

```bash
python async_infer.py --model <your_model_path> --input <input_file> --output <output_file>
```

---

## Example: Persona → Question → Distilled Knowledge

**Persona**

> A scientist who advocates for the ethical use of stem cells in research, particularly embryonic stem cells.
> Domain: *Stem Cell Biology & Ethics*

**Thought-Provoking Question**

> How should India navigate the ethical complexities of stem cell research and therapy, balancing the potential for groundbreaking medical advancements with moral considerations like embryonic stem cells, informed consent, and equitable access?

**Generated Knowledge**

* Detailed exploration of **ethical dilemmas**, **cultural values**, and **regulatory frameworks** in India.
* Emphasis on **informed consent**, **public healthcare access**, and **alternative research methods** (like iPSCs).
* Provides a **vision of India as a global leader** in ethical biomedical innovation.

(See full example in the `opensource-release/training-corpus/indian-english` directory.)

---

## Release

A representative PersonaHub subset with personas and generated data is released in this repository:

- [Indic PersonaHub](https://github.com/anonymous-submitter0104/acl-arr-submission/tree/main/opensource-release/indic-personahub)

---

## Applications

* Creating **knowledge-rich, culturally grounded synthetic data** that enriches the pretraining corpus with high-entropy tokens.
* Domain-specific **question answering** and **policy exploration**.
* Cultural and ethical grounding of scientific or social debates.

---

## Limitations

The cultural-compliance review is itself LLM-based and inherits the judges' own cultural priors. Persona-conditioned generation can over-weight majority viewpoints and formal registers relative to minority positions and concrete material concerns (along lines including gender, religion, caste and socio-economic status). Our cultural-grounding claim is therefore distributional (alignment with observed Indic sources) and not a claim of verified per-persona authenticity; we release the persona subset to enable external audit of these failure modes. See the Limitations section of the paper.
