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

**And our own functions:**

```python
import importlib
import functions
importlib.reload(functions)
from functions import *
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
groups = pd.Series(hspc_data['samples'].index).str.replace(r"\..*", "", regex=True)
groups
```

Here, `.str` gives access to **vectorized string operations** for every entry in the pandas Series.
Instead of looping over each sample name, pandas applies the string method to all entries at once, which is shorter and much faster.

The `replace` used here is a **pandas string method**, not the normal Python string method.
The `.str` accessor tells pandas to apply the string operation to every element in the Series, so `str.replace` works on the whole column at once instead of a single string.

A slower, manual way would be to loop over the sample names and process each string one by one:

```python
import re
groups = [re.sub(r"\..*", "", name) for name in hspc_data['samples'].index]
```

This applies a regular expression replacement to each sample name one by one, which works the same way but is usually slower and less convenient than the vectorized pandas `.str` methods.


---

# Fit one model for one gene

Pick a gene (first row) and build a small DataFrame for modelling:

```python
df = pd.DataFrame({
    "expression": hspc_data['expression'][0, :],
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

|           |  sum_sq  | df | F     | PR(>F) |
|-----------|----------|----|-------|--------|
| group     | 26.44    | 2  | 1.50  | 0.227  |
| Residual  | 1295.18  | 6  |       |        |

Meaning:

| Column   | Meaning |
|----------|---------|
| `sum_sq` | how much variation is explained |
| `df`     | degrees of freedom |
| `F`      | F-statistic (signal / noise) |
| `PR(>F)` | p-value |


**What does “Residual” mean?**
`Residual` is the variation **not explained by the group labels** — the within-group scatter (noise) that remains after fitting different group means. In ANOVA, the F-statistic is essentially **explained variation (group) divided by unexplained variation (residual)**, so large residual variation makes it harder to detect real group differences.


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

- small p-value (e.g. << 0.05) → gene is likely **differentially expressed**
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
data= expression_df(hspc_data)
n_genes = data.shape[0]
pvals = [0.0] * n_genes

for i in range(n_genes):
    vals = hspc_data.iloc[i, :].values
    pvals[i] = do_anova(vals, groups.values)

pvals = pd.Series(pvals, index=hspc_data.index)
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
}, index=pvals.index)

print(f"{sum(reject)} p values now fail to impress")

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
2. Make a heatmap of significant genes for your chosen threshold - use your own function.
3. Create a ``run_anova`` function for our dictionary.


---

# Why this matters for bioinformatics

Differential expression and many “gene tests” follow this idea:

- build a design (groups/conditions)
- fit a model per gene
- correct p-values
- interpret significant genes

You now have the core workflow.

Next section (optional) can cover an algorithmic topic (e.g., k-means + a final project).
