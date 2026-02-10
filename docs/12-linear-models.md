---
title: "Linear models and ANOVA (per-gene testing)"
---

# Goal of this section

By the end of this section you will be able to:

- Create a sample-group vector from column names
- Fit a simple linear model using `statsmodels`
- Perform an ANOVA and extract a p-value
- Correct many p-values using Benjamini–Hochberg (FDR)
- Extract significant genes and visualize them as a heatmap

This section mirrors a very common bioinformatics pattern:
**fit a model per gene → collect p-values → correct → select genes**.

---

# Import libraries

```python
import numpy as np
import pandas as pd
import seaborn as sns
import matplotlib.pyplot as plt

import statsmodels.api as sm
import statsmodels.formula.api as smf
from statsmodels.stats.multitest import multipletests
```

---

# Create a group label per sample

Your expression matrix is typically:

- rows = genes
- columns = samples (often with informative names)

Example: if columns look like:

```
stem.1  stem.2  prog.1  prog.2
```

we can extract group labels by removing everything after the dot.

```python
groups = pd.Series(hspc_data.columns).str.replace(r"\..*", "", regex=True)
groups
```

---

# Fit one model for one gene

Pick a gene (first row) and build a small DataFrame for modelling:

```python
df = pd.DataFrame({
    "expression": hspc_data.iloc[0, :].values,
    "group": groups.values
})

df
```

Fit a linear model and run ANOVA:

```python
model = smf.ols("expression ~ group", data=df).fit()
anova_table = sm.stats.anova_lm(model, typ=2)
anova_table
```

Extract the p-value for the group effect:

```python
p_value = anova_table["PR(>F)"]["group"]
p_value
```

## What are we actually doing here? (ANOVA explained)

```python
model = smf.ols("expression ~ group", data=df).fit()
anova_table = sm.stats.anova_lm(model, typ=2)
anova_table
```

This fits a **linear model** where gene expression is explained by group membership:

\[
\text{expression} = \mu + \text{group effect} + \text{error}
\]

In words:
- each sample belongs to a biological group (e.g. `stem` or `prog`)
- we ask whether the **average expression differs between groups**

---

### What does ANOVA do?

ANOVA (Analysis of Variance) splits the total variation in expression into:

1. variation **between groups**
2. variation **within groups** (noise)

It then computes an **F-statistic**:

\[
F = \frac{\text{variation between groups}}{\text{variation within groups}}
\]

If groups differ strongly, this ratio will be large.

---

### What does the ANOVA table show?

When you print:

```python
anova_table
```

you see a table like this:

|           | sum_sq | df | F     | PR(>F) |
|-----------|--------|----|-------|--------|
| group     | 12.34  | 1  | 5.67  | 0.031  |
| Residual  | 20.10  | 6  |       |        |

Meaning:

| Column   | Meaning |
|----------|---------|
| `sum_sq` | how much variation is explained |
| `df`     | degrees of freedom |
| `F`      | F-statistic (signal / noise) |
| `PR(>F)` | p-value |

---

### Extracting the p-value

```python
p_value = anova_table["PR(>F)"]["group"]
p_value
```

This gives the p-value for the **group effect**.

It answers the question:

> If there were really no difference between groups,  
> how likely would it be to observe a difference this large just by chance?

---

### Biological interpretation

For one gene:

- small p-value (e.g. < 0.05) → gene is likely **differentially expressed**
- large p-value → no evidence for a group difference

The null hypothesis is:

\[
H_0: \text{mean expression is the same in all groups}
\]

---

# Turn this into a function

We’ll write a function that:
- takes expression values for a single gene
- takes group labels
- returns the p-value

```python
def do_anova(vals, grps):
    df = pd.DataFrame({
        "expression": vals,
        "group": grps
    })
    model = smf.ols("expression ~ group", data=df).fit()
    anova_table = sm.stats.anova_lm(model, typ=2)
    return anova_table["PR(>F)"]["group"]
```

Test it:

```python
do_anova(hspc_data.iloc[0, :].values, groups.values)
```

---

# Apply to all genes (many tests)

For each gene (row), we compute one p-value.

This is not as easily vectorized as mean/variance because model fitting is complex.
So a row-wise loop (or `apply`) is acceptable here.

```python
pvals = hspc_data.apply(lambda row: do_anova(row.values, groups.values), axis=1)
pvals.head()
```

---

# Multiple testing correction (FDR)

When testing thousands of genes, we must correct p-values.
A standard approach is Benjamini–Hochberg FDR control.

```python
reject, pvals_fdr, _, _ = multipletests(pvals.values, alpha=0.05, method="fdr_bh")

pvals_df = pd.DataFrame({
    "pval": pvals.values,
    "pval_fdr": pvals_fdr,
    "significant_0_05": reject
}, index=hspc_data.index)

pvals_df.head()
```

---

# Select significant genes

Example threshold:

```python
sig_genes = pvals_df.index[pvals_df["pval_fdr"] < 0.0005]
len(sig_genes)
```

Subset the expression matrix:

```python
sig_data = hspc_data.loc[sig_genes]
sig_data.shape
```

---

# Visualize significant genes (heatmap)

```python
sns.clustermap(
    sig_data,
    method="ward",
    metric="euclidean",
    cmap="magma",
    z_score=0
)
plt.show()
```

---

# Exercise

1. Change the FDR threshold until you get around 500 significant genes.
2. Make a heatmap of significant genes for your chosen threshold.

---

# Performance note (important)

Per-gene modelling can be slow because we fit thousands of models.

Ways to speed this up (later, when you need it):
- run on fewer genes (e.g., variable genes only)
- parallelize across CPU cores
- use specialized packages (e.g., limma/DESeq2 in R for count models)

For this course, the goal is understanding the modelling pattern.

---

# Why this matters for bioinformatics

Differential expression and many “gene tests” follow this idea:

- build a design (groups/conditions)
- fit a model per gene
- correct p-values
- interpret significant genes

You now have the core workflow.

Next section (optional) can cover an algorithmic topic (e.g., k-means + a final project).
