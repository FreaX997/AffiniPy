# 🧬 AffiniPy: Automated Molecular Docking and Scoring Pipeline  

[![Python](https://img.shields.io/badge/Python-3.11%2B-blue)](https://www.python.org/)
[![License: Apache-2.0](https://img.shields.io/badge/License-Apache%202.0-yellowgreen.svg)](LICENSE)
![Platform](https://img.shields.io/badge/Platform-Linux-lightgrey)
![Status](https://img.shields.io/badge/Status-Active-brightgreen)
![Conda](https://img.shields.io/badge/Conda-Compatible-blueviolet)
![AutoDock Vina](https://img.shields.io/badge/Docking-AutoDock%20Vina-orange)

---

## 🧠 Overview  

**AffiniPy** is an automated, end-to-end molecular docking and scoring pipeline built in Python.  
It integrates **RDKit**, **Dimorphite-DL**, **Meeko**, **pdb2pqr**, **AutoDock Vina**, and **MDAnalysis** to streamline reproducible protein–ligand virtual screening, descriptor profiling, and composite scoring.  

AffiniPy automates everything from file preparation to ranked ligand output — ideal for rapid structure-based prioritization in drug discovery workflows.

---

## ✨ Key Features  

- ⚙️ Automated protein and ligand preparation  
- 🧪 Protonation with **Dimorphite-DL**  
- 🚀 Docking via **AutoDock Vina** (parallelized using Joblib)  
- 🧮 Descriptor profiling using **RDKit** (Lipinski, Veber, SA, LE)  
- 🧩 Composite affinity scoring and ranking  
- 📊 Color-coded Excel summary of results  
- 💾 Automated best-pose extraction and receptor–ligand complex generation  
- 🧯 Transparent logs and error tracking  

---

## ⚙️ Installation  

Clone this repository and create the environment using Conda.  

```bash
git clone https://github.com/FreaX997/AffiniPy.git
cd AffiniPy
conda env create -f Informatics.yml
conda activate Informatics
```

Verify that the required external tools are available:  
```bash
vina --help
obabel -V
pdb2pqr --version
```

> 🧠 *Tested on Linux (Ubuntu 22.10), Python 3.11, RDKit 2025.03, and AutoDock Vina 1.2.7. Windows compatibility not tested yet.*

---

## 📂 Input Data Setup  

To get started, keep **all required input files** in a single folder.  
This folder should include:

```
AffiniPy/
├── AffiniPy.ipynb
├── sascorer.py
├── fpscores.pkl.gz          # companion file for sascorer
├── Informatics.yml
├── 7JGW.pdb                 # Protein PDB(s)
├── ligand1.sdf              # Ligand SDF(s)
├── ligand2.sdf
└── ...
```

When you run **AffiniPy.ipynb**, the pipeline automatically creates the following filesystem:

```
AffiniPy/
├── Data/
│   ├── Protein/
│   └── Ligands_Raw/
├── Prep/
|   ├── Ligands_prepared
|   └── Proteins_prepared
├── Docking/
│   ├── Result_PDBQTs/
│   ├── BestPoses/
│   └── ...
```

You’ll only need to:
- Move your protein `.pdb` files into `/Data/Protein/`
- Move your ligand `.sdf` files into `/Data/Ligands_Raw/`

Everything else — preparation, docking, descriptor calculation, and scoring — is fully automated.


### 🧩 Notes on Protein Input:
- The pipeline expects **two protein files**:  
  - `7JGW.pdb`: original structure downloaded from RCSB PDB.  
  - `7JGW_2.pdb`: prepared in **UCSF Chimera’s Dock Prep** to fix alternate residues, incomplete side chains, and remove water/ligands.  
- AffiniPy automatically isolates the co-crystallized ligand if the **ligand ID from the RCSB PDB page of the protein** is manually specified in **Code Cell 3** of the notebook.  

---

## 🎯 Grid Box Configuration  

In **Code Cell 5**, define the docking grid box:

**Manual mode (recommended for reproducibility):**
```python
gbox_centre = [-2.96, -2.55, 26.75]
gbox_size   = [11.25, 20.10, 24.24]
```

**Automatic mode (optional) (uncomment these lines from Code Cell 5):**
```python
gbox_centre = lig_only.center_of_geometry().tolist()
gbox_size = lig_only.positions.max(axis=0) - lig_only.positions.min(axis=0) + 10.0
```
*(Requires the co-crystallized ligand to be present.)*

---

## 💊 Ligand Preparation  

- Place **multiple `.sdf` files** in `Data/Ligands_Raw/` — one ligand per file.  
- AffiniPy automatically performs:  
  - Protonation (Dimorphite-DL at pH 7.4)  
  - 3D conformer generation (RDKit ETKDGv3)  
  - MMFF94 energy minimization  
  - Conversion to `.pdbqt` using Meeko  

> ❗ Failed ligands are automatically skipped and logged (e.g., 3D generation or protonation failure).

---

## 🚀 Running the Pipeline  

Open the Jupyter Notebook (`AffiniPy.ipynb`) in **VS Code** or **JupyterLab** and execute cells sequentially.  
Each code block corresponds to a distinct stage of the workflow:

> **Protein Prep → Ligand Prep → Docking → Descriptor Calculation → Composite Scoring → Output**

You can re-run from any stage; prepared files are reused automatically.

---

## 📊 Output Files and Interpretation  

The output structure after execution:

```
AffiniPy/
├── Docking/
│   ├── Result_PDBQTs/              # Docking output and logs
│   ├── BestPoses/                  # Complexed protein–ligand best poses
│   └── 7JGW_screening_results_summary.xlsx
```

### 🧾 Excel Summary Spreadsheet Breakdown

| Sheet | When It Appears | Description |
|--------|----------------|--------------|
| **Docking + Descriptors** | Always | Main results sheet: lists all ligands with docking scores, ligand efficiency, synthetic accessibility (SA), Lipinski/Veber flags, and composite scores. |
| **Errors** | If some ligands fail during docking | Contains ligand names and specific error messages captured from Vina output. |
| **Pose Extraction Errors** | If best-pose generation fails | Lists ligands where PDBQT → PDB conversion or protein merging failed. |

---

## 🧮 Composite Scoring  

AffiniPy ranks ligands by a **Composite Score** integrating binding energy, efficiency, and synthetic feasibility.

| Parameter | Weight | Direction |
|------------|---------|------------|
| Docking Score | 0.45 | Lower is better |
| Ligand Efficiency | 0.30 | Higher is better |
| Synthetic Accessibility | 0.15 | Lower is better |
| SA Penalty | 0.10 | Lower is better |
| Lipinski / Veber Bonus | +0.05 each | Pass increases score |

> 💡 The composite score highlights ligands that balance strong binding, efficiency, and synthetic feasibility.  
> Users can manually adjust the weights in **Code Cell 11** to re-prioritize parameters.

---

## 🧪 Example Dataset  

An example dataset derived from **B. Hassan Ganesh, B. Aruchamy, S. Mudradi, S. Mohanty, H. Padinjarathil, S. Carradori, P. Ramani, ChemMedChem 2025, 20, e202400371.**  
[https://doi.org/10.1002/cmdc.202400371](https://doi.org/10.1002/cmdc.202400371)  
is included under `Example_Data/` to demonstrate input and output formats. To use these example files, please move the files from the `Example_Data/` folder into the folder where `AffiniPy.ipynb`, `sascorer.py` and `fpscores.pkl.gz` are located.

---

## 🧰 Dependencies  

All dependencies are listed in `Informatics.yml`.  
Key libraries and tools include:

| Category | Tools |
|-----------|--------|
| Core | Python 3.11+, RDKit, MDAnalysis, pandas, scikit-learn |
| Docking | AutoDock Vina, Meeko |
| Protonation | Dimorphite-DL |
| Descriptor | sascorer |
| Utilities | openbabel, pdb2pqr, joblib, numpy, openpyxl |

---

## 📄 License  

This project is licensed under the [Apache License 2.0](LICENSE) © 2025 Sarthak Mohanty  

You are free to use, modify, and distribute this code under the terms of the Apache 2.0 license with proper attribution.  

---

## 🌟 Citation  

If you use **AffiniPy** in your research, please cite:

> **Sarthak Mohanty (2025).** *AffiniPy: Automated Molecular Docking and Scoring Pipeline.*  
> GitHub: [https://github.com/FreaX997/AffiniPy](https://github.com/FreaX997/AffiniPy)

---

## ⚠️ Current Limitations & Planned Improvements  

- Requires pre-cleaned PDB structures (manual Dock Prep in Chimera recommended)  
- Grid box must currently be set manually for best reliability  
- Composite scoring weights are static (can be changed manually in code)  
- Designed and tested on Linux systems; Windows compatibility untested  

---

## 🧭 Notes  

AffiniPy is a continuously evolving open-source project.  
Feature additions, optimization strategies, and modular improvements will be introduced over time as the workflow matures and community feedback grows.  
Stay tuned on the repository for updates. 🚀

---

> 🦉 *Built by Sarthak with curiosity, late-nights, impostor syndrome, and a healthy disregard for un-automated workflows.*  
> “Reproducible docking doesn’t have to be painful.”
