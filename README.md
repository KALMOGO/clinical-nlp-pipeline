#  Clinical NLP Pipeline — Structured Data Extraction from Medical Notes

> End-to-end NLP/LLM pipeline that transforms unstructured French clinical notes into structured, research-ready data.

[![Python](https://img.shields.io/badge/Python-3.10+-blue)](https://python.org)
[![Streamlit](https://img.shields.io/badge/Streamlit-Demo-red)](https://streamlit.io)
[![HuggingFace](https://img.shields.io/badge/HuggingFace-Models-yellow)](https://huggingface.co)

---

##  Project Overview

This project addresses a critical challenge in healthcare AI: extracting structured information from free-text clinical notes written by medical staff.

**The pipeline automatically extracts:**
- Patient **comorbidities** → standardized ICD-10 codes
- Usual **treatments** → standardized ATC codes  
- **Lifestyle factors** → structured JSON (tobacco, alcohol, autonomy, sport...)

**Key constraint**: Fully on-premises deployment — no patient data sent to external APIs, compliant with European privacy requirements (GDPR).

---

##  Pipeline Architecture

```
Raw Clinical Text (French)
        │
        ▼
┌───────────────────────┐
│   1. ANONYMIZATION    │  CamemBERT-NER + Piranha PII model
│   Remove PII          │  → [NOM], [DATE], [ADRESSE]
└──────────┬────────────┘
           │
           ▼
┌───────────────────────┐
│   2. TEXT LABELLING   │  Local LLM (zero-shot or fine-tuned)
│   XML segmentation    │  → semantic XML tags
└──────────┬────────────┘
           │
           ▼
┌───────────────────────┐
│   3. EXTRACTION       │
│   ├── Comorbidities   │  → ICD-10 codes (LLM zero-shot)
│   ├── Treatments      │  → ATC codes (dictionary + fuzzy match)
│   └── Lifestyle       │  → Structured JSON (LLM zero-shot)
└──────────┬────────────┘
           │
           ▼
    Structured Database
    (comorbidities / treatments / lifestyle tables)
```

---

##  Dataset

>  **No data is included in this repository.** The pipeline was developed and evaluated on a real clinical dataset under strict privacy and ethical constraints (GDPR compliant).
> 
| Property | Value |
|----------|-------|
| **Volume** | ~14,000 clinical notes |
| **Time span** | ~12 years |
| **Mean note length** | ~390 words per note |
| **Language** | French (medical) |

---


Evaluated on a random sample of 500 annotations, using manual human annotation as ground truth.

### Comorbidity Extraction (ICD-10 Coding)

| Pipeline | Method | Accuracy | 95% CI |
|----------|--------|----------|--------|
| Pipeline 1 | Hybrid dictionary + medical embeddings | 0.740 | [0.700 ; 0.777] |
| Pipeline 2 | Pre-computed semantic embeddings | inconclusive | — |
| Pipeline 3 (no pre-processing) | LLM zero-shot | 0.886 | [0.855 ; 0.911] |
| **Pipeline 3 ★ (with pre-processing)** | **LLM zero-shot + abbreviation expansion** | **0.970** | **[0.955 ; 0.985]** |

> **Key finding**: Simple abbreviation expansion + temperature tuning improved accuracy from 88.6% to **97%** — outperforming the hybrid approach by +23 points.

### Usual Treatment Extraction (ATC Coding)

| Metric | Score | 95% CI |
|--------|-------|--------|
| Accuracy | **0.905** | [0.902 ; 0.907] |

### Lifestyle Factor Extraction

| Factor | Accuracy | 95% CI |
|--------|----------|--------|
| Tobacco use | **1.000** | [0.978 ; 1.000] |
| Tobacco quantity | **0.950** | [0.937 ; 0.974] |
| Alcohol use | **1.000** | [0.967 ; 1.000] |
| Autonomy | **0.940** | [0.874 ; 0.969] |
| Sport | **0.870** | [0.808 ; 0.954] |

---



| Component | Technology |
|-----------|-----------|
| **Anonymization** | CamemBERT-NER, Piranha PII (HuggingFace) |
| **Text Labelling** | Local LLM via LM Studio (Qwen 4B), Fine-tuned LLM (QLoRA) |
| **Comorbidity coding** | LLM zero-shot + medical abbreviation expansion → ICD-10-FR |
| **Treatment coding** | ATC dictionary + fuzzy matching + Unicode normalization |
| **Lifestyle extraction** | LLM zero-shot → structured JSON |
| **Medical NLP** | DrBERT (French medical BERT), MeSH thesaurus (Inserm) |
| **Web Demo** | Streamlit |
| **Frameworks** | HuggingFace Transformers, PyTorch, Scikit-learn |

---

##  Streamlit Demo

An interactive demo app was built to showcase the pipeline step by step.

**Workflow**: `Upload text → Anonymization → Extraction → Structured Results`

- **Step 1** — Upload a raw clinical note (.txt or .pdf)
- **Step 2** — Anonymization: visualize detected PII with confidence scores
- **Step 3** — Extraction: run the NLP pipeline on anonymized text
- **Step 4** — Results: browse structured tables, export CSV/JSON

> This is a local demo application. No real patient data is included in this repository.

---

## Installation & Usage

```bash

# Install dependencies
pip install -r requirements.txt

# Run the demo app
streamlit run app.py
```

> **Hardware**: GPU with 8GB+ VRAM recommended for LLM inference. CPU inference is supported but slower.

---



---

##  Key Technical Choices

**Why local LLMs instead of GPT-4/Claude API?**
Clinical notes contain sensitive patient data. On-premises inference with open-source models (Qwen, BioMistral) eliminates the risk of transmitting identifiable information to external services.

**Why XML for text labelling?**
XML markup is human-readable and naturally represents hierarchical clinical sections, supporting transparency and manual quality control by clinicians.

---

##  AI Usage

> This README was written with the assistance of Claude (Anthropic) for structure and English phrasing. The pipeline code and Streamlit application were developed by the author.

---

## Team

| Name | Role | 
|------|------|
| CUNGI Pierre-Julien, M.D | Principal Investigator, Clinical Expertise | 
| **KALMOGO Lucien (Lucien), MSc** | **Pipeline Development, Streamlit App** |
| MONDRAGON Osvaldo, Ing | Quality control and evaluation | 
| NDITANCHOU Rogers, M.D, PhD | Medical Validation | 
| OUATTARA Cheick Ahmed, M.D, MSc, PhD | Medical Validation|

---



##  License

This repository contains only the application code and pipeline architecture.
No clinical data, patient records, or institution-specific configurations are included.
