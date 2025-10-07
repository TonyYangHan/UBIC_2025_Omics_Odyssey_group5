# UBIC 2025 Omics Odyssey Group 5 Analysis Repo <br>

**Authors: Yang Han and Alain Li** <br>
Hosted by Undergraduate Bioinformatics Club at UCSD. Organized by Kyra Fetter and Willard Ford.

Over 24 hours we hosted 6 teams to help answer interesting bioinformatics questions. Participants ranged from first-years to masters students. 

## Repository summary

This project explores mouse retinal bipolar cells with single-cell RNA-seq analysis. It includes:

- Quality control and exploration of the provided AnnData object (`bipolar_raw.h5ad`).
- Pseudobulk differential expression between young (P17) and mature (W8) animals per bipolar cell type using edgeR (R).
- GO Biological Process enrichment of up/down DEGs using clusterProfiler (R) and summary dotplots.
- A targeted age classifier (XGBoost) trained within a selected cell subtype (Python), with feature importance inspection to the contribution of genes to the development of the cell subtype at various times.
- A simple pseudotime analysis (Scanpy DPT) for discovering developmental ordering at transcriptome level.

Primary notebooks:

- `QC_Exploration.ipynb`: sanity checks, distributions, UMAPs.
- `DEG_data_prep_age.ipynb`: pseudobulk generation per cell type and export of expressed gene background.
- `Run_DEG_GO.ipynb` (R): edgeR DEG per cell type (W8 vs P17), volcano plots, and GO enrichment (up/down) plus dotplots.
- `Classify_age.ipynb`: balances W8/P17 for a chosen cell type (type 5a cone bipolar cells), trains an XGBoost classifier, evaluates and inspects gene's contribution to different age classification.
- `Pseudotime.ipynb`: basic preprocessing and DPT pseudotime with visualization. Demostrating the development trajectory of rod bipolar cells at transcriptome level overlaps with actual time of development.

Expected inputs:

- `bipolar_raw.h5ad` in the repository root, with at least the following in `adata.obs`:
  - `cell_type`, `age` (values include P17 and W8), `donor_id`
  - raw counts available in `adata.raw`

Generated outputs (under `results/`):

- `age_deg_edgeR/` with pseudobulk counts and metadata per cell type.
- `W8_vs_P17_DEG/` DE tables per cell type (edgeR).
- `W8_vs_P17_GO_up/` and `W8_vs_P17_GO_down/` GO enrichment tables.
- `DEG_volcano_plots/` volcano PNGs per cell type.
- Summary dotplots: `W8_vs_P17_GO_up_plot.png`, `W8_vs_P17_GO_down_plot.png`.

## Requirements

General

- OS: Windows, macOS, or Linux (tested workflows are platform-agnostic; commands below use Windows PowerShell).
- Data: `bipolar_raw.h5ad` in the repo root with required obs fields and `adata.raw` counts. [download link](https://datasets.cellxgene.cziscience.com/44809d09-72c7-45e4-8744-5e48e36ab0d8.h5ad) [data visualization link](https://cellxgene.cziscience.com/e/40e25c08-8d33-4768-89dd-501def6191d7.cxg/)

Python (>=3.10 recommended)

- scanpy, anndata, numpy, pandas, seaborn, matplotlib
- xgboost (GPU optional), scikit-learn, joblib

R (>=4.2 recommended)

- Seurat
- edgeR
- ComplexHeatmap, viridis, ggplot2, tidyverse
- doParallel, foreach
- clusterProfiler, org.Mm.eg.db, dplyr

Optional GPU acceleration

- XGBoost can use CUDA. Install a CUDA-enabled xgboost build and appropriate NVIDIA drivers if available; otherwise the CPU build works.

## Setup and usage

1) Create a Python virtual environment and install packages

```powershell
# From the repo root
python -m venv .venv; .\.venv\Scripts\Activate.ps1; pip install --upgrade pip
pip install scanpy anndata numpy pandas seaborn matplotlib scikit-learn xgboost joblib
```

2) Install R packages (run inside R)

```r
install.packages(c("Seurat","tidyverse","ggplot2","viridis","dplyr"))
# For Bioconductor packages
if (!requireNamespace("BiocManager", quietly = TRUE)) install.packages("BiocManager")
BiocManager::install(c(
  "edgeR",
  "ComplexHeatmap",
  "doParallel",
  "foreach",
  "clusterProfiler",
  "org.Mm.eg.db"
))
```

3) Place input data

- Copy `bipolar_raw.h5ad` into the repository root.

4) Run the notebooks in order

- `QC_Exploration.ipynb`: Inspect data quality and metadata.
- `DEG_data_prep_age.ipynb`: Produces `results/age_deg_edgeR/{counts,meta_data}/...` and `results/expressed_genes.csv`.
- `Run_DEG_GO.ipynb` (R kernel):
  - Generates `results/W8_vs_P17_DEG/*.csv` via edgeR.
  - Saves volcano plots to `results/DEG_volcano_plots/`.
  - Writes GO enrichment to `results/W8_vs_P17_GO_up/` and `results/W8_vs_P17_GO_down/` and summary dotplots.
- `Classify_age.ipynb`: Trains age classifier for a selected cell type (defaults to type 5a), produces confusion matrix and feature importance plots.
- `Pseudotime.ipynb`: Runs DPT pseudotime and visualizes trajectories.

## Results and structure

- `results/age_deg_edgeR/`
  - `counts/counts_<cell_type>.csv`: gene-by-sample pseudobulk counts.
  - `meta_data/meta_<cell_type>.csv`: sample metadata with age and donor.
- `results/W8_vs_P17_DEG/`: edgeR QL results with logFC, FDR.
- `results/DEG_volcano_plots/`: volcano PNGs per cell type.
- `results/W8_vs_P17_GO_up/` and `..._down/`: GO BP enrichment tables per cell type.
- Summary dotplots at repo root `results/`: `W8_vs_P17_GO_up_plot.png`, `W8_vs_P17_GO_down_plot.png`.

Notes and troubleshooting

- AnnData assumptions: `adata.obs` has `cell_type`, `age` (contains P17, W8), `donor_id`; `adata.raw` holds raw counts used for pseudobulk.
- DEG and GO use ENSEMBL IDs as produced from pseudobulk; ensure your gene identifiers match the `OrgDb` keyType (`ENSEMBL`).
- If xgboost GPU is unavailable, switch the classifier to CPU by removing `device='cuda'` in `Classify_age.ipynb`.
- On first R run, some packages may require system tools; install per usual R/Bioconductor guidance.

## Copyright

Copyright (c) 2025 Undergraduate Bioinformatics Club (UBIC) and project contributors.
All rights reserved. This repository is provided for educational and research purposes during the 2025 Omics Odyssey. If you plan to reuse or redistribute, please add an explicit open-source license file or obtain permission from the authors.
