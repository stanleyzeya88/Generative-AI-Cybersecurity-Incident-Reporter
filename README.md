# Enterprise Generative AI: Automated Cybersecurity Incident Report Generator

## 📌 Project Overview
Translating low-level network telemetry logs into professional, readable Security Operations Center (SOC) incident briefs using fine-tuned causal language models.

## 📊 Data Ingestion & Imbalance Management
- **Source Dataset:** NSL-KDD Network Intrusion Benchmark (Tavallaee et al., 2009).
- **Causal Model:** `distilgpt2` (82M parameters, BPE vocab 50,257).
- **Split Allocation:** 1,464 training samples | 296 validation samples.
- **Imbalance Mitigation:** Partition-first splitting with training-only oversampling (expanding Malware from 40 to 320 samples while keeping the validation partition unmodified).

## 📈 Training Diagnostics & Metrics
- **Epoch 1:** Train Loss 0.5986 | Val Loss 0.1764 | Val PPL 1.19
- **Epoch 5:** Train Loss 0.1242 | Val Loss 0.1372 | Val PPL 1.15

## ⚠️ Limitations & Generalization Boundaries
- **PPL 1.15 Artifact:** The low perplexity of 1.15 indicates rapid template memorization of our 5 category templates, not mathematical reasoning or telemetry generalization. Final validation loss (0.1372) exceeds training loss (0.1242), reflecting residual template memorization rather than true generalization.
- **Template Collapse:** Unconstrained stochastic decoding strategy leads to metric omissions and class template leakage under OOD inputs.
- **Recommended Mitigation:** Deploy a Hybrid Triage Pipeline with deterministic validation layers.
