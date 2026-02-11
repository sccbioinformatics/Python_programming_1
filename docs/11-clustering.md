---
title: "Clustering and heatmaps"
---

# Goal of this section

By the end of this section you will be able to:

- Perform hierarchical clustering on genes and samples
- Understand what a dendrogram represents
- Create heatmaps of expression data
- Use `seaborn.clustermap` for a high-level “all-in-one” clustering plot

We will use the scaled variable-gene matrix from the previous section:
- `hspc_var` (variable genes)
- `hspc_zs` (z-scored per gene)


# Improved start-up

So far we have written many small helper functions directly inside our notebooks.
That works for experiments, but it quickly becomes messy:

* You need to copy-paste functions between notebooks
* It is hard to reuse code
* Mistakes in one notebook do not automatically get fixed in others

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


```python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns

import scipy.cluster.hierarchy as sch
```

# Get the Data

```python
url = "https://raw.githubusercontent.com/shambam/R_programming_1/main/Mouse_HSPC_reduced.txt"
hspc_data = pd.read_csv(
    url,
    sep="\t",
    header=0,
    index_col=0
)
hspc_data = from_pd(hspc_data )


hspc_var = get_top_variable_genes(hspc_data, top_n=500)
hspc_zs = hspc_var.copy()
# remember Excercise in Performace? Use the function you created there.
zscore_rows(hspc_zs)
```


---

# Hierarchical clustering (genes)

Hierarchical clustering groups similar genes together.

We need a distance between rows (genes). A common choice is Euclidean distance.
Then we build a hierarchy using a linkage method.
Ward is a linkage method that starts with each gene as its own cluster.

At every step, it looks at all pairs of clusters, computes the distance between their centers using Euclidean distance, and asks: if we merge these two clusters, how much would the total variance inside the cluster increase?
It then merges the pair that produces the smallest increase in within-cluster variance, which leads to compact, tight clusters. 

```python
gene_dist = vectorized_dist(hspc_zs, metric="euclidean")
gene_link = sch.linkage(gene_dist, method="ward")
```

Plot a dendrogram (for small numbers of genes this is fine):

```python
plt.figure(figsize=(10, 6))
sch.dendrogram(gene_link, labels=hspc_zs.index.tolist(), leaf_rotation=90)
plt.title("Hierarchical clustering (genes)")
plt.xlabel("Gene")
plt.ylabel("Distance")
plt.tight_layout()
plt.show()
```

Note: With hundreds of genes, dendrogram labels can become unreadable. That’s normal.

---

# Hierarchical clustering (samples)

To cluster samples, transpose the matrix so samples become rows:

```python
sample_dist = vectorized_dist(hspc_zs.transformed() )
sample_link = sch.linkage(sample_dist, method="ward")
```

```python
plt.figure(figsize=(10, 6))
sch.dendrogram(sample_link, labels=hspc_zs.['samples'].tolist(), leaf_rotation=90)
plt.title("Hierarchical clustering (samples)")
plt.xlabel("Sample")
plt.ylabel("Distance")
plt.tight_layout()
plt.show()
```

---

# A first heatmap (no clustering)

```python
plt.figure(figsize=(8, 6))
plt.imshow(hspc_zs.values, aspect="auto")
plt.title("Heatmap (no clustering)")
plt.xlabel("Samples")
plt.ylabel("Genes")
plt.show()
```

This shows the data but doesn’t order genes/samples by similarity.

---

# Ordering rows/columns using the dendrogram

We can extract the ordering from the dendrogram output and reorder the matrix.

```python
genes_hc = sch.dendrogram(gene_link, no_plot=True, labels=hspc_zs.index.tolist())
samples_hc = sch.dendrogram(sample_link, no_plot=True, labels=hspc_zs.columns.tolist())

ordered = expression_df(hspc_zs).loc[genes_hc["ivl"], samples_hc["ivl"]]
ordered.shape
```

Plot the ordered heatmap:

```python
plt.figure(figsize=(8, 6))
plt.imshow(ordered.values, aspect="auto")
plt.title("Heatmap (genes and samples ordered by clustering)")
plt.xlabel("Samples")
plt.ylabel("Genes")
plt.show()
```

---

# Heatmap with seaborn

Seaborn produces nicer heatmaps (and supports colorbars, labels, etc.).

```python
plt.figure(figsize=(10, 8))
sns.heatmap(ordered, cmap="magma")
plt.title("Heatmap (ordered)")
plt.show()
```

---

# Clustermap: clustering + heatmap in one step

`seaborn.clustermap` does clustering and plotting together.
We use `z_score=0` to z-score rows (genes) inside clustermap.
(If you already z-scored, you can set `z_score=None`.)

```python
cm = sns.clustermap(
    hspc_var,
    method="ward",
    metric="euclidean",
    cmap="magma",
    z_score=0
)
plt.show()
```

Save it:

```python
cm.fig.savefig("clustermap.png", dpi=200)
```

---

# Assigning genes to clusters (example: 5 clusters)

Sometimes we want to label each gene with a cluster ID.

```python
gene_clusters_k5 = sch.fcluster(cm.dendrogram_row.linkage, t=5, criterion="maxclust")
gene_clusters_k5[:10]
```

Count genes per cluster:

```python
counts = pd.Series(gene_clusters_k5).value_counts().sort_index()
counts
```

Plot counts:

```python
counts.plot(kind="bar")
plt.title("Genes per cluster (k=5)")
plt.xlabel("Cluster")
plt.ylabel("Number of genes")
plt.show()
```

---

# Exercise

You have seen how you can create heatmaps for a pandas DataFrame.
Please implement a function for our own data dictionary that does plot a heatmap.
**Hint** 
- `*args` collects extra positional arguments into a tuple
- `**kwargs` collects extra named arguments into a dictionary

**How to use `*args` and `**kwargs`**

You can document them in the function like this:

```python
def plot_heatmap(data, *args, **kwargs):
    """
    Plot a heatmap of the expression matrix.

    Parameters
    ----------
    data : dict
        Data dictionary containing the key 'expression'.
    *args :
        Extra positional arguments passed to seaborn.clustermap.
    **kwargs :
        Extra keyword arguments passed to seaborn.clustermap.
    """
```

Inside the function, you simply forward them to seaborn:

```python
sns.clustermap(mat, *args, **kwargs)
```

This passes all additional arguments directly to `clustermap` without you needing to specify them one by one.

You can and should also add options that your function processes itself, for example a `save` argument to write the image to a file, or options like `title`, `xlabel`, and `ylabel` to control the plot labels.
A good rule is: handle the most common options directly in your function, and pass everything else through `*args` and `**kwargs` to keep the function flexible without becoming overly complicated.


---

# Why this matters for bioinformatics

Clustering + heatmaps are standard tools to:

- discover gene programs
- see sample relationships
- identify outliers
- validate expected biology (e.g. known cell types)

In the next section, we will introduce a simple statistical model (ANOVA) and compute p-values across genes.
