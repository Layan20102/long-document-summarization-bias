# Extractive vs. Abstractive Summarization of Long Documents

**A Comparative Study with Demographic Bias Analysis** — on the BookSum long-document benchmark.

This repository accompanies our research paper comparing a **DistilBERT extractive baseline** against three **instruction-tuned LLMs** (LLaMA 3-8B, Mistral-7B, Qwen2-1.5B) on chapter-level literary summarization, and introduces a controlled framework for evaluating **demographic bias** under demographic prompt framing.

> IT 469 – Human Language Technologies · King Saud University

---

## TL;DR

- **Abstractive LLMs clearly beat the extractive baseline** across ROUGE and BERTScore. LLaMA 3-8B was best (ROUGE-1 = 0.3376, BERTScore F1 = 0.7763) vs. the DistilBERT baseline (ROUGE-1 = 0.1904).
- **No practically meaningful demographic bias.** Although the Wilcoxon test flagged some prompt conditions as statistically significant, all Cohen's *d* effect sizes were negligible (|d| < 0.2), with the largest being d = −0.129.
- Key takeaway: a clear distinction between **statistical** and **practical** significance in bias evaluation — large *n* (1,174 paired samples) makes tiny differences "significant."

---

## Motivation

Long-form literary summarization (full book chapters) stresses context retention, narrative coherence, and semantic compression. Extractive methods are efficient and interpretable but cannot paraphrase or restructure discourse, so they tend to fail on long narratives. We test this empirically and additionally probe whether LLM summaries shift in quality when the prompt implies the author has a particular **age, gender, or literacy level**.

---

## Dataset

- **BookSum** (Kryściński et al., 2022) — chapter–summary pairs from 8 literary-analysis sources (CliffsNotes, GradeSaver, Shmoop, SparkNotes, NovelGuide, PinkMonkey, TheBestNotes, BookWolf), 170 unique books; chapters sourced from Project Gutenberg.
- Loaded via the `kmfoda/booksum` HuggingFace mirror (BSD-3-Clause).
- After preprocessing: **7,990 train / 1,326 validation / 1,174 test**. All experiments run on the **1,174 test samples**.
- Median chapter length ≈ 2,934 words (mean ≈ 4,021); 96% of chapters exceed BERT's ~380-word limit, motivating a chunking strategy.

A noted methodological confound: reference summary length varies by source (GradeSaver ≈ 502 words vs. BookWolf ≈ 194), which affects ROUGE across all models.

---

## Method

### Preprocessing & Chunking
Removed aggregate/empty rows, applied Unicode normalization and whitespace/control-char cleaning, dropped chapters < 100 words or summaries < 20 words. Applied **sentence-aware hierarchical chunking** (350 words/chunk, 50-word overlap) that respects sentence boundaries.

### Models
| System | Type | Notes |
|---|---|---|
| DistilBERT (`distilbert-base-uncased`) | Extractive | Centroid cosine-similarity sentence selection; top-3 sentences, original order |
| Qwen2-1.5B-Instruct | Abstractive | Zero-shot, compact multilingual model |
| Mistral-7B-Instruct-v0.1 | Abstractive | Zero-shot, GQA + sliding-window attention |
| Meta-Llama-3-8B-Instruct | Abstractive | Zero-shot, strongest reasoning baseline |

LLMs were used **off-the-shelf** (no fine-tuning), prompted for ~200-word summaries with deterministic decoding (`do_sample=False`, `max_new_tokens=200`); inputs truncated to the first 3,000 characters.

### Bias Prompt Conditions (7)
`baseline` (no cue), `ageism_young`, `ageism_old`, `gender_male`, `gender_female`, `literacy_high`, `literacy_low` — each prepends a single framing sentence describing the author, keeping chapter content and instruction constant.

### Evaluation
ROUGE-1/2/L, BERTScore F1, and an extractive ratio. Statistical analysis: **Wilcoxon signed-rank test**, **Cohen's d** effect size, and **95% confidence intervals** — to separate statistical from practical significance.

---

## Results

### Overall performance (1,174 test samples, neutral prompt)

| Model | R-1 | R-2 | R-L | BERTScore F1 | Extr. Ratio |
|---|---|---|---|---|---|
| DistilBERT Extractive | 0.1904 | 0.0205 | 0.1055 | 0.7163 | 1.0000 |
| Qwen2-1.5B-Instruct | 0.2854 | 0.0441 | 0.1503 | 0.7630 | 0.0000 |
| Mistral-7B-Instruct | 0.3192 | 0.0577 | 0.1645 | 0.7747 | 0.0001 |
| **LLaMA 3-8B-Instruct** | **0.3376** | **0.0620** | **0.1683** | **0.7763** | 0.0000 |

The extractive baseline copies entire sentences (ratio = 1.0); LLM outputs show near-zero sentence-level overlap with references (a cautious proxy for abstractive behavior). Notably, Qwen2 beats the baseline despite having < 1/5 the parameters of the other LLMs.

### Bias analysis
ROUGE-1 spread across all 7 conditions is tiny — LLaMA 3: 0.0106, Mistral: 0.0060, Qwen2: 0.0046. Some Wilcoxon comparisons reached p < .05, but **every Cohen's d was negligible** (largest |d| = 0.129). Confidence intervals largely overlapped the baseline. Conclusion: current instruction-tuned LLMs are **robust to demographic framing** in long-document summarization.


---

## Limitations

- **Reference variance:** BookSum references differ in length by source, a confound for ROUGE across all models.
- **No fine-tuning:** zero-shot only; doesn't reflect each model's full potential on this task.
- **Prompt-level bias only:** manipulates a single framing sentence, not chapter content or adversarial prompts.
- **Metric coverage:** ROUGE/BERTScore don't capture factual consistency or narrative fidelity.
- **Single language:** English only; multilingual bias evaluation (esp. for Qwen2) is future work.

---

## Authors

Layan Alfawzan · Rose Mady · Jana Alruzuq · Layan Aldbays · Aljawharah Alwabel
King Saud University — *IT 469: Human Language Technologies*
