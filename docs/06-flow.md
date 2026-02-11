---
title: "Flow control (loops)"
---

# Goal of this section

By the end of this section you will be able to:

- Use `for` loops to repeat operations
- Use `while` loops for conditional repetition
- Store results produced inside loops
- Use loops to work with our dictionary-based dataset
- Understand why loops can be slow for large datasets (and when to avoid them)

---

# Import libraries

```python
import numpy as np
import pandas as pd
```

---

# Our example dataset

In previous sections, we created a dataset stored in a dictionary:

```python
# data = {
#     "expression": expr,   # genes x samples (DataFrame)
#     "genes": genes,       # gene metadata (DataFrame)
#     "samples": samples    # sample metadata (DataFrame, includes cluster)
# }
```

We will use:

- `data["expression"]`  (genes × samples)
- `data["samples"]`     (sample metadata, including cluster)

---

# For loops

A `for` loop repeats an action for each value in a sequence.

```python
for i in range(5):
    print(i)
```

This prints numbers from 0 to 4.

---

# Looping over gene names

Print each gene name from the expression table:

```python
for gene in data["genes"].index:
    print(gene)
```

---

# Looping over sample names

Print each sample name:

```python
for sample in data["samples"].index:
    print(sample)
```

---

# Looping over rows in a DataFrame (by position)

Print the first 2 gene rows (as a reminder of indexing):

```python
expr = data["expression"]

for i in range(2):
    print(expr[i, :])
```

---

# Step size in loops

Print every second sample name:

```python
samples = list(data["samples"].index)

for i in range(0, len(samples), 2):
    print(samples[i])
```

---

# Storing results from loops

Data created inside a loop disappears unless we store it.
We often store results in a list.

```python
values = []

for i in range(5):
    values.append(i * 2)

values
```

---

# Example: compute gene means using a loop

Compute the mean expression of each gene (across all samples) using a loop:

```python
expr = data["expression"]

gene_means = []

for gene in range(expr.shape[0]):
    m = expr[gene].mean()
    gene_means.append(m)

gene_means
```

This works, but for large datasets it can be slow.

(We will later learn faster ways, but loops are important for understanding the logic.)

---

# Example: compute sample means using a loop

Compute the mean expression of each sample (across genes):

```python
expr = data["expression"]

sample_means = []

for sample in range(expr.shape[1]):
    m = expr[:, sample].mean()
    sample_means.append(m)

sample_means
```

---

# Example: work with clusters inside a loop

The sample metadata includes cluster IDs:

```python
data["samples"]
```

Goal: collect the sample names that belong to cluster 1.

```python
cluster1_samples = []

for sample in data["samples"].index:
    if data["samples"].loc[sample, "cluster"] == 1:
        cluster1_samples.append(sample)

cluster1_samples
```

---

# While loops

A `while` loop runs until a condition becomes false.

Example:

```python
w = 0
while w <= 5:
    print(w)
    w = w + 1
```

---

# While loop example: count until we find a sample from cluster 1

This shows how `while` can stop early when a condition is met.

```python
cluster1_samples = []
i = 0

while i+1 < len(data["samples"]):
    if data["samples"]["cluster"].iloc[i] == 1:
        cluster1_samples.append(data["samples"].index[i])
    i = i + 1

cluster1_samples
```

---

# When to use loops

Loops are useful for:

- printing values
- building logic step by step
- teaching how algorithms work
- working with metadata and conditions (like clusters)

They are NOT ideal for:

- numerical operations on large matrices (where vectorized operations are faster)

---

# Exercise

## Part A (warm-up)

1. Use a `for` loop to print all gene names in `data`.
2. Use a `for` loop to print all sample names in `data`.

## Part B (store results)

3. Use a loop to compute the mean expression of each gene.
4. Store the results in the object.
5. Plot the mean expression of all genes.

## Part C (clusters)

6. Use a loop to create a list called `cluster0_samples` with all sample names where cluster is 0.
7. Print `cluster0_samples`.

---

# Why this matters for bioinformatics

Many algorithms (clustering, optimisation, simulations) are based on loops.
Understanding flow control lets you:

- follow algorithm logic
- debug code
- write your own simple algorithms

In the next section, we will turn loop-based code into reusable functions.