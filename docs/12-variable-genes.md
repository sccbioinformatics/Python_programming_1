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

# Import libraries

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
def get_top_variable_genes(mat, top_n=500):
    variances = mat.var(axis=1, ddof=1)          # variance per row (gene)
    top_genes = variances.nlargest(top_n).index  # gene names of the largest variances
    return mat.loc[top_genes]
```

Use it:

```python
hspc_var = get_top_variable_genes(hspc_data, top_n=500)
hspc_var.shape
```

---

# Check the value range

```python
hspc_var.values.min(), hspc_var.values.max()
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
    m = mat.mean(axis=1)
    s = mat.std(axis=1, ddof=1)
    return mat.sub(m, axis=0).div(s, axis=0)
```

Run it:

```python
hspc_zs = zscore_rows(hspc_var)
hspc_zs.shape
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

Before:

```python
hspc_var.boxplot(rot=90)
plt.title("Before scaling (raw values)")
plt.show()
```

After:

```python
hspc_zs.boxplot(rot=90)
plt.title("After scaling (z-scores)")
plt.show()
```

---

# Exercise

1. Select the top 200 variable genes into `hspc_var200`.
2. Z-score them into `hspc_zs200`.
3. Verify that row means are ~0 and row standard deviations are ~1.

---

# Why this matters for bioinformatics

Z-scoring makes clustering focus on **patterns** rather than absolute expression levels.

For example, two genes might have different expression magnitudes but similar behaviour across samples:

- gene A: low counts but increases in the same samples as gene B
- gene B: high counts but the same pattern

Z-scoring helps the clustering algorithm see the shared pattern.

In the next section, we will cluster genes and visualise them using dendrograms and heatmaps.
