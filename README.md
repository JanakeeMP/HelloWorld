# Heart Disease Analysis (Python + MATLAB)

> End‑to‑end data analysis and modeling to study cardiovascular disease risk using Python, with select algorithms and signal processing implemented via the MATLAB Engine for Python.

---

![status-badges](https://img.shields.io/badge/python-3.10%2B-blue) ![matlab](https://img.shields.io/badge/MATLAB-Engine%20API-orange) ![license](https://img.shields.io/badge/license-MIT-green)

## Table of Contents

* [Overview](#overview)
* [Project Structure](#project-structure)
* [Requirements](#requirements)
* [Quick Start](#quick-start)
* [Detailed Setup](#detailed-setup)

  * [1) Python Environment](#1-python-environment)
  * [2) MATLAB Engine for Python](#2-matlab-engine-for-python)
  * [3) Project Configuration](#3-project-configuration)
* [Usage](#usage)
* [Reproducing Results](#reproducing-results)
* [Testing & Quality](#testing--quality)
* [Troubleshooting](#troubleshooting)
* [Contributing](#contributing)
* [License](#license)

---

## Overview

This repository contains code to explore and model heart disease risk. It includes:

* Data ingestion & cleaning pipelines (CSV/Parquet).
* Exploratory data analysis (EDA) with plots and summary statistics.
* Feature engineering for clinical and signal data (e.g., ECG time/frequency features).
* Model training & evaluation (classical ML + optional MATLAB algorithms).
* Reproducible experiments (config‑driven via YAML).

> **Note**: The phrase "heart dieasies" in earlier drafts has been corrected to **heart diseases** throughout this README.

## Project Structure

```
heart-disease-analysis/
├─ data/                       # (gitignored) raw/processed datasets
│  ├─ raw/
│  └─ processed/
├─ notebooks/                  # Jupyter notebooks for EDA & prototypes
├─ src/
│  ├─ hdlib/                   # Python package
│  │  ├─ __init__.py
│  │  ├─ data.py               # loading/cleaning
│  │  ├─ features.py           # feature engineering
│  │  ├─ models.py             # training/evaluation
│  │  ├─ matlab_bridge.py      # MATLAB Engine wrappers
│  │  └─ viz.py                # plotting utilities
│  └─ cli.py                   # command-line interface
├─ matlab/                     # .m files used by MATLAB Engine
│  ├─ ecg_features.m
│  └─ helpers/...
├─ configs/
│  ├─ default.yaml
│  └─ experiments/...
├─ tests/                      # unit tests
├─ .env.example
├─ requirements.txt
├─ pyproject.toml              # (or setup.cfg/requirements)
├─ README.md
└─ LICENSE
```

## Requirements

* **Python**: 3.10 or newer
* **MATLAB**: R2021b or newer, with **MATLAB Engine for Python**
* **OS**: Windows 10/11, macOS 12+, or Linux (Ubuntu 20.04+ tested)

### Python dependencies (partial)

* pandas, numpy, scikit‑learn, scipy, matplotlib
* pyyaml, python‑dotenv, rich, click

Install exact versions from `requirements.txt`.

## Quick Start

```bash
# 1) Clone the repo
git clone https://github.com/<your-user>/<your-repo>.git
cd <your-repo>

# 2) Create and activate a virtual environment
python -m venv .venv
# Windows
.\.venv\Scripts\activate
# macOS/Linux
source .venv/bin/activate

# 3) Install Python deps
pip install -r requirements.txt

# 4) (Optional) Install MATLAB Engine for Python (see details below)

# 5) Run a sample workflow
python -m src.cli run --config configs/default.yaml
```

## Detailed Setup

### 1) Python Environment

1. Install Python 3.10+.
2. Create a virtualenv and activate it.
3. Install dependencies:

   ```bash
   pip install -r requirements.txt
   ```

### 2) MATLAB Engine for Python

If you plan to use MATLAB‑based routines (e.g., `ecg_features.m`), you must install the MATLAB Engine for Python that matches your Python version.

**macOS/Linux**

```bash
# Replace R2024a with your MATLAB version
cd "/Applications/MATLAB_R2024a.app/extern/engines/python"
python setup.py install --user
```

**Windows**

```powershell
# Replace R2024a with your MATLAB version
cd "C:\Program Files\MATLAB\R2024a\extern\engines\python"
python setup.py install
```

> After installation, verify in Python:
>
> ```python
> import matlab.engine
> eng = matlab.engine.start_matlab()
> print(eng.version())
> eng.quit()
> ```

Place `.m` files under `matlab/` and ensure MATLAB can see them by adding the path within Python (handled by `hdlib/matlab_bridge.py`).

### 3) Project Configuration

Copy the example environment file and adjust as needed:

```bash
cp .env.example .env
```

Key settings live in YAML under `configs/`. Override per‑experiment with your own YAML files.

## Usage

The project exposes a CLI for common tasks.

### CLI Help

```bash
python -m src.cli --help
```

### Typical Workflow

```bash
# 1) Prepare data
python -m src.cli data prep --input data/raw/heart.csv --output data/processed/heart.parquet

# 2) Explore (generates EDA report & plots)
python -m src.cli eda --input data/processed/heart.parquet --report reports/eda.html

# 3) Engineer features (optionally call MATLAB for ECG)
python -m src.cli features --input data/processed/heart.parquet --use-matlab true

# 4) Train and evaluate
python -m src.cli train --config configs/experiments/baseline.yaml

# 5) Predict on new data
python -m src.cli predict --model artifacts/best_model.pkl --input data/new.csv --output predictions.csv
```

### Using MATLAB from Python (example)

```python
from hdlib.matlab_bridge import MatlabSession

with MatlabSession() as eng:
    # Calls ecg_features.m and returns a numpy array
    feats = eng.ecg_features("data/ecg/sample.mat")
```

## Reproducing Results

* All random seeds are set via config (see `configs/default.yaml`).
* Datasets are versioned under `data/` (not committed). Provide download links or use a data registry.
* Trained models and metrics are written to `artifacts/` and `reports/`.

## Testing & Quality

```bash
# Run tests
pytest -q

# Lint & format
ruff check .
ruff format .

# Optional: pre-commit hooks
pre-commit install
```

## Troubleshooting

* **`ModuleNotFoundError: matlab.engine`**: Ensure MATLAB Engine is installed for the active Python interpreter. Check `python -V` vs. MATLAB Engine build.
* **Engine start is slow**: The first call spawns MATLAB; consider reusing the session or running batch MATLAB computations.
* **`MATLAB:UndefinedFunction`**: Confirm `.m` files are on the MATLAB path; see `matlab_bridge.py` for `addpath` logic.
* **Compiler toolchain warnings** (Windows): Install supported MSVC Build Tools that match your Python/NumPy wheels.

## Contributing

1. Fork the repo and create a feature branch.
2. Add/modify tests for your change.
3. Run linters and formatters.
4. Open a PR with a clear description and motivation.

## License

This project is licensed under the MIT License. See [LICENSE](LICENSE) for details.
