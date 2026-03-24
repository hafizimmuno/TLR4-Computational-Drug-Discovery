# TLR4 Computational Drug Discovery

![Python](https://img.shields.io/badge/Python-3.10-blue)
![License](https://img.shields.io/badge/License-MIT-green)
![Platform](https://img.shields.io/badge/Platform-Google%20Colab-orange)
![Status](https://img.shields.io/badge/Status-Complete-brightgreen)

A computational pipeline for **TLR4-MD2** ligand discovery combining machine learning-based bioactivity prediction, molecular docking and rule-based analysis.

---

## Table of Contents

- [Background](#background)
- [Project Overview](#project-overview)
- [Key Results](#key-results)
- [Repository Structure](#repository-structure)
- [Notebooks](#notebooks)
- [Installation](#installation)
- [Methods](#methods)
- [Top Hits](#top-hits)
- [Binding Pocket Analysis](#binding-pocket-analysis)
- [Contact](#contact)

---

## Background

**Toll-Like Receptor 4 (TLR4)** is a pattern recognition receptor on immune cells that detects bacterial lipopolysaccharide (LPS) and initiates innate immune responses. TLR4 signals through its co-receptor **MD-2**, which contains a deep hydrophobic pocket where LPS binds. This makes TLR4-MD2 a critical therapeutic target in two areas:

- **Vaccine adjuvants** — TLR4 agonists enhance immune responses to vaccines
- **Sepsis and inflammatory disease** — TLR4 antagonists can suppress dangerous cytokine storms

This project builds a fully integrated computational pipeline to identify small molecules that bind the MD-2 pocket — from raw ChEMBL bioactivity data to docked 3D poses with interpretable ML predictions.

---

## Project Overview

```
ChEMBL Database
      │
      ▼
Bioactivity Data Retrieval & Curation
      │
      ▼
ECFP4 Molecular Fingerprints
      │
      ├──► Random Forest Regressor (pChEMBL prediction)
      │         │
      │         ▼
      │    SHAP Feature Importance
      │
      ├──► Decision Tree Classifier (IF-THEN rules)
      │
      ▼
Virtual Screening — Drug-like Candidates
      │
      ▼
AutoDock Vina — MD-2 Pocket Docking (PDB: 3FXI)
      │
      ▼
Consensus Ranking (ML + Docking)
      │
      ▼
Top Hit Candidates
```

---

## Key Results

| Metric | Value | Significance |
|--------|-------|-------------|
| ML–Docking Correlation | r = 0.67, p = 0.006 | Two independent methods significantly agree |
| Random Forest ROC-AUC | 0.840 (5-fold CV) | Strong active vs inactive discrimination |
| Decision Tree ROC-AUC | 0.805 (5-fold CV) | Interpretable rules with minimal accuracy loss |
| Best Docking Score | −9.37 kcal/mol | Strong predicted binding in MD-2 pocket |
| Consensus Hits | 3 compounds | High-confidence — ranked well by both methods |
| ML Prediction Accuracy | Predicted ≈ Observed pChEMBL | Model validated against real lab measurements |

---

## Repository Structure

```
TLR4-Computational-Drug-Discovery/
│
├── 01_TLR4_ML_Bioactivity_Modeling.ipynb
├── 02_TLR4_Molecular_Docking.ipynb
├── 03_TLR4_Integrated_ML_Docking_Pipeline.ipynb
├── 04_TLR4_Decision_Tree_Rule_Extraction.ipynb
├── requirements.txt
├── LICENSE
└── README.md
```

---

## Notebooks

### Notebook 01 — ML Bioactivity Modeling
`01_TLR4_ML_Bioactivity_Modeling.ipynb`

Retrieves TLR4 bioactivity data from ChEMBL (human single-protein TLR4), filters to quality-controlled IC50/EC50/Ki measurements, converts molecules to ECFP4 Morgan fingerprints (radius=2, 2048 bits), and trains a Random Forest Regressor to predict pChEMBL values. Uses SHAP (SHapley Additive exPlanations) to identify which molecular features drive TLR4 activity.

- **Data source:** ChEMBL functional assays, human TLR4, nM units
- **Features:** ECFP4 fingerprints (2048-bit)
- **Model:** Random Forest (300 trees, 5-fold cross-validation)
- **Explainability:** SHAP beeswarm plot

---

### Notebook 02 — Molecular Docking
`02_TLR4_Molecular_Docking.ipynb`

Downloads the TLR4-MD2 crystal structure (PDB: 3FXI) in mmCIF format from RCSB, parses it with BioPython's MMCIFParser, cleans chains A and C by removing heteroatoms and non-standard residues, saves as PDB, and converts to PDBQT format using Open Babel. Calculates the geometric center of the MD-2 pocket (x=29.39, y=1.30, z=20.49) and runs AutoDock Vina docking with a 24×24×24 Å search box.

- **Protein structure:** PDB 3FXI (human TLR4-MD2 complex)
- **Docking software:** AutoDock Vina (exhaustiveness=4, 3 poses)
- **Binding site:** MD-2 hydrophobic pocket (Chain C)
- **Format conversion:** Open Babel (PDB → PDBQT)

---

### Notebook 03 — Integrated ML to Docking Pipeline
`03_TLR4_Integrated_ML_Docking_Pipeline.ipynb`

The core pipeline notebook. ML model shortlists the most promising compounds (filtered for drug-likeness: MW < 600 Da, rotatable bonds ≤ 10), which are then docked into the MD-2 pocket. Consensus ranking combines ML-predicted pChEMBL rank and docking score rank to identify top hits validated by both independent methods.

- **Virtual screening:** 15 drug-like candidates docked
- **Consensus method:** Average of ML rank + docking rank
- **Validation:** Pearson r = 0.67 between ML prediction and docking score
- **Statistical significance:** p = 0.006

---

### Notebook 04 — Decision Tree and Rule Extraction
`04_TLR4_Decision_Tree_Rule_Extraction.ipynb`

Trains a shallow Decision Tree classifier (max_depth=4) on the same TLR4 data and extracts explicit IF-THEN rules from fingerprint bits. Achieves ROC-AUC of 0.805 — only 3.5% below Random Forest — while providing fully interpretable predictions that a medicinal chemist can apply directly without software.

- **Model:** Decision Tree (max_depth=4, balanced class weights)
- **ROC-AUC:** 0.805 (5-fold CV)
- **Output:** IF-THEN rules from ECFP4 fingerprint bits
- **Trade-off:** 3.5% accuracy for full explainability

---

## Installation

All notebooks are designed to run on **Google Colab**. Clone the repository and open any notebook directly in Colab:

```bash
git clone https://github.com/hafizimmuno/TLR4-Computational-Drug-Discovery.git
```

Install Python dependencies:

```bash
pip install -r requirements.txt
```

Additional system-level installations required in Colab (included at the top of each notebook):

```bash
# Open Babel for molecular format conversion
apt-get install openbabel

# AutoDock Vina for molecular docking
pip install vina
```

---

## Methods

| Step | Method | Tool |
|------|--------|------|
| Data retrieval | ChEMBL API query | chembl-webresource-client |
| Molecular representation | ECFP4 fingerprints (r=2, 2048-bit) | RDKit |
| Activity prediction | Random Forest Regressor | scikit-learn |
| Model explainability | SHAP values | shap |
| Classification | Decision Tree Classifier | scikit-learn |
| Protein preparation | Chain selection, heteroatom removal | BioPython |
| Format conversion | PDB to PDBQT | Open Babel |
| Molecular docking | AutoDock Vina | vina |
| Visualization | 3D protein-ligand visualization | PyMOL |

---

## Top Hits

Consensus hits ranked by combined ML prediction and docking score:

| Compound | ML Predicted pChEMBL | Docking Score | Observed pChEMBL |
|----------|---------------------|---------------|-----------------|
| CHEMBL3261034 | 6.28 | −9.37 kcal/mol | 6.41 |
| CHEMBL4643502 | 6.28 | −9.19 kcal/mol | 6.60 |
| CHEMBL224563 | 6.46 | −7.36 kcal/mol | 6.80 |

ML predictions closely match experimentally observed pChEMBL values, confirming model validity on top-ranked compounds.

---

## Binding Pocket Analysis

PyMOL analysis of the best hit **CHEMBL3261034** docked into the MD-2 pocket (PDB: 3FXI) identified 34 contact residues within 4 Å. Key interactions:

**Critical hydrophobic contacts:**
PHE76, PHE104, PHE119, PHE121, PHE126, PHE147, PHE151

**Polar and H-bond interactions:**
TYR65, TYR131, THR115, SER120

**Electrostatic anchoring:**
ARG90 (key positive charge), GLU92

PHE76, PHE126, and ARG90 are established critical residues for MD-2 ligand binding in the literature, validating that the docking pipeline correctly identifies the biologically relevant binding mode.

---

## Contact

**Hafiz Ahmad**
Université Claude Bernard Lyon 1
hafiz.hassan.ahmad88@gmail.com
GitHub: [@hafizimmuno](https://github.com/hafizimmuno)

---

## License

This project is licensed under the MIT License — see the [LICENSE](LICENSE) file for details.

---

*Built with Google Colab | RDKit | AutoDock Vina | scikit-learn | PyMOL*
