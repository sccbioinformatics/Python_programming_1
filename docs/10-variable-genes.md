---
title: "Variable genes and z-score scaling"
---

# Goal of this section

By the end of this section you will be able to:

- Select the most variable genes from an expression matrix
- Explain what variance means in biological terms
- Z-score (standardize) gene expression per gene (row-wise)
- Check scaling visually

This is a common preprocessing step before clustering and heatmaps.

---

# Improved start-up

So far we have written many small helper functions directly inside our notebooks.
That works for experiments, but it quickly becomes messy:

* You need to copy-paste functions between notebooks
* It is hard to reuse code
* Mistakes fixed in one notebook do not automatically get fixed in others

A better approach is to store your functions in a **separate Python file** and import them when needed.

---

## Step 1: Create a functions file

In the Jupyter interface:

1. Look at the **file browser panel** on the left.
2. Click the blue **“+”** button.
3. Choose:

   * **Other**
   * **Python File**
4. Name the file:

```
functions.py
```

---

## Step 2: Collect your functions

Open `functions.py` and move all the following into it:

* All `import` statements
* All helper functions you created in previous sections

For example:

```python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt


def dist(vec1, vec2):
    """
    Euclidean distance between two vectors
    """
    diff = vec1 - vec2
    return np.sum(diff**2) ** 0.5


def get_top_variable_genes(data, top_n=500):
    """
    Return top N most variable genes
    """
    var = data.var(axis=1)
    top = var.sort_values(ascending=False).head(top_n)
    return data.loc[top.index]
```

You can keep adding functions to this file as the course continues.

---

## Step 3: Import the functions into your notebook

At the top of your notebook, simply write:

```python
from functions import *
```

This will:

* Load all libraries defined in `functions.py`
* Load all functions you defined there
* Make them available in your notebook

---

## Important note (very useful in Jupyter)

If you change `functions.py`, Jupyter **does not automatically reload it**.

To reload it, run:

```python
import importlib
import functions
importlib.reload(functions)
from functions import *
```

Or simply restart the kernel.

---

## Why this is good practice

This approach:

* Keeps notebooks clean
* Encourages code reuse
* Makes debugging easier
* Is closer to how real projects are structured

Later, this idea naturally grows into:

* Python modules
* Packages
* Reusable analysis libraries



---

# Import libraries

These are the libraries we use in the here described functions.
When you put new function into ``function.py`` make sure that 
all necessary libraries are also loaded in the file.


```python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
```

---

# Reminder: what is our data shape?

For the course data:

- rows = genes
- columns = samples (or cells)

So when we compute variance **per gene**, we operate along `axis=1` (across columns).

---

# Why do we select variable genes?

Most genes do not change much across samples.
Genes with higher variability are more likely to:

- reflect biological differences between populations
- separate sample groups in clustering
- show interesting patterns in heatmaps

So we often focus on the **top N most variable genes**.

---

# Select the top variable genes

```python
def get_top_variable_genes(data, top_n=500):
    mat = data['expression']
    var = mat.var(axis=1, ddof=1)          # variance per row (gene)
    
    # 2) get sorted gene indices (small → large variance)
    sorted_ids = np.argsort(var)

    # return a subset (bottom top_n)
    return subset_genes(data, sorted_ids[-top_n:])
```

Use it:

```python
hspc_var = get_top_variable_genes(hspc_data, top_n=500)
hspc_var['expression'].shape
```

---

# Check the value range

```python
hspc_var['expression'].min(), hspc_var['expression'].max()
```

---

# Z-score scaling (standardization)

For clustering and heatmaps, it is often useful to scale each gene so that:

- mean = 0
- standard deviation = 1

For a gene (row) with values \(x_1, x_2, ..., x_n\), the z-score is:

\[
z_i = \frac{x_i - \mu}{\sigma}
\]

where:
- \(\mu\) is the mean of the gene across samples
- \(\sigma\) is the standard deviation of the gene across samples

This makes genes comparable even if they have very different expression ranges.

---

# A fast z-score function (vectorized)

```python
def zscore_rows(mat):
    m = np.mean(mat, axis=1, keepdims=True)
    s = np.std(mat, axis=1, ddof=1, keepdims=True)
    return (mat - m) / s
```

Run it:

```python
hspc_zs = hspc_data_tiny.copy()
hspc_zs['expression'] = zscore_rows(hspc_data_tiny['expression'])
hspc_zs['expression'].shape
```

---

# Quick sanity check

After z-scoring, each gene should have:

- mean ~ 0
- std ~ 1

Check the first few genes:

```python
print(hspc_zs.mean(axis=1).head())

print(hspc_zs.std(axis=1, ddof=1).head())
```

---

# Visual check: boxplots before and after scaling

There is a boxplot function in pandas, but our dict stores the data as numpy array.
The easiest here is to define one more function that actually converts out dict to a pandas DatFrame - the reverse of our from_df function earlier:

```python
def expression_df(data):
    return pd.DataFrame(
        data["expression"],
        columns=data["samples"].index,
        index=data["genes"].index
    )
```

Before z-scoring:

```python
expression_df(hspc_var).T.boxplot(rot=90)
plt.title("Before scaling (raw values)")
plt.show()
```

After:

```python
expression_df(hspc_zs).T.boxplot(rot=90)
plt.title("After scaling (z-scores)")
plt.show()
```

---

# Exercise

We used ``expression_df(hspc_var).T`` there, but we likely also need a transform for our own data.

 1. Implement a ``def transpose()`` that returns a transformed data structure. 
 2. Change the ``zscore_rows()`` function to accept our dict. 
    Also change the function to zscore in place. We could store the mean and std per gene in the data dist, too.

---

# Why this matters for bioinformatics

Z-scoring makes clustering focus on **patterns** rather than absolute expression levels.

For example, two genes might have different expression magnitudes but similar behaviour across samples:

- gene A: low counts but increases in the same samples as gene B
- gene B: high counts but the same pattern

Z-scoring helps the clustering algorithm see the shared pattern.

In the next section, we will cluster genes and visualise them using dendrograms and heatmaps.
