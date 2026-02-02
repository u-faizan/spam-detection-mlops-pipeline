
# MLOps Pipeline & Experiment Workflow

This document outlines the standard workflow for building, running, and tracking experiments using DVC and Git.

## 📁 1. Project Initialization
**Goal:** Set up the repository structure and version control.

1.  **Repo Setup:** Create a GitHub repository and clone it locally.
2.  **Source Code:** Create a `src/` folder and ensure all Python components (ingestion, preprocessing, etc.) run individually without errors.
3.  **Git Ignore:** Create a `.gitignore` file and strictly ignore data and artifact folders:
    ```text
    data/
    models/
    reports/
    __pycache__/
    *.pyc
    .DS_Store
    ```
4.  **Initial Commit:**
    ```bash
    git add .
    git commit -m "feat: initial project structure"
    git push
    ```

---

## 🚀 2. DVC Pipeline Setup (Basic)
**Goal:** Automate the script execution order using a DVC DAG.

1.  **Define Stages:** Create `dvc.yaml` and define your stages (`data_ingestion`, `data_preprocessing`, etc.).
2.  **Initialize DVC:**
    ```bash
    dvc init
    ```
3.  **Test Pipeline:**
    ```bash
    dvc repro
    ```
4.  **Visualize DAG:** Check if dependencies are linked correctly:
    ```bash
    dvc dag
    ```
5.  **Commit DVC Files:**
    ```bash
    git add dvc.yaml dvc.lock
    git commit -m "chore: setup basic dvc pipeline"
    git push
    ```

---

## ⚙️ 3. Parameterization
**Goal:** Control experiment settings from a single file without changing code.

1.  **Create Params:** Create a `params.yaml` file in the root directory.
2.  **Update Python Scripts:** Add the `load_params` function (see [Code Snippets](#code-snippets)) to your scripts.
3.  **Update dvc.yaml:** Add the `params:` section to each stage in `dvc.yaml`.
4.  **Test:** Run the pipeline to ensure it reads the parameters:
    ```bash
    dvc repro
    ```
5.  **Commit:**
    ```bash
    git add params.yaml src/
    git commit -m "feat: integrate params.yaml"
    git push
    ```

---

## 🧪 4. Experiment Tracking (DVCLive)
**Goal:** Track metrics and parameters for every run automatically.

1.  **Install:**
    ```bash
    pip install dvclive
    ```
2.  **Integrate:** Add the `Live` context manager to your evaluation script (see [Code Snippets](#code-snippets)).
3.  **Run Experiment:**
    ```bash
    dvc exp run
    ```
    * *Note:* This will execute the pipeline and generate a new entry in the experiment table.
4.  **View Results:**
    * **Terminal:** `dvc exp show`
    * **VS Code:** Use the DVC Extension to view plots and metrics tables.
5.  **Manage Experiments:**
    * **Remove:** `dvc exp remove {exp-name}`
    * **Restore:** `dvc exp apply {exp-name}`
6.  **Iterate:**
    * Change values in `params.yaml`.
    * Run `dvc exp run` again.
    * Compare results.

---

## 📝 Code Snippets

### A. Loading Parameters (Python)
Add this helper function to your scripts to read `params.yaml` safely.

```python
import yaml
import logging

def load_params(params_path: str) -> dict:
    """Load parameters from a YAML file."""
    try:
        with open(params_path, 'r') as file:
            params = yaml.safe_load(file)
        logging.debug('Parameters retrieved from %s', params_path)
        return params
    except FileNotFoundError:
        logging.error('File not found: %s', params_path)
        raise
    except yaml.YAMLError as e:
        logging.error('YAML error: %s', e)
        raise
    except Exception as e:
        logging.error('Unexpected error: %s', e)
        raise

# --- Usage Examples ---

# In data_ingestion.py
params = load_params(params_path='params.yaml')
test_size = params['data_ingestion']['test_size']

# In feature_engineering.py
params = load_params(params_path='params.yaml')
max_features = params['feature_engineering']['max_features']

```

### B. DVCLive Integration

Use this block in `src/model_evaluation.py` to log metrics and parameters.

```python
from dvclive import Live
from sklearn.metrics import accuracy_score, precision_score, recall_score

# 1. Load params
params = load_params('params.yaml')

# 2. Calculate predictions
y_pred = clf.predict(X_test)

# 3. Log to DVC
with Live(save_dvc_exp=True) as live:
    # Log Metrics
    live.log_metric('accuracy', accuracy_score(y_test, y_pred))
    live.log_metric('precision', precision_score(y_test, y_pred, average='weighted'))
    live.log_metric('recall', recall_score(y_test, y_pred, average='weighted'))

    # Log Parameters (Connects config to results)
    live.log_params(params)

```
