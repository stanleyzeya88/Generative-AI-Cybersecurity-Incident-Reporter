# Generative AI Cybersecurity Incident Reporter

Fine-tuning a Causal Language Model (`distilgpt2`) for automated Security Operations Center (SOC) telemetry-to-text incident report generation.

## 📌 Project Overview
This repository implements an autoregressive sequence-to-sequence pipeline that transforms raw network intrusion telemetry into structured, analyst-ready security incident reports. Using pre-trained **DistilGPT-2 (82M parameters)**, the model learns to synthesize discrete protocol headers, severity levels, error rates, and attack classifications into actionable triage recommendations.

## 📊 Dataset & Class Balancing
The model is trained on authentic benchmark network telemetry from the **NSL-KDD Network Intrusion Dataset**:
- **Total Dataset Size:** \\(N = 1,931\\) sequences (\\(N_{\text{train}} = 1,600\\) | \\(N_{\text{val}} = 331\\))
- **Train/Val Split:** 80% / 20% split performed **before** oversampling to guarantee zero data leakage into validation metrics.
- **Class Balancing:** Minority attack categories (such as `Malware`) were oversampled in the training set to achieve an even distribution of 320 rows across all 5 classes (`DoS`, `Normal`, `Probe`, `Unauthorized_Access`, `Malware`).
- **Label Masking:** PyTorch `-100` label masking applied to prompt headers so cross-entropy loss is computed exclusively on target incident report tokens.

## ⚙️ Model Fine-Tuning Hyperparameters
- **Base Architecture:** `distilgpt2` (Causal LM, 82M parameters)
- **Optimizer:** `AdamW` (\\(\text{lr} = 5 \times 10^{-5}\\), \\(\epsilon = 1 \times 10^{-8}\\))
- **Batch Size:** 8 | **Epochs:** 5
- **Sequence Length:** `max_length = 180` tokens
- **Padding Token:** `pad_token = eos_token` (`<|endoftext|>`)

## 📈 Convergence Metrics
- **Initial Validation Loss (Epoch 1):** 0.0184 (Perplexity: 1.02)
- **Final Validation Loss (Epoch 5):** **0.0093** (Perplexity: **1.01**)
- **Training Time:** ~15.5 seconds on CUDA GPU

## 🧪 Qualitative Decoding Audit & Findings
We evaluated three decoding strategies on holdout telemetry prompts:
1. **Stochastic Sampling (\\(T = 0.7\\), Top-\\(P = 0.9\\)):** Generated natural language narratives but introduced occasional numerical parameter drift/hallucinations (e.g., writing "zero" or inaccurate error rates).
2. **Deterministic Greedy Search (\\(T = 0.0\\), `repetition_penalty = 1.0`):** Achieved **100% numerical metric fidelity**, accurately copying exact floating-point telemetry parameters (`Serror: 0.95`, `Serror: 0.10`) directly into the alert narrative without distortion.

### 🛡️ SOC Deployment Recommendation
For enterprise production environments, we recommend a **Hybrid Pipeline Architecture**:
- **Deterministic Filter / Extraction Layer:** Ensures 100% fidelity on raw IP addresses, error rates, and port metrics.
- **Fine-Tuned LLM Layer:** Generates fluent incident summaries and recommended analyst mitigation actions.

## 🚀 Repository Structure
```text
├── README.md                  # Project documentation
├── generative_model.ipynb     # Full 9-section Colab/Jupyter execution notebook
├── requirements.txt           # Python dependencies
├── .gitignore                 # Excludes local model weight binaries (>100MB)
└── data/                      # Sample telemetry data
```

## 📌 Project Overview
Translating low-level network telemetry logs into professional, readable Security Operations Center (SOC) incident briefs using fine-tuned causal language models.

## 📊 Data Ingestion & Imbalance Management
- **Source Dataset:** NSL-KDD Network Intrusion Benchmark (Tavallaee et al., 2009).
- **Causal Model:** `distilgpt2` (82M parameters, BPE vocab 50,257).
- **Split Allocation:** 1,464 training samples | 296 validation samples.
- **Imbalance Mitigation:** Partition-first splitting with training-only oversampling (expanding Malware from 40 to 320 samples while keeping the validation partition unmodified).

## 📈 Training Diagnostics & Metrics
- **Epoch 1:** Train Loss 0.5957 | Val Loss 0.1775 | Val PPL 1.19
- **Epoch 5:** Train Loss 0.1250 | Val Loss 0.1346 | Val PPL 1.14

## 🧪 Qualitative Decoding Audit & Findings
We evaluated three decoding strategies on holdout telemetry prompts:
1. **Decoding Uniformity:** Standard decoding configurations like Greedy Search, Temperature $0.7$, and Nucleus sampling yield **character-identical** outputs because the model assigns almost $1.0$ probability mass to the structured templates.
2. **Metric Slot Fidelity:** Serror metric copying is highly class-dependent, holding only for the DoS category (**1 out of 5** categories passed under greedy constraints).

## ⚠️ Limitations & Generalization Boundaries
- **PPL 1.14 Artifact:** The low perplexity of 1.14 indicates rapid template memorization of our 5 category templates, not mathematical reasoning or telemetry generalization. Final validation loss (0.1346) exceeds training loss (0.1250), reflecting residual template memorization rather than true generalization.
- **Template Collapse:** Unconstrained stochastic decoding strategy leads to metric omissions and class template leakage under OOD inputs.
- **Recommended Mitigation:** Deploy a Hybrid Triage Pipeline with deterministic validation layers.
"""
