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

---

# Import libraries

```python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns

import scipy.cluster.hierarchy as sch
import scipy.spatial.distance as ssd
```

---

# Hierarchical clustering (genes)

Hierarchical clustering groups similar genes together.

We need a distance between rows (genes). A common choice is Euclidean distance.
Then we build a hierarchy using a linkage method (Ward is popular for expression data).

```python
gene_dist = ssd.pdist(hspc_zs.values, metric="euclidean")
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
sample_dist = ssd.pdist(hspc_zs.T.values, metric="euclidean")
sample_link = sch.linkage(sample_dist, method="ward")
```

```python
plt.figure(figsize=(10, 6))
sch.dendrogram(sample_link, labels=hspc_zs.columns.tolist(), leaf_rotation=90)
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

ordered = hspc_zs.loc[genes_hc["ivl"], samples_hc["ivl"]]
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

1. Create a clustermap using correlation distance instead of Euclidean:
   - hint: `metric="correlation"`

2. Try different linkage methods:
   - `"average"`
   - `"complete"`
   - `"ward"` (note: Ward is typically paired with Euclidean)

3. For your favorite plot, save it to a PNG file.

---

# Why this matters for bioinformatics

Clustering + heatmaps are standard tools to:

- discover gene programs
- see sample relationships
- identify outliers
- validate expected biology (e.g. known cell types)

In the next section, we will introduce a simple statistical model (ANOVA) and compute p-values across genes.
