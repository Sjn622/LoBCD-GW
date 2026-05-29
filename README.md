#LoBCD-GW Experiments 

LoBCD-GW: A Fast and Data-Dependent Algorithm for Computing Gromov-Wasserstein Distance via Localized Block Coordinate Descent


## 1. Overview

Included components:
- **Solvers**
  - `LoBCD_GW.py`: localized/block-coordinate LoBCD-GW implementation 1.
  - `LoBCD_GW2.py`: localized/block-coordinate LoBCD-GW implementation 2.
- **Runners**
  - `run_LoBCD_GW.py`: graph matching benchmarks.
  - `run_scGEM_LoBCD_GW.py`: scGEM two-modality alignment.
  - `run_scNMT_LoBCD_GW.py`: scNMT three-modality alignment grid search.
- **Generate Synthetic Data**
  - `generate_synthetic_data.py`: the script that generates synthetic data
---

## 2. Repository Structure

Minimal expected layout:

```
.
├── LoBCD_GW.py
├── LoBCD_GW2.py
├── run_LoBCD_GW.py
├── run_scGEM_LoBCD_GW.py
├── run_scNMT_LoBCD_GW.py
├── generate_synthetic_data.py
└── data/
```

Notes:
- Datasets are expected under `data/`.

---

## 3. Requirements

### 3.1 Python
- Python **3.8+** recommended

### 3.2 Dependencies
Core packages:
- `numpy==1.26.4`
- `scipy==1.11.4`
- `torch==2.2.2`
- `networkx==3.2.1`
- `pandas==2.2.2`
- `scikit-learn==1.4.2`

Optional:
- CUDA-enabled PyTorch for GPU acceleration.

### 3.3 Installation (pip)

Create an environment (example):
```bash
python -m venv .venv
source .venv/bin/activate
pip install --upgrade pip
```

Install dependencies:
```bash
pip install numpy scipy torch networkx pandas scikit-learn
```

For a specific CUDA build of PyTorch, install according to official PyTorch instructions for your platform.

---

## 4. Runtime Environment

### 4.1 Hardware
- CPU-only execution is supported.
- GPU acceleration is supported if PyTorch CUDA is available.

### 4.2 Numerical / Performance Notes
Some scripts enable cuDNN benchmarking and TF32 for speed (if supported by hardware):
- For stricter FP32 behavior, set:
  - `torch.backends.cuda.matmul.allow_tf32 = False`
  - `torch.backends.cudnn.allow_tf32 = False`
  - `cudnn.benchmark = False`

---

## 5. Datasets

All datasets should be placed under:
- `data/`

The runners expect the corresponding dataset files to exist under `data/` following their default internal paths.

Generate the synthetic dataset, then execute the above command:
```bash
python generate_synthetic_data.py
```

Covered tasks:
- Graph matching benchmarks (multiple datasets supported by `run_LoBCD_GW.py`).
- Multi-omics alignment benchmarks:
  - scGEM (`run_scGEM.py`)
  - scNMT (`run_scNMT.py`)

If you want to use a custom location, pass `--data_path` accordingly (for scGEM/scNMT runners), or modify paths in the runner for graph datasets.

---

## 6. Usage

### 6.1 Graph Matching (LoBCD-GW)

Example (use GPU if available):
```bash
python run_LoBCD_GW.py --dataset reddit --use_gpu 1 --rho 0.1 --max_iter 500 --eps 1e-5
```

Common arguments:
- `--dataset`: dataset name (choices are defined in the script)
- `--noise_level`: add random edges (robustness testing)
- `--rho`: one or multiple values, e.g. `--rho 0.1 0.01`
- `--max_iter`: outer iterations
- `--eps`: internal convergence threshold
- `--use_gpu`: `1` use CUDA if available, `0` force CPU
- `--amp`: `1` enable autocast (may slightly change numerics)
- `--compile`: `1` enable `torch.compile` if supported

Outputs:
- Per-graph metrics and timing printed to stdout.
- Summary appended to `--output_file` (default: `result_optimized.txt`).

### 6.2 scGEM Alignment (LoBCD-GW)

Dense graph mode:
```bash
python run_scGEM.py --data_path data/scGEM --graph_mode dense --pca 14 --rho 5e-4 --min_rho 1e-5 --sinkhorn_iters 1 --max_iter 2200 --device cuda
```

kNN graph mode:
```bash
python run_scGEM.py --data_path data/scGEM --graph_mode knn --k 5 --pca 14 --rho 5e-4 --min_rho 1e-5 --sinkhorn_iters 1 --max_iter 2200 --device cuda
```

Outputs:
- Feature dimensions (before PCA), runtime, and bidirectional label-transfer accuracy.

### 6.3 scNMT Grid Search (LoBCD-GW2)

Dense mode:
```bash
python run_scNMT.py --data_path data/scNMT --graph_mode dense --sinkhorn_iters 1 --max_iter 2000 --device cuda
```

kNN mode:
```bash
python run_scNMT.py --data_path data/scNMT --graph_mode knn --k 15 --sinkhorn_iters 1 --max_iter 2000 --device cuda
```

Outputs:
- Per-setting accuracies and runtime, plus best configs for:
  - MET ↔ RNA
  - ACC ↔ RNA

---
