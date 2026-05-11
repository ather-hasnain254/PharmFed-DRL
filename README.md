# PharmFed-DRL

**Privacy-Preserving Harmonised Adaptive Multi-modal Reinforced Federated Drug Response Learning**

[![Python 3.9+](https://img.shields.io/badge/python-3.9+-blue.svg)](https://www.python.org/)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![Framework: NumPy](https://img.shields.io/badge/framework-NumPy-orange.svg)](https://numpy.org/)

---

## Overview

PharmFed-DRL is a privacy-preserving multi-institutional framework for cancer drug sensitivity prediction. It integrates:

- **Federated Learning (FedAvg)** — collaborative training across hospital institutions without sharing raw patient data
- **Pathway-level Feature Engineering** — ssGSEA and Random Walk with Restart on 38 cancer driver pathways
- **Dual Autoencoders** — separate encoders for somatic mutations and drug Morgan fingerprints
- **Medical IoT (MIoT) Integration** — cross-attention fusion of 14-channel physiological signals
- **Deep Reinforcement Learning (PPO)** — intelligent client selection and treatment-policy optimisation
- **Integrated Gradients** — biologically interpretable pathway-level attributions

Benchmarked against **DeepCDR** (Liu et al., *Bioinformatics* 2020), **DualGCN** (Ma et al., *BMC Bioinformatics* 2022), and **DrugCell** (Kuenzi et al., *Cancer Cell* 2020).

---

## Project Structure

```
pharmfed/
├── README.md                    ← This file
├── requirements.txt             ← Python dependencies
├── config.py                    ← Global configuration & hyperparameters
├── main.py                      ← Entry point — runs full pipeline
│
├── data/
│   ├── __init__.py
│   └── dataset.py               ← Synthetic data generator + real data loader
│                                   (auto-switches when real CSVs are present)
│
├── models/
│   ├── __init__.py
│   ├── model.py                 ← PharmFed-DRL architecture
│   │                               (autoencoders + cross-attention + PPO agent)
│   └── trainer.py               ← Training engine
│                                   (FedAvg + PPO + k-fold CV + metrics)
│
├── experiments/
│   ├── __init__.py
│   └── figures.py               ← All 10 publication-quality figures
│
└── results/
    ├── figures/                 ← Generated PNG figures (auto-created)
    └── tables/                  ← Generated CSV tables (auto-created)
```

---

## Quick Start

### 1. Install dependencies

```bash
pip install -r requirements.txt
```

### 2. Run with synthetic data (no download required)

```bash
python main.py
```

### 3. Quick run (fewer rounds, for testing)

```bash
python main.py --rounds 10 --runs 2
```

### 4. Run with real data (after downloading — see Dataset section)

```bash
python main.py --real
```

### Command-line arguments

| Argument | Default | Description |
|---|---|---|
| `--rounds` | 30 | Number of federated communication rounds |
| `--runs` | 5 | Independent training runs (for mean ± std) |
| `--seed` | 42 | Random seed |
| `--real` | False | Use real GDSC2/CCLE data if downloaded |

---

## Generated Outputs

Running the pipeline produces **10 figures** and the model is evaluated on **5-fold cross-validation across 5 independent runs**, matching the exact protocol used by DeepCDR and DualGCN.

### Figures 1–5 (Standard)

| Figure | Description | Mirrors |
|---|---|---|
| `fig1_performance_comparison.png` | Bar chart: PCC / Spearman / RMSE vs all baselines | DeepCDR Table 1 |
| `fig2_scatter_cancer_types.png` | Observed vs predicted ln(IC₅₀) across 6 cancer types | DeepCDR Fig. 2A–D |
| `fig3_per_drug_waterfall.png` | Per-drug PCC ranked waterfall + PCC vs RMSE scatter | DrugCell Fig. 2E |
| `fig4_training_convergence.png` | Loss curves + PCC over rounds + PPO dynamics | — |
| `fig5_tsne_embeddings.png` | t-SNE pathway space coloured by sensitivity + category | — |

### Figures 6–10 (Complex, Publication-Quality)

| Figure | Description | Mirrors |
|---|---|---|
| `fig6_ig_network.png` | IG heatmap across clients + global bar + circular pathway network | DualGCN interpretability |
| `fig7_roc_pr_curves.png` | ROC + Precision-Recall per cancer type + overall | DualGCN clinical style |
| `fig8_federated_ppo_analysis.png` | 6-panel: client profiles, selection matrix, PPO reward, convergence | Novel |
| `fig9_miot_analysis.png` | Cross-attention heatmap + device-group importance + correlation matrix | Novel |
| `fig10_clinical_panel.png` | Kaplan-Meier (3 groups) + resistance trajectory + rare subpop scatter | DrugCell Fig. 6–7 |

---

## Recommended Datasets

Use the following four datasets to generate results that improve upon the comparison papers:

### Primary Training — GDSC2
**Genomics of Drug Sensitivity in Cancer v2**
- 809 cancer cell lines × 170 drugs, ln(IC₅₀) values
- Improved experimental protocol over GDSC1 (used by DeepCDR and DualGCN)
- **Download:** https://www.cancerrxgene.org/downloads/bulk_download
- **Files needed:**
  - `GDSC2_fitted_dose_response_25Feb20.xlsx` → convert to CSV → rename `GDSC2_fitted_dose_response.csv`
  - `Cell_line_RMA_proc_basalExp.txt` → rename `GDSC2_gene_expression.csv`
- **Place at:** `data/GDSC2_fitted_dose_response.csv` and `data/GDSC2_gene_expression.csv`

### Secondary Validation — CCLE
**Cancer Cell Line Encyclopedia**
- 1,019 cell lines, gene expression (TPM), somatic mutations
- **Download:** https://depmap.org/portal/download/
- **Files needed:**
  - `CCLE_expression.csv` (22Q4 release or later)
  - `CCLE_mutations.csv`
- **Place at:** `data/CCLE_expression.csv`

### Cross-Study Validation — gCSI
**Genentech Cell Line Screening Initiative**
- 788 cell lines × 44 drugs, independently replicated from GDSC/CCLE
- This is the **key dataset** for demonstrating cross-study generalisation — no prior FL paper has used it
- **Download:** https://pharmacodb.ca/ (PharmacoDB v3, PharmacoGx R package)
- **Place at:** `data/gCSI_drug_sensitivity.csv`

### Clinical Translation — TCGA
**The Cancer Genome Atlas**
- 11,315 patient tumours, RNA-seq + clinical outcomes + mutation profiles
- Used for survival analysis and clinical validation
- **Download:** https://portal.gdc.cancer.gov/
- **Access:** Requires free account; most expression data is open-access

### MIoT Surrogate — MIMIC-IV v3.1
**Medical Information Mart for Intensive Care IV**
- ICU patient vital signs: heart rate, SpO₂, temperature, labs, drug infusion rates
- Maps directly to the 14 MIoT channels in PharmFed-DRL
- **Download:** https://physionet.org/content/mimiciv/3.1/
- **Access:** Requires CITI training + PhysioNet credentialing (~1 week)
- **Place at:** `data/MIMIC_IV_vitals.csv`

### Why These Datasets Beat the Comparison Papers

| Paper | Dataset Used | PharmFed-DRL Advantage |
|---|---|---|
| DeepCDR (2020) | GDSC1 (561 lines, 238 drugs) | GDSC2 has 809 lines, improved protocol |
| DualGCN (2022) | GDSC1 (525 lines, 208 drugs) | +gCSI cross-study validation (independent replication) |
| DrugCell (2020) | GDSC + CTRP (AUC metric) | ln(IC₅₀) on GDSC2 + federated privacy + MIoT |
| All three | Centralised, single-source | Multi-institutional federation + DRL client selection |

---

## Model Architecture

```
INPUT MODALITIES
─────────────────────────────────────────────────────────
Gene Expression (697 genes)
    ↓ ssGSEA / RWR
Pathway Scores (38 pathways) ──────────────────┐
                                               │
Mutations (512-dim binary)                     │
    ↓ Mutation Autoencoder (512→128→64)        │
Mutation Latent z_m (64-dim) ─────────────────→ Genomic
                                               │ Concat
Drug Fingerprints (256-bit Morgan)             │ (38+64+64=166)
    ↓ Drug Autoencoder (256→128→64)            │     ↓
Drug Latent z_d (64-dim) ─────────────────────→ Genomic
                                               │ Projection
                                               │ (166→64)
                                               ↓
                                    Genomic Embedding g (64-dim)
                                               │
                                    Cross-Attention Fusion
                                    ┌──────────┴──────────┐
                                    Q=g·W_Q          K,V = z_v·W_K,V
                                    └──────────┬──────────┘
MIoT Features (14 channels)                   │ α = σ(Q·K^T/√d)·V
    ↓ MIoT Encoder (14→64→64)                 │
MIoT Latent z_v (64-dim) ─────────────────────┘
                                    Fused Embedding f (64-dim)
                                               │
                                    Predictor FC Network
                                    (64→128→64→32→1)
                                               │
                                    ln(IC₅₀) prediction
─────────────────────────────────────────────────────────
PPO AGENT (per federated client)
  State:  [pathway_attributions(38) | val_loss(1) | size_rank(1) | t/T(1)]
  Action: participate ∈ {0, 1}  (Bernoulli policy)
  Reward: −Δ(val_loss) + 0.05·communication_saving
  Update: PPO clipped surrogate, GAE returns, entropy bonus
```

---

## Key Hyperparameters

| Parameter | Value | Description |
|---|---|---|
| `FED_ROUNDS` | 30 | Communication rounds |
| `N_CLIENTS` | 5 | Federated institutions |
| `LOCAL_EPOCHS` | 3 | Local SGD epochs per round |
| `K_FOLDS` | 5 | Cross-validation folds (matches DeepCDR/DualGCN) |
| `N_RUNS` | 5 | Independent runs for std (matches DeepCDR) |
| `LATENT_DIM` | 64 | Shared latent dimension |
| `DROPOUT` | 0.3 | Dropout rate |
| `CLIENT_LR` | 5e-3 | Local learning rate |
| `PPO_LR` | 3e-4 | PPO agent learning rate |
| `PPO_CLIP` | 0.2 | PPO clipping epsilon ε |
| `PPO_GAMMA` | 0.99 | Discount factor γ |
| `N_PATHWAYS` | 38 | MSigDB Hallmark cancer driver pathways |
| `N_MIOT_CH` | 14 | MIoT physiological channels |

---

## Evaluation Metrics

All metrics are reported as **mean ± std over 5 independent runs**, exactly matching DeepCDR Table 1 and DualGCN Table 1 format:

| Metric | Formula | Direction |
|---|---|---|
| **PCC** | Pearson correlation between observed and predicted ln(IC₅₀) | ↑ Higher is better |
| **Spearman ρ** | Rank correlation | ↑ Higher is better |
| **RMSE** | √(mean((y - ŷ)²)) | ↓ Lower is better |
| **R²** | 1 − SS_res / SS_tot | ↑ Higher is better |
| **AUC-ROC** | Area under ROC curve (binary classification) | ↑ Higher is better |
| **AUC-PR** | Area under Precision-Recall curve | ↑ Higher is better |

---

## Reproducing Published Baselines

The published performance numbers from the three comparison papers are hardcoded in `data/dataset.py`:

```python
DEEPCDR_REPORTED = dict(PCC=0.923, PCC_std=0.006,
                        Spearman=0.903, Spearman_std=0.004,
                        RMSE=1.058, RMSE_std=0.006)

DUALGCN_REPORTED = dict(PCC=0.925, PCC_std=0.001,
                        Spearman=0.907, Spearman_std=0.002,
                        RMSE=1.079, RMSE_std=0.007)

DRUGCELL_REPORTED = dict(Spearman=0.80, Spearman_std=0.01)
```

These are taken verbatim from:
- DeepCDR: Table 1, *Bioinformatics* 36(Suppl. 2):i911–i918 (2020)
- DualGCN: Table 1, *BMC Bioinformatics* 23(Suppl. 4):129 (2022)
- DrugCell: Figure 2A, *Cancer Cell* 38(5):672–684 (2020)

To replicate their experiments exactly, download their code and data from:
- DeepCDR: https://github.com/kimmo1019/DeepCDR
- DualGCN: https://github.com/horsedayday/DualGCN
- DrugCell: https://github.com/idekerlab/DrugCell

---

## Citation

If you use PharmFed-DRL in your research, please cite:

```bibtex
@article{pharmfed_drl_2026,
  title   = {PharmFed-DRL: Privacy-Preserving Harmonised Adaptive Multi-modal
             Reinforced Federated Drug Response Learning},
  author  = {[Authors]},
  journal = {[Target: Nature Communications / Cell Reports Medicine]},
  year    = {2026},
  note    = {Under review}
}
```

And the comparison papers:

```bibtex
@article{liu2020deepcdr,
  title   = {DeepCDR: a hybrid graph convolutional network for predicting
             cancer drug response},
  author  = {Liu, Qiao and Hu, Zhiqiang and Jiang, Rui and Zhou, Mu},
  journal = {Bioinformatics},
  volume  = {36}, number  = {Suppl. 2}, pages = {i911--i918},
  year    = {2020}, doi = {10.1093/bioinformatics/btaa822}
}

@article{ma2022dualgcn,
  title   = {DualGCN: a dual graph convolutional network model to predict
             cancer drug response},
  author  = {Ma, Tianxing and Liu, Qiao and Li, Haochen and Zhou, Mu
             and Jiang, Rui and Zhang, Xuegong},
  journal = {BMC Bioinformatics},
  volume  = {23}, number  = {Suppl. 4}, pages = {129},
  year    = {2022}, doi = {10.1186/s12859-022-04664-4}
}

@article{kuenzi2020drugcell,
  title   = {Predicting Drug Response and Synergy Using a Deep Learning
             Model of Human Cancer Cells},
  author  = {Kuenzi, Brent M. and Park, Jisoo and Fong, Samson H.
             and others},
  journal = {Cancer Cell},
  volume  = {38}, number  = {5}, pages = {672--684},
  year    = {2020}, doi = {10.1016/j.ccell.2020.09.014}
}
```

---

## References

1. Yang, W. et al. GDSC. *Nucleic Acids Research* 41, D955 (2013).
2. Barretina, J. et al. CCLE. *Nature* 483, 603 (2012).
3. Tirosh, I. et al. *Science* 352, 189 (2016).
4. Chen, J. et al. scDEAL. *Nature Communications* 13, 6494 (2022).
5. Yao, Y. et al. scPDS. *Small Methods* 9, e2400991 (2025).
6. Qi, G. et al. scXDR. *Communications Biology* 9, 139 (2026).
7. Xu, X. et al. HFDL-fl. *Heliyon* 9, e18615 (2023).
8. Salahat, M. et al. FWAFL. *Scientific Reports* 15, 31763 (2025).
9. McMahan, B. et al. FedAvg. *AISTATS* (2017).
10. Schulman, J. et al. PPO. *arXiv:1707.06347* (2017).
11. Sundararajan, M. et al. Integrated Gradients. *ICML* (2017).
12. Elmarakeby, H.A. et al. P-NET. *Nature* 598, 348 (2021).

---

## License

MIT License. See [LICENSE](LICENSE) for details.

> **Note on synthetic data:** The default run uses synthetically generated data that mirrors the statistical properties of GDSC2, CCLE, and gCSI. All published baseline performance numbers (DeepCDR, DualGCN, DrugCell) displayed in Figure 1 are taken verbatim from their original papers and are NOT derived from the synthetic data.
