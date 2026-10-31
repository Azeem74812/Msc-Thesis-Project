# Comparative Evaluation of Phishing and Malicious Email Detection Frameworks (2024–2026)

MSc Computing thesis project by **Azeem Mansha** (G00486661), Atlantic Technological University (ATU), Galway. Supervisor: Dr Owen Foley.

This repository holds the dissertation and the three student reimplementation notebooks. The work **compares existing methods**; it does not propose a new detector.

**Dissertation:** [Azeem_Mansha_Thesis.pdf](Azeem_Mansha_Thesis.pdf)

## What this project does

Three published email-detection approaches are reimplemented under controlled, documented settings and compared **within** each paper’s own dataset and protocol. Scores are **not** treated as one cross-paper leaderboard.

| Year | Approach | Notebook | Dataset / input |
|---|---|---|---|
| 2024 | TF-IDF + Naive Bayes, Random Forest, linear SVM, plus LIME | `Phishing_2024_vs_Baseline_Comparison.ipynb` | Merged Kaggle text corpus ([naserabdullahalam/phishing-email-dataset](https://www.kaggle.com/datasets/naserabdullahalam/phishing-email-dataset)) |
| 2025 | Baseline BERT vs a BERT + attention + CNN-1D + BiGRU hybrid | `Phishing_2025_vs_BERT_Complete_Comparison.ipynb` | Kaggle phishing emails ([subhajournal/phishingemails](https://www.kaggle.com/datasets/subhajournal/phishingemails)) |
| 2026 | PhishSigma++-style typed entity relations, PSO mask, linear SVM | `Phishing_2026_PhishSigma_vs_Baseline_Comparison.ipynb` | Raw RFC822 mail: Enron, Phishing Pot, Nazario, SpamAssassin |

## Source papers

1. Al-Subaiey et al. (2024). *Novel Interpretable and Robust Web-based AI Platform for Phishing Email Detection.* [doi:10.1016/j.compeleceng.2024.109625](https://doi.org/10.1016/j.compeleceng.2024.109625)
2. Hosseinzadeh et al. (2025). *Improving phishing email detection performance through deep learning with adaptive optimization.* [Nature Scientific Reports](https://www.nature.com/articles/s41598-025-20668-5)
3. Shang et al. (2026). *PhishSigma++: Malicious Email Detection with Typed Entity Relations.* [arXiv:2605.11619](https://arxiv.org/abs/2605.11619)

## Repository contents

```
Azeem_Mansha_Thesis.pdf
Phishing_2024_vs_Baseline_Comparison.ipynb
Phishing_2025_vs_BERT_Complete_Comparison.ipynb
Phishing_2026_PhishSigma_vs_Baseline_Comparison.ipynb
README.md
```

These notebooks are **transparent student reimplementations**, not the authors’ official code.

## How the experiments work

### 2024 — interpretable TF-IDF classifiers

- Clean text (lowercase, URL token, punctuation), drop empty/duplicate emails.
- Stratified **80/20** split, seed `42`, max **40,000** TF-IDF features.
- Train Multinomial NB, Random Forest (100 trees), and linear SVC on the same split.
- Compare against the paper’s Table 2 values, run McNemar (SVM vs NB), dump FN/FP, explain one prediction with **LIME**.
- Optional Word2Vec ablation is **off** by default.

### 2025 — BERT vs fixed hybrid

- Stratified **80/10/10** train/val/test split, seed `42`.
- Shared settings: `bert-base-uncased`, 256 tokens, 3 epochs, AdamW.
- **Baseline:** `AutoModelForSequenceClassification`.
- **Hybrid:** BERT `last_hidden_state` → BatchNorm → multi-head attention → Conv1D → BiGRU → masked mean-pool → dense. The paper mentions a pooled BERT vector; a single vector cannot feed CNN/GRU, so the notebook uses the token sequence and records that as an interpretation.
- Duplicate emails are **not** removed (`REMOVE_DUPLICATES = False`).
- Optional dropout/L2 ablations and **MGO** search are **off**. The hybrid result is a fixed-hyperparameter run, not an MGO-optimised model.
- Needs a GPU (Colab T4 is the intended runtime).

### 2026 — typed entity relations (PhishSigma++)

- Parse raw messages from Enron (benign, capped at 15k), Phishing Pot, Nazario, and SpamAssassin.
- Extract **40** entity types (headers, URLs, keywords) and **5** pairwise relations (exact, containment, LCS, Jaccard, length ratio).
- Collapse to a **1,600-d** feature vector.
- **PSO** (30 particles, **25** iterations; the paper uses 50) learns a sparse mask; fitness is inner 3-fold F1 minus a sparsity penalty.
- Final model: MaxAbsScaler + LinearSVC.
- Ablation: counts-only vs dense mask vs PSO mask.
- Baseline: TF-IDF + Naive Bayes on body text.
- **Good Word** body-padding robustness at ρ = 0.0, 0.2, 0.4, 0.6, 0.8 (headers/URLs left intact).
- Positive class is **malicious/spam**, not phishing-only. CPU is enough; feature extraction and PSO are slow.

## How to run

The notebooks target **Google Colab**.

1. Open a notebook in Colab.
2. **2024 / 2026:** CPU runtime. **2025:** Runtime → T4 GPU.
3. Run cells in order. Kaggle downloads use `kagglehub`; if that fails, place the CSV/raw files as the notebook instructs.
4. Results are written under `/content/phishing_*_project/results/`.

Local Python (3.9+) can run the same cells if you change `PROJECT_DIR` away from `/content/...` and install the packages listed in the first `pip` cell of each notebook.

## Reported student results (from the dissertation)

These numbers come from the executed Colab runs discussed in Chapter 5, not from a new run of this repo.

| Experiment | Main student result | Caution |
|---|---|---|
| 2024 linear SVC | F1 **0.9914** on 16,414 test emails | Merged corpus mixes spam/phishing labels |
| 2025 baseline BERT | F1 **0.9710** vs hybrid **0.9640** on 1,864 test emails | Duplicates were not removed before the split; MGO was not run |
| 2026 typed relations | F1 **0.9999** vs TF-IDF+NB **0.8833** | Near-perfect score is likely inflated by source/label confounding (Enron ≈ benign, Phishing Pot/Nazario ≈ malicious) |

Do not rank 2026 > 2024 > 2025 from these F1 values. The datasets, labels, and protocols are different.

## Known limitations (also stated in the thesis)

- Reproductions, not bit-exact replicas of unpublished author code.
- 2025 MGO and regularisation ablations are disabled.
- 2026 uses 25 PSO iterations (paper: 50), heuristic keyword lexicons, and a 15k Enron cap.
- 2026 SpamAssassin/Nazario parsing can fail or yield very few records.
- The 2026 “top evidence relations” mapping from LinearSVC weights back onto the PSO mask is not audited and may be incorrect.
- No shared raw-email benchmark is used for all three methods.

## License and use

Public academic thesis artefacts. Use the cited papers and datasets under their own terms. Do not send phishing content to real recipients; these experiments are offline only.
