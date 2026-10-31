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
