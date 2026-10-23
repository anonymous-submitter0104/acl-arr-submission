# MILA Open-Source Release

This folder contains the open-source artifacts released with the anonymous ACL ARR submission *MILA (Multilingual Indic Language Archive): A Dataset for Equitable Multilingual LLMs*. Each subfolder has its own README describing its exact contents, structure and usage.

---

## 📂 Released Artifacts

1. **Indic MMLU**
   A multilingual adaptation of the MMLU test set covering the **22 scheduled Indian languages and English**, used to evaluate LLMs on Indic languages, measure cross-lingual knowledge transfer, and select models for the LLM-as-a-judge roles in the MILA pipelines.

2. **Image–Text Pairs**
   A collection of aligned image–text pairs from Indian-language documents, for Indic OCR and Document-VLM research.

3. **Indic Persona Hub**
   Virtual Indian personas grounded in an Indian demographic taxonomy, together with persona-generated data. A representative PersonaHub subset with personas and generated data is released.

4. **ISOB-Small**
   The Indic Synthetic OCR Benchmark: 110 synthetically generated pages spanning 22 Indian languages, for evaluating OCR systems on hard-to-OCR Indic documents.

5. **Training Corpus**
   Text from the MILA corpus: **Indian English**, **parallel (translated) corpora** and **web-crawled text**, curated with deduplication, toxicity filtering, PII redaction, language identification, quality filtering and benchmark decontamination.

---

## ⚖️ Licensing & Responsible Use

* **Licensing governance:** every source in MILA is tagged with licensing metadata and provenance. Items with restrictive or ambiguous licenses are excluded from the open release.
* **OCR-extracted text:** public release of OCR-extracted text is limited to 10 of the 22 languages (Assamese, Bengali, Hindi, Kannada, Malayalam, Marathi, Sanskrit, Tamil, Telugu and Urdu) because of licensing restrictions on the underlying scanned sources. ISOB is synthetic and is released for all 22 languages.
* **PII:** multilingual PII detection and redaction is applied during curation.
* **Toxicity:** a multilingual toxicity filter is applied during curation. As with any web-scale corpus, residual harmful or sensitive content may remain.
* **Intended use:** the artifacts are released for academic and research use.
