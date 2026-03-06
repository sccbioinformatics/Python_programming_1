---
title: "Dictionaries (key-value data)"
---

# Goal of this section

By the end of this section you will be able to:

- Create dictionaries in Python
- Store multiple related objects together
- Access dictionary elements by name
- Understand why dictionaries are useful for biological data
- Use a dictionary-based dataset to make a gene–gene plot for one cluster

---

# What is a dictionary?

A **dictionary** stores data as:

```
key -> value
```

Example:

```python
gene_info = {
    "name": "Gata1",
    "chromosome": "chrX",
    "length": 2345
}

gene_info
```

You access values using the key:

```python
gene_info["name"]
gene_info["chromosome"]
```

---

# Dictionaries vs lists

A list stores values by position:

```python
genes = ["Gata1", "Runx1", "Spi1"]
genes[0]
```

A dictionary stores values by name:

```python
genes = {
    "erythroid": "Gata1",
    "stem": "Runx1",
    "myeloid": "Spi1"
}

genes["erythroid"]
```

This is often clearer when working with biological data where information is paired with metadata.

---

# A mini experiment dataset (stored in one dictionary)

In bioinformatics we often want to keep **multiple related tables together**, for example:

- an expression matrix (genes × samples)
- a gene table (gene metadata)
- a sample table (sample metadata, e.g. clusters)

We will store all of these in a single dictionary called `data`.

```python
import numpy as np
import pandas as pd

# ---- expression (genes x samples) ----
expr = np.array(
    [[8, 7, 2, 3, 9, 1, 2, 8],  # Gata1
     [1, 2, 7, 8, 1, 6, 7, 2],  # Spi1
     [4, 4, 5, 5, 4, 6, 6, 4],  # Runx1
    ],dtype=float
)

# ---- gene metadata table ----
genes = pd.DataFrame(
    {
        "symbol": ["Gata1", "Spi1", "Runx1"],
        "role":   ["erythroid", "myeloid", "stem"],
    },
    index=["Gata1", "Spi1", "Runx1"]
)

# ---- sample metadata table (store clusters here) ----
samples = pd.DataFrame(
    {
        "cluster": [0, 0, 1, 1, 0, 1, 1, 0],
        "type":    ["A", "A", "B", "B", "A", "B", "B", "A"],
    },
    index=["sample1", "sample2", "sample3", "sample4",
           "sample5", "sample6", "sample7", "sample8"]
)

# ---- store everything together ----
data = {
    "expression": expr,
    "genes": genes,
    "samples": samples
}

data
```

Notes:

- `data["expression"]` is the expression matrix (genes × samples)
- `data["genes"]` is gene metadata (one row per gene)
- `data["samples"]` is sample metadata (one row per sample), including **cluster**

---

# Accessing dictionary elements

Example: get the expression matrix

```python
data["expression"]
```

Get the sample table

```python
data["samples"]
```

Combine dictionary access and DataFrame indexing:

```python
data["expression"].loc["Gata1", :]
```

But this does not work like that - does it?

Remember what ``data["expression"]`` is - a ndarray that is very efficient for calculations.
But it lacks row and column names. We have stored row and column data in ``genes`` and ``samples``. Actually the gene names are stored in the rownames of the genes object.

```python
print (data['genes'].index)
data['genes'].index == "Gata1"
```

Do you remember how to subset a ndarray using a boolean array? Try it!

??? exercise "Solution:"
    ```python
    data['expression'][ data['genes'].index == "Gata1" ]
    ```

---

# Modifying a dictionary

Add a new entry:

```python
data["species"] = "mouse"
data
```

Replace an entry (example):

```python
data["samples"]["type"] = ["A", "A", "B", "B", "A", "B", "B", "A"]
data["samples"]
```

---


# Why this structure is useful in bioinformatics

Many packages store information in named containers:

- **AnnData / Scanpy**: `adata.X`, `adata.obs`, `adata.var`, `adata.uns`
- **Seurat** stores data and metadata in named slots
- many file formats store data + metadata + settings

A dictionary is a simple way to model the same idea:  
**named pieces of data stored together**.

---

# Exercise: Gene–gene plot for one cluster

## Goal

Create a scatter plot for **cluster 1**:

- x-axis: expression of **Gata1**
- y-axis: expression of **Spi1**
- points: samples that belong to **cluster 1**

Use the dataset stored in `data`.

---

## Hints

### 1) Access an entry from a dictionary (reminder)

```python
dict1 = {"a": 10, "b": 20}
dict1["a"]
```

### 2) Get the cluster labels from the sample table

- The cluster labels are stored in the **sample metadata** table.
- You will need something like: “give me the `cluster` column”.

### 3) Create a mask for cluster 1

Example pattern:

```python
v = np.array([0, 1, 1, 0])
mask = v == 1
mask
```

### 4) Subset the expression matrix by columns

You have seen column subsetting in the DataFrame section.

Remember the idea:

- rows = genes
- columns = samples

Select only the sample columns that match your mask.

### 5) Extract the two genes

You need one vector for x (Gata1) and one for y (Spi1).

### 6) Scatter plot reminder

```python
import matplotlib.pyplot as plt
plt.scatter(x_values, y_values)
plt.xlabel("...")
plt.ylabel("...")
plt.title("...")
plt.show()
```

---

# Why this matters for bioinformatics

In real single-cell workflows you often:

- store expression + sample metadata together
- plot gene vs gene for a subset of cells (e.g. one cluster)
- iterate quickly over different genes and clusters

In the next section, we will learn how to read and write data from files.
