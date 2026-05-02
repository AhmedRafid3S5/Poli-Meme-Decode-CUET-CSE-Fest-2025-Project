## Click image to read full report
[![PDF Preview](Complete_pipeline.jpg)](Team_CopyPaste_Report.pdf)

# 🧠 PoliMemeDecode — Humor That Speaks Politics

> **CUET CSE Fest 2025 — Datathon** | Team CopyPaste

---

## ⭐ ⭐ 2nd Place out of 152+ Teams — CUET CSE Fest Datathon 2025 ⭐ ⭐

---

## 👥 Team Members

| Name | Kaggle Handle | Email |
|------|--------------|-------|
| Rumman Adib *(Leader)* | rummanadib | rummanadib@iut-dhaka.edu |
| Ahmed Rafid | ahmedrafid210041123 | ahmedrafid@iut-dhaka.edu |
| Muhammad Tausif Ul Islam | tausifulislam9786 | tausifulislam21@iut-dhaka.edu |
| Sameen Yeaser Adib | jwuptr | sameenyeaser@iut-dhaka.edu |

---

## 📌 Problem Statement

Classify social media memes into two categories:

- 🟥 **Political** — memes referencing political bodies, figures, parties, or conveying political satire
- 🟩 **Non-Political** — memes with no political context, relating to daily life or other topics

The challenge is particularly difficult due to multilingual content (Bengali + English), overlapping image-text relationships, and culturally specific political references.

---

## 📊 Dataset

| Split | Samples |
|-------|---------|
| Training (labeled) | 2,860 images |
| Test (unlabeled) | 330 images |

**Class imbalance ratio — NonPolitical : Political = 2.35 : 1**
- NonPolitical: 2,007 images
- Political: 853 images

Image resolutions range from `110×103` to `2048×2048` across both splits.

---

## 🔬 Methodology

### 1. 🤖 Model Selection
Evaluated multiple 4-bit quantized Vision Language Models (VLMs) via Unsloth:

| Model | Result |
|-------|--------|
| Qwen3 VL-8B ✅ | **Best performer — selected** |
| Qwen2.5 VL-7B | Runner-up |
| Gemma-3-4B | Lower accuracy |
| Qwen3 VL-32B | Didn't fit free-tier GPU |

**Base model:** `Qwen3-VL-8B-Instruct` (4-bit quantized) — chosen for its strong multilingual understanding.

### 2. 🛠️ Fine-Tuning
- **LoRA** with rank 16 and alpha 16 — only **0.50% of parameters trained** (43.6M / 8.8B)
- Vision layers **frozen** to prevent catastrophic forgetting
- Trained using **GRPO (Group Relative Policy Optimization)** — a reinforcement learning approach
- Custom **DISCO-based reward function** to address class imbalance:
  - Political (minority): weight `1.47`, reward `±2.94`
  - Non-Political (majority): weight `0.88`, reward `±1.76`
- Trained for **0.5 epochs (700 steps)**

### 3. 🔍 RAG-Augmented Inference
A two-pass inference pipeline to handle the model's weakness with recent/obscure Bangladeshi political references:

1. **Pass 1 — OCR:** The VLM extracts all visible text from the meme; Bengali text is translated to English
2. **Pass 2 — RAG Lookup:** Extracted text is matched against a political knowledge base using cosine similarity via `paraphrase-multilingual-MiniLM-L12-v2` (384-dim embeddings)
   - Top-10 entries above similarity threshold `0.3` are retrieved
   - Context is injected into the final classification prompt

**Benefits of RAG design:**
- Handles multilingual & fuzzy name matching (e.g. *"shaikh hasina"* → *"Sheikh Hasina"*)
- No retraining needed to add new political figures
- Interpretable similarity scores
- Separates entity detection from classification for better reliability

---

## 📈 Results

| Model | Public Macro F1 | Private Macro F1 |
|-------|----------------|-----------------|
| Base model (Qwen3 VL-8B, no fine-tuning) | 0.85895 | 0.78098 |
| **Fine-tuned + RAG Inference** | **0.88509** | **0.85274** |
| Improvement | +3.04% | **+9.19%** |

- 330/330 samples processed with **zero failures**
- Final predictions: 166 NonPolitical (50.30%) / 164 Political (49.70%)
- Results were **fully reproducible** across multiple inference runs

---

## 🧰 Tech Stack
- **Model:** Qwen3-VL-8B-Instruct (4-bit via Unsloth)
- **Training:** GRPO Trainer, LoRA, 8-bit AdamW
- **RAG:** `sentence-transformers/paraphrase-multilingual-MiniLM-L12-v2`, cosine similarity
- **Data:** Pandas, PIL
- **Platform:** Kaggle (free-tier GPU)

