# Dataset Metadata — MLCommons Croissant

This repository includes five dataset metadata files in the [MLCommons Croissant](https://mlcommons.org/working-groups/data/croissant/) format, covering a suite of web-crawled, OCR, and synthetic Indic-language datasets.

---

## What is Croissant?

Croissant is a standardized, machine-readable metadata format for datasets. A single Croissant file captures everything a reader or tool needs to understand a dataset before touching the raw data:

- Name, description, and canonical URL
- Which files belong to the dataset and in what format
- Field-level schema for every record set
- How the data was produced (provenance)
- Licensing, known limitations, and responsible-use notes

Croissant is especially useful for research releases, dataset cards, automated dataset loaders, and reproducibility audits.

---

These files accompany the anonymous ACL ARR submission *MILA (Multilingual Indic Language Archive): A Dataset for Equitable Multilingual LLMs*. They describe the representative subsets available in this repository for reviewer access. Upon acceptance, the full open release (1T high-quality tokens across 22 scheduled Indic languages and English, plus the associated OCR, translation, persona and synthetic data) will be published with complete metadata.

## Datasets in this Repository

| File | Dataset | Type |
|---|---|---|
| `croissant_rai_Curated-Web-Crawl (1).json` | [Curated Web Crawl](https://huggingface.co/datasets/anonymous000023/Curated-Web-Crawl) | Multilingual web text |
| `croissant_rai_India-Centric_ImagendashText_Pairs.json` | [India-Centric Image–Text Pairs](https://github.com/anonymous-submitter0104/acl-arr-submission/tree/main/opensource-release/image-text-pairs) | OCR / vision-language |
| `croissant_rai_indic-persona-hub.json` | [Indic PersonaHub](https://github.com/anonymous-submitter0104/acl-arr-submission/tree/main/opensource-release/indic-personahub) | Synthetic persona corpus |
| `croissant_rai_indic-personahub-domain-qa-english.json` | [Indic PersonaHub Domain QA (English)](https://github.com/anonymous-submitter0104/acl-arr-submission/tree/main/opensource-release/training-corpus/indian-english) | Synthetic English QA |
| `croissant_rai_Synthetic-Indic-Multi-Lingual-Dataset-From-IndicPe (1).json` | [Synthetic Indic Multi-Lingual Dataset](https://github.com/anonymous-submitter0104/acl-arr-submission/tree/main/opensource-release/training-corpus/parallel-corpus) | Multilingual synthetic instructions |

### Curated Web Crawl

A multilingual web-crawl dataset curated through filtering, deduplication, and quality control. Key fields: `fasttext_score`, `id`, `language`, `language_score`, `text`, `url`, `nemo_id`, `quality_pred`.

### India-Centric Image–Text Pairs

Aligned image–text pairs from Indian-language documents, intended for OCR, document-VLMs, and document understanding. Organized as per-language Parquet files. Responsible-use notes cover uneven coverage, degraded scans, and research-only restrictions.

### Indic PersonaHub

A large-scale synthetic persona corpus with fields `id`, `persona`, and `domain`. Optimized for indexing, clustering, search, and retrieval workflows.

### Indic PersonaHub Domain QA (English)

An English-only synthetic QA corpus derived from Indic personas. Fields: `id` and `text`. Suited for instruction tuning, reasoning, and long-form generation tasks.

### Synthetic Indic Multi-Lingual Dataset

A multilingual synthetic instruction corpus generated from Indic PersonaHub across multiple language-specific Parquet files. Production pipeline included preprocessing, model-based filtration, and human evaluation.

---

## How to Use These Files

### Option 1 — Read directly as JSON

Croissant files are JSON-LD documents and open in any editor:

```bash
code croissant_rai_indic-persona-hub.json
```

Useful for inspecting the raw schema or checking a specific field definition.

### Option 2 — Validate with the Croissant CLI

```bash
pip install mlcroissant

python -m mlcroissant validate "croissant_rai_Curated-Web-Crawl (1).json"
```

Validation confirms that the metadata follows the Croissant specification, that file and column references are internally consistent, and that the dataset description is machine-readable. Recommended before any public or conference release.

### Option 3 — Load programmatically

```python
import json
from pathlib import Path

path = Path("croissant_rai_indic-personahub-domain-qa-english.json")
meta = json.loads(path.read_text())

print("Name:", meta.get("name"))
print("Description:", meta.get("description", "")[:300])
print("License:", meta.get("license"))

for rs in meta.get("recordSet", []):
    print("\nRecord set:", rs.get("@id", rs.get("name")))
    for field in rs.get("field", []):
        print(" •", field.get("name"), "→", field.get("dataType"))
```

### Option 4 — Extract distribution file names

```python
import json
from pathlib import Path

path = Path("croissant_rai_Synthetic-Indic-Multi-Lingual-Dataset-From-IndicPe (1).json")
meta = json.loads(path.read_text())

for dist in meta.get("distribution", []):
    print(dist.get("name"), dist.get("contentUrl"))
```

---

## Reading a Croissant File

Work through sections in this order for the clearest picture:

**1. `name`, `description`, `url`** — what the dataset is and where it lives.

**2. `distribution`** — which files belong to the dataset, their formats (e.g., Parquet), URL patterns, and checksums (MD5 / SHA256).

**3. `recordSet`** — the core schema: record-set names, field names, data types, and source mappings. This is where you learn what a row actually contains.

**4. `prov:wasDerivedFrom` / `prov:wasGeneratedBy`** — provenance: original sources, processing steps (filtering, OCR, model generation, human review).

**5. `rai:*` fields** — responsible-AI metadata: limitations, known biases, intended use, social impact, and synthetic-data indicators.

---

## Insights You Can Extract

**Schema** — number of record sets, field names and types, modality (text / image-text / tabular / synthetic QA), language or domain coverage.

**Provenance** — whether the data is web-derived, OCR-derived, synthetically generated, or mixed; whether deduplication, toxicity filtering, or decontamination was applied.

**Coverage** — number of language subsets, presence of domain labels, dataset balance, release completeness.

**Responsible use** — intended applications, known limitations, possible bias sources, suitability for high-stakes deployment.

---

## For Reviewers

Each Croissant file is a self-contained dataset blueprint. To get the fastest overview:

1. Open the file in any JSON viewer.
2. Check `distribution` for files and formats.
3. Check `recordSet` for fields and schema.
4. Check `rai:*` for responsible-use notes and limitations.

Alternatively, run `mlcroissant validate` on any file to confirm machine-readability in under a minute.

