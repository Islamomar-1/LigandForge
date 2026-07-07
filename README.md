# 🧪 Automated Virtual Screening Pipeline

A modular and reproducible **virtual screening system** for drug discovery.  
This pipeline integrates **ADMET filtering, 3D structure generation, ligand preparation, and docking with AutoDock Vina**, with robust provenance tracking and clean restarts.

---

## 📂 Pipeline Overview

The system is divided into four modules, each of which can be run independently or sequentially:

1. **Module 1 – ADMET Screening**  
   - Input: `input/input.csv` (id, SMILES, notes, params_json)  
   - Computes descriptors via **RDKit** (MW, logP, TPSA, HBD, HBA, RotB, InChIKey).  
   - Applies **drug-likeness rules** (Lipinski, Veber, optional Egan/Ghose).  
   - Outputs:  
     - `output/admet.csv`  
     - `state/admet_pass.list`, `state/admet_fail.list`  
     - Updates `state/manifest.csv`  

2. **Module 2 – SMILES → 3D (SDF)**  
   - Uses **RDKit ETKDGv3** + **UFF/MMFF minimization**.  
   - Converts SMILES into clean 3D `.sdf` files.  
   - Outputs:  
     - `3D_Structures/<id>.sdf`, `<id>.smi`  
     - Logs (`<id>_rdkit.log`)  
     - Updates manifest with `sdf_*` status  

3. **Module 3 – SDF → PDBQT (Ligand Prep)**  
   - Uses **Meeko** to prepare ligands for docking.  
   - Outputs:  
     - `prepared_ligands/<id>.pdbqt`  
     - Logs (`<id>_meeko.log`)  
     - Updates manifest with `pdbqt_*` status  

4. **Module 4 – Docking with AutoDock Vina**  
   - Runs **AutoDock Vina** using `VinaConfig.txt` placed next to the binary.  
   - Performs docking against target receptor.  
   - Outputs:  
     - `results/<id>_out.pdbqt` (poses)  
     - `results/<id>_vina.log`  
     - `results/summary.csv` (scores + metadata)  
     - `results/leaderboard.csv` (ranked ligands)  
     - Updates manifest with `vina_*` status  

---

## 🧹 Utility Script

### `WARNING_PURGE_PIPELINE.py`  
- Cleans pipeline outputs for a **fresh run**.  
- Preserves folder structure + CSV headers.  
- Deletes generated ligands/receptors/results.  
- Keeps `VinaConfig.txt` intact.  

---

## 📊 Manifest Tracking

A central **`state/manifest.csv`** records per-ligand progress through the pipeline:

- **ADMET**: status/reason, descriptors, InChIKey  
- **SDF**: file path, validation reason  
- **PDBQT**: ligand prep status  
- **Vina**: docking score, pose path, receptor SHA1  
- **Provenance**: config hashes, RDKit/Meeko/Vina versions, timestamps  

This makes the pipeline **idempotent** (safe to re-run without duplication).

---

## ⚙️ Installation

### Requirements
- Python **3.10+**
- [RDKit](https://www.rdkit.org/)
- [Meeko](https://github.com/forlilab/meeko)  
- [AutoDock Vina](http://vina.scripps.edu/) (binary placed in project root)  
- PyYAML (optional, for configs)

### Clone and Setup
```bash
git clone https://github.com/yourusername/virtual-screening-pipeline.git
cd virtual-screening-pipeline
pip install -r requirements.txt
```

*(Create a `requirements.txt` like below)*

```txt
rdkit-pypi
meeko
pyyaml
```

---

## 🚀 Usage

### 1. Prepare Input
Create `input/input.csv` with columns:
```
id,smiles,notes,params_json
```

### 2. Run Modules in Order
```bash
# 1. ADMET filtering
python Module\ 1.py

# 2. SMILES → 3D SDF
python Module\ 2.py

# 3. Ligand preparation
python Module\ 3.py

# 4. Docking
python Module\ 4.py
```

### 3. Reset Pipeline
```bash
python WARNING_PURGE_PIPELINE.py
```

---

## 📂 Project Structure

```
├── input/                  # input.csv (SMILES)
├── output/                 # admet.csv
├── 3D_Structures/          # generated SDFs
├── prepared_ligands/       # ligand PDBQTs
├── results/                # docking poses, summary, leaderboard
├── state/                  # manifest.csv, pass/fail lists
├── config/                 # optional run.yml, machine.yml
├── logs/                   # logs from modules
├── Module 1.py             # ADMET filtering
├── Module 2.py             # SMILES → SDF
├── Module 3.py             # SDF → PDBQT
├── Module 4.py             # Docking
└── WARNING_PURGE_PIPELINE.py  # reset script
```

---

## 🔑 Key Features
- ✅ **Modular**: each stage can be run independently  
- ✅ **Validated**: descriptors, SDF, PDBQT, docking pose checks  
- ✅ **Idempotent**: safe re-runs without duplication  
- ✅ **Atomic writes**: avoids corrupted/empty files  
- ✅ **Provenance**: config hash + tool versions logged  
- ✅ **Resettable**: one-command clean restart  

---

## 📜 License
MIT License – feel free to use and adapt.  
# LigandForge
