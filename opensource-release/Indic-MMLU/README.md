# Indic MMLU Data Generation

This repository provides scripts and methodologies for generating the Indic versions of the MMLU (Massive Multitask Language Understanding) benchmark. The process involves translating the original English MMLU dataset into multiple Indic languages, ensuring linguistic accuracy and cultural relevance.

## 🧪 Data Generation Pipeline

The data generation process encompasses the following steps:

### 1. **Translation**

* **Objective**: Translate the English MMLU dataset into selected Indic languages.
* **Method**: Utilize machine translation models trained on high-quality parallel corpora to ensure accurate and contextually appropriate translations.

### 2. **Quality Filtering**

* **Objective**: Ensure high-quality translations.
* **Method**: Implement quality metrics such as BLEU, chrF++, and TER to filter out low-quality translations based on predefined thresholds.

### 3. **Data Formatting**

* **Objective**: Prepare the dataset for model evaluation.
* **Method**: Format the translated data into standardized structures (e.g., JSON, CSV) compatible with evaluation frameworks.

### 4. **Evaluation**

* **Objective**: Assess model performance on the Indic MMLU dataset.
* **Method**: Use evaluation scripts to compute accuracy and other relevant metrics, comparing model predictions against the ground truth.


## ⚙️ Usage Instructions
# Example wrapper which runs evaluation for a model snapshot against the Indic MMLU dataset
```
  bash lm-eval-llm.sh /path/to/model_snapshot /benchmark-result-path
```



