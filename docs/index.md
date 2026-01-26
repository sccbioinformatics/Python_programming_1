---
title: "Setup"
---

# Goal of this section

By the end of this section you will be able to:

- Install Python in a **reproducible environment**
- Start JupyterLab and run code cells
- Know what an “environment” is and why we use it in bioinformatics

## Why environments matter

In bioinformatics, results must be reproducible. If you and a collaborator install “Python + packages” on different days, you can end up with different package versions and different results.

A **conda environment** is a self-contained box that holds Python plus exactly the packages we choose.

---

# Install Miniconda

Install Miniconda from the official instructions:

- Miniconda download + install guide: https://www.anaconda.com/docs/getting-started/miniconda/main

After installation, open a terminal.

---

# Create and activate the course environment

Create an environment (one time):

```bash
conda create -n quarto.env python=3.10
```

Activate it:

```bash
conda activate quarto.env
```

From now on, whenever you work on this course, start by activating:

```bash
conda activate quarto.env
```

---

# Install the packages we’ll use

```bash
conda install -c conda-forge \
  jupyterlab notebook nbclient ipykernel \
  numpy pandas matplotlib seaborn scipy statsmodels scikit-learn
```

Notes:
- `numpy` + `pandas`: data handling
- `matplotlib` + `seaborn`: plotting
- `scipy`: clustering and distances
- `statsmodels`: linear models / ANOVA
- `scikit-learn`: standard ML tools (we’ll use it to compare with “from-scratch” ideas)

---

# Register the environment as a Jupyter kernel

This makes the environment appear inside Jupyter as a selectable “kernel”:

```bash
python -m ipykernel install --user --name quarto-env --display-name "Python (quarto.env)"
```

**Important:** the kernel name here is `quarto-env`. We’ll match this to your `_quarto.yml` later.

---

# Start JupyterLab

Run:

```bash
jupyter lab
```

A browser window should open. Create a new notebook using the kernel:

**Python (quarto.env)**

---

# Choosing an editor (later)

For notebooks, JupyterLab is great. For writing reusable code (scripts/packages), use an IDE:

- **VS Code** (recommended): general purpose, very popular
- **PyCharm**: powerful Python IDE, lots of features

For this course: we’ll mostly use JupyterLab, and optionally VS Code later.

---

# Quick check

Run this in a notebook cell:

```python
import numpy as np
import pandas as pd

print(np.arange(1, 6))
print(pd.DataFrame({"a": [1, 2], "b": [3, 4]}))
```

If this runs without errors, you’re ready.
