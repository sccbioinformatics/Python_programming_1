---
title: "Functions and errors"
---

# Goal of this section

By the end of this section you will be able to:

- Define your own functions
- Pass arguments into functions
- Return results from functions
- Handle incorrect input using errors
- Write small reusable functions that work with our `data` dictionary model

Functions let us reuse code and make programs easier to read.

---

# Our data model (reminder)

In previous sections we used a dictionary that stores:

- `data["expression"]` : numpy ndarray (genes × samples)
- `data["genes"]`      : DataFrame (gene metadata)
- `data["samples"]`    : DataFrame (sample metadata, includes `cluster`)

We will now write functions that work with this structure.

---

# Defining a function

A function is a named block of code.

```python
def square(x):
    return x * x
```

Call the function:

```python
square(4)
```

---

# A first useful function: get one gene across all samples

Goal: return the expression values for one gene.

```python
def get_gene(data, gene):
    """
    Return expression values for one gene across all samples.
    """

    g_idx = data["genes"].index.get_loc(gene)
    return data["expression"][:, g_idx]
```

Try it:

```python
get_gene(data, "Gata1")
```

---

# Handling missing genes (raise an error)

What happens if the gene does not exist?

```python
get_gene(data, "NotAGene")
```

Better: check first and raise a helpful error.

```python
def get_gene(data, gene):
    """
    Return expression values for one gene across all samples.
    """
    if gene not in data["genes"].index:
        raise ValueError(
            f"Unknown gene '{gene}'. Expected one of: {list(data["genes"].index)}"
        )

    g_idx = data["genes"].index.get_loc(gene)
    return data["expression"][:, g_idx]
```

Now:

```python
get_gene(data, "NotAGene")
```

---

# Functions with multiple arguments: subset samples by cluster

Goal: get the sample names that belong to a given cluster.

```python
def samples_in_cluster(data, cluster_id):
    """
    Return a list of sample names belonging to cluster_id.
    """
    samples = data["samples"]

    if "cluster" not in samples.columns:
        raise ValueError("data['samples'] must contain a column named 'cluster'.")

    mask = samples["cluster"] == cluster_id
    return list(samples.index[mask])
```

Try it:

```python
samples_in_cluster(data, 1)
```

Excercise:

Create a function 

``samples_in(data, col_name, value ):`` 

that returns the sample names where the column `col_name`
in the sample table equals `value`.

??? Do not peak before yours works ;-)

   ```python
   samples_in(data, col_name, value ):
      """
      Returns a list of samples wher <column_name> equals <value>
      """"
      samples = data["samples"]

      if col_name not in samples.columns:
        raise ValueError(f"data['samples'] must contain a column named '{col_name}'.")
      mask = samples[col_name] == value
      return list(samples.index[mask])
    ````

---

# Returning multiple results

Sometimes we want to return several values.
A simple way is to return a dictionary.

Example: compute mean expression for each gene and each sample.

```python
def mean_expression_summary(data):
    """
    Return mean expression per gene and per sample.
    """
    expr = data["expression"]

    gene_means = []
    for gene in range(expr.shape[0]):
        gene_means.append(expr[gene,:].mean())

    sample_means = []
    for sample in range(expr.shape[1]):
        sample_means.append(expr[:,sample].mean())

    return {
        "gene_means": gene_means,
        "sample_means": sample_means
    }
```

Use it:

```python
summary = mean_expression_summary(data)
summary["gene_means"]
summary["sample_means"]
```

---

# Conditional logic in functions (with useful errors)

Example: choose how to compute a summary.

```python
def summarize_gene(data, gene, method="mean"):
    """
    Summarize one gene across samples using a chosen method.
    method: 'mean' or 'max'
    """
    v = get_gene(data, gene)

    if method == "mean":
        return v.mean()
    elif method == "max":
        return v.max()
    else:
        raise ValueError(
            f"Unknown method '{method}'. Expected 'mean' or 'max'."
        )
```

Try:

```python
summarize_gene(data, "Gata1", method="mean")
summarize_gene(data, "Gata1", method="max")
summarize_gene(data, "Gata1", method="median")
```

---

# Exercise 1: Check the data model

## Goal

Write a function:

```python
check_data_model(data)
```

It should raise a helpful error if:

1. One of the keys is missing:
   - `"expression"`, `"genes"`, `"samples"`

2. The objects are not pandas DataFrames.

3. The indexes do not match the data model:
   - `data["expression"].index` must match `data["genes"].index`
   - `data["expression"].columns` must match `data["samples"].index`

---

??? exercise "Solution: check_data_model"
    ```python
    def check_data_model(data):
        required = ["expression", "samples", "genes"]

        # Check keys
        for key in required:
            if key not in data:
                raise ValueError(f"Missing key '{key}' in data dictionary.")

        expr = data["expression"]
        genes = data["genes"]
        samples = data["samples"]

        # Check types
        import pandas as pd
        if not isinstance(expr, np.ndarray):
            raise ValueError("'expression' must be a numpy ndarray.")
        if not isinstance(genes, pd.DataFrame):
            raise ValueError("'genes' must be a pandas DataFrame.")
        if not isinstance(samples, pd.DataFrame):
            raise ValueError("'samples' must be a pandas DataFrame.")

        # Check index alignment
        if not expr.shape[0] == len(genes.index):
            raise ValueError(
                f"Shape[0] of 'expression' must match index length of 'genes' ({expr.shape[0]} != {len(genes.index)})."
            )

        if not expr.shape[1] == len(samples.index):
            raise ValueError(
                "Shape[1] of 'expression' must match index length of 'samples' ({expr.shape[1]} != {len(samples.index)})."
            )
    ```

---


# Why this matters for bioinformatics

Bioinformatics scripts often:

- take user input (gene names, thresholds, cluster IDs)
- process tables and metadata together
- produce plots and output files

Functions make it possible to:

- structure code
- test small pieces
- reuse analysis steps
- fail early with clear error messages

In the next section, we will learn how to save and load our module savely.
