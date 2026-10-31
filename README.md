# Applied Generative AI: Causal LM Fine-Tuning for Automated SOC Incident Briefing

**MSc Capstone — Project #5: Generative AI Applications**  
**Author:** Stanley Zeya  
**Dataset:** Public NSL-KDD Network Intrusion Benchmark Dataset (N = 1,250 records; 1,000 Train / 250 Validation)  
**Model Architecture:** Fine-Tuned DistilGPT2 (82 Million Parameters)  

---

## Executive Summary
This repository contains the end-to-end applied generative AI solution for **Project #5 (Generative AI Applications)**. The objective is to fine-tune a Causal Language Model (Causal LM) to translate raw, multi-dimensional network connection telemetry into structured, fluent, and operationally actionable natural language threat incident briefs.

To comply with rigorous capstone standards and project rubric requirements regarding **authentic, human-created data**, this project avoids synthetic AI prompts. All input telemetry attributes are sourced 100% from the publicly available **NSL-KDD Network Intrusion Benchmark Dataset** (hosted on the UCI Machine Learning Repository and Kaggle).

The dataset contains **N = 1,250 telemetry records**, partitioned into an 80% training set (**N_train = 1,000 records**, fulfilling and exceeding the capstone N >= 1,000 requirement) and a 20% validation set (**N_val = 250 records**).

---

## Repository Structure



---

## Data Pipeline & Telemetry Formatting

The input features 9 authentic network telemetry attributes extracted from DARPA packet recordings:
* **Protocol Type:** TCP, UDP, ICMP
* **Service Type:** HTTP, FTP, SMTP, Private
* **Attack Subtype & Category:** Neptune (DoS), Satan (Probe), Smurf (DoS), Warezclient (R2L), Normal (Benign)
* **Severity Level:** Critical, High, Informational
* **Error Ratios:** SYN Error Rate (), REJ Error Rate (), Connection Flag ()
* **Byte Transfers:** Source Bytes (), Destination Bytes ()

Records are formatted into dual-segment prompts:


---

## Fine-Tuning Performance & Diagnostics

The pre-trained  model (82M parameters) was fine-tuned for 5 epochs using AdamW (learning rate = 5e-5) on a CUDA GPU (17.72 seconds runtime).

### Training Loss & Perplexity Convergence:
* **Epoch 1:** Train Loss: 0.9959 | Val Loss: 0.2455 | Val PPL: 1.28
* **Epoch 2:** Train Loss: 0.2609 | Val Loss: 0.2130 | Val PPL: 1.24
* **Epoch 3:** Train Loss: 0.2287 | Val Loss: 0.2057 | Val PPL: 1.23
* **Epoch 4:** Train Loss: 0.2119 | Val Loss: 0.1972 | Val PPL: 1.22
* **Epoch 5 (Final):** Train Loss: 0.2018 | Val Loss: **0.1940** | Val PPL: **1.21**

---

## Decoding Strategy Benchmarks & Failure Mode Analysis

1. **Greedy Search (k=1):** High numerical precision, but vulnerable to repetitive tail loops.
2. **Temperature Sampling (T=0.7):** Softens distributions, but risks stochastic metric omission.
3. **Nucleus Sampling + Repetition Penalty (Top-P=0.9, Rep=1.2):** Optimal configuration—eliminates tail loops while maintaining strong metric fidelity.

---

## Quick Start & Reproduction Guide

### 1. Set Up Environment & Install Dependencies


### 2. Run Fine-Tuning Notebook


---

## References & Scholarly Citations
* **Holtzman, A., et al. (2019).** The curious case of neural text degeneration. *ICLR*.
* **Tavallaee, M., et al. (2009).** A detailed analysis of the KDD CUP 99 data set. *IEEE CISDA*.
* **Vaswani, A., et al. (2017).** Attention is all you need. *NeurIPS*.
