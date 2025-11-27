# I-ADOPT LLM Service

### *Automatic Variable Decomposition and Semantic Linking*

This repository contains a Jupyter Notebook that demonstrates a simple, end-to-end workflow for converting a **natural-language variable definition** into structured **I-ADOPT components**, and then linking each component to **Wikidata**.

The workflow is easy to follow and designed as a reference implementation.

---

# 📘 What This Notebook Does

The notebook performs **two major steps**:

---

## 1️⃣ Phase 1 — Decomposition (LLM)

The model takes a natural-language definition such as:

> *“Mass concentration of isobutylene in chloroform.”*

…and breaks it into I-ADOPT elements:

* **hasProperty** → “mass concentration”
* **hasObjectOfInterest** → “isobutylene”
* **hasMatrix** → “chloroform”
* **hasConstraint** → (if present)

It uses:

* **`qwen/qwen3-32b`** (small, HPC-friendly)
* **`qwen/qwen3-235b-thinking`** (larger, more accurate)
* **5-shot prompting**
* **The `matrix_extender` prompt**, which contains improved rules for distinguishing matrices vs. constraints
* **Temperature = 0.5**

### Diagram: Phase 1

```
┌──────────────────────┐
│ Variable Definition  │
│ "Mass concentration…"│
└───────────┬──────────┘
            │
            ▼
  ┌────────────────────────┐
  │ LLM (Qwen models)      │
  │ + 5-shot examples      │
  │ + matrix_extender      │
  └───────────┬────────────┘
              │
              ▼
     ┌────────────────────┐
     │ I-ADOPT JSON       │
     │ { hasProperty,     │
     │   hasMatrix, … }   │
     └────────────────────┘
```

---

## 2️⃣ Phase 3 — Linking (Cross-Encoder)

For each extracted label (e.g., “isobutylene”), the system:

1. Searches Wikidata
2. Uses a **cross-encoder reranker** (`Qwen3-Reranker-0.6B`)
3. Selects a match **only if score ≥ 0.9**

This produces URIs such as:

* `https://www.wikidata.org/wiki/Q776976`
* `https://www.wikidata.org/wiki/Q589446`

### Diagram: Phase 3

```
┌────────────────────────────┐
│ Extracted labels from P1   │
│ ("isobutylene", "chloroform") 
└──────────────┬─────────────┘
               │
               ▼
     ┌───────────────────────┐
     │ Wikidata Search API   │
     └─────────┬─────────────┘
               │ candidates
               ▼
   ┌─────────────────────────────┐
   │ Cross-Encoder Re-Ranker     │
   │ (Qwen3-Reranker-0.6B)       │
   └──────────────┬──────────────┘
                  │ best score ≥ 0.9?
                  ▼
     ┌──────────────────────────┐
     │ Matched Wikidata URI     │
     └──────────────────────────┘
```

---

# 📦 Requirements

Use **Python 3.12.1**.

Install dependencies:

```bash
pip install httpx python-dotenv openai sentence_transformers requests-cache torch
```

Create a `.env` file:

```
OPENROUTER_API_KEY=your_key_here
```

---

# ▶️ How to Run

1. Clone the repository
2. Create .env file and add your api key to OPENROUTER_API_KEY=your_key_here
3. Open the notebook
4. Run all cells

The notebook will output:

* The decomposition JSON
* The Wikidata-linked JSON
* A wrapper function `process_variable()` to reuse the pipeline

---

# 🔮 Roadmap (What Comes Next)

We are currently extending the system beyond a simple prompt-based approach.

### ✔ Decision-Tree + Pattern Library (RAG)

* 100+ manually curated ground-truth variables
* Expert-defined **decomposition patterns**
* A decision tree that chooses the correct pattern
* A RAG (retrieval-augmented) system that inserts the pattern into the prompt

This will **greatly increase accuracy** and **reduce hallucinations**.

### ✔ Multi-Vocabulary Linking

Beyond Wikidata, users will be able to select:

* ENVO
* CHEBI
* SWEET
* AGROVOC
* and more…

The same cross-encoder ranking will choose the best concept across all vocabularies.

### ✔ Publication (Brief)

A paper describing this approach is being prepared for a
**Semantic Web Journal Special Issue: “Bridging Machine Learning and Knowledge Representation”**.

---

# 📄 Summary

This repository provides a clear, reproducible demonstration of:

* LLM-based I-ADOPT decomposition
* High-accuracy semantic linking via cross-encoders
* A foundation for future neuro-symbolic, pattern-guided decomposition

The notebook is intentionally simple and easy to follow, making it ideal for demos, workshops, and early prototyping.
