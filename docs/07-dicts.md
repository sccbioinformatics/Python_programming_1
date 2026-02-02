---
title: "Dictionaries (key-value data)"
---

# Goal of this section

By the end of this section you will be able to:

- Create dictionaries in Python
- Store multiple related objects together
- Access dictionary elements by name
- Understand why dictionaries are useful for biological data

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

This is often clearer when working with biological data where the raw information is paired with additional metadata.


---

# Storing biological data together

In bioinformatics, we often want to store:

- gene names
- expression matrix
- cluster assignments

together in one object.

Example:

```python
import numpy as np
import pandas as pd

gene_names = ["gene1", "gene2", "gene3"]
expression = pd.DataFrame(
    np.random.randn(3, 4),
    index=gene_names,
    columns=["sample1", "sample2", "sample3", "sample4"]
)

clusters = [0, 1, 0]

experiment = {
    "genes": gene_names,
    "expression": expression,
    "clusters": clusters
}

experiment
```

---

# Accessing dictionary elements

```python
experiment["genes"]
```

```python
experiment["expression"]
```

```python
experiment["clusters"]
```

You can also combine indexing:

```python
experiment["expression"].iloc[0, :]
```

---

# Modifying dictionaries

Add a new entry:

```python
experiment["condition"] = "treated"
experiment
```

Replace an entry:

```python
experiment["clusters"] = [1, 1, 0]
experiment
```

---

# Looping over a dictionary

```python
for key in experiment:
    print(key)
```

Keys and values:

```python
for key, value in experiment.items():
    print(key, type(value))
```

---

Python packages like scanpy also store a lot of the informations in dictionaries. The ``uns`` slot int an anndata objects is a dictionary and containe e.g. color information for the grouping or in ``uns_m`` the neighbor information needed for the UMAP algorithm.

# Exercise

1. Create a dictionary called `dataset` with:
   - key `"genes"` containing a list of gene names
   - key `"matrix"` containing a random DataFrame
2. Add a key `"species"` with value `"mouse"`.
3. Print only the expression matrix from the dictionary.
4. Print the first row of the matrix using dictionary access.

---

# Why this matters for bioinformatics

Many real-world data structures in bioinformatics are dictionaries:

- AnnData objects (Scanpy)
- Seurat objects (R)
- JSON metadata files
- configuration files

They all follow the idea of:
**named pieces of data stored together**.

In the next section, we will learn how to read and write data from files.
