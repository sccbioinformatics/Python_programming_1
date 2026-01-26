---
title: "NumPy essentials"
---

# Goal of this section

By the end of this section you will be able to:

- Create numeric vectors using NumPy
- Perform vectorized mathematical operations
- Use logical conditions to select values
- Find overlaps between vectors

These operations are the foundation of almost all numerical bioinformatics work.

---

# Import NumPy

Always start by importing NumPy with its standard alias:

```python
import numpy as np
```

---

# Creating vectors

The most common way to generate a numeric sequence is `arange`:

```python
x = np.arange(1, 11)
x
```

This creates values from 1 to 10.

You can also specify a step size:

```python
np.arange(0, 21, 5)
```

To generate a fixed number of evenly spaced values, use `linspace`:

```python
b = np.linspace(3, 987, 53)
len(b)
```

---

# Vectorized mathematics

Operations on NumPy arrays are applied element-wise:

```python
c = np.arange(1, 11)
d = 1 / c
d
```

Compute statistics:

```python
np.mean(d)
np.std(d)
```

This avoids slow Python loops and is much faster.

---

# Logical comparisons

We can compare vectors to values:

```python
x = np.arange(1, 11)

x == 5
x < 5
x >= 5
x != 5
```

These comparisons return **boolean arrays** (True / False for each element).

---

# Using logical results to select values

We can use boolean arrays to subset data:

```python
x[x < 5]
x[x >= 5]
x[x != 5]
```

This is a very common pattern in data analysis.

---

# Finding positions with np.where

`np.where` returns the indices where a condition is true:

```python
np.where(x < 5)
np.where(x == 5)
```

You can use these indices to subset:

```python
idx = np.where(x < 5)
x[idx]
```

---

# Working with two vectors

Create another vector:

```python
y = np.arange(7, 16)
y
```

Find common values:

```python
common = np.intersect1d(x, y)
common
```

Find values in `x` that are NOT in `y`:

```python
result = x[~np.isin(x, y)]
result
```

---

# Exercise

1. Create a vector from 10 to 50 in steps of 2.
2. Select only values larger than 25.
3. Create a second vector from 30 to 70.
4. Find the values common to both vectors.
5. Find the values in the first vector that are not in the second.

---

# Why this matters for bioinformatics

In gene expression analysis we often need to:

- select genes based on thresholds
- find overlaps between gene sets
- remove unwanted genes
- filter samples based on conditions

These operations are performed thousands of times in real workflows.  
Using NumPy logical indexing is the fastest and clearest way to do this.

In the next section, we will use these vectors to create plots.
