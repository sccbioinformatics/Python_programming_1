---
title: "Python basics and NumPy vectors"
---

# Goal of this section

By the end of this section you will be able to:

- Create and use variables
- Understand the difference between Python lists and NumPy arrays
- Add comments to your code
- Use `help()` to look up documentation
- Perform vectorized numerical operations
- Use logical conditions to select values

These are the building blocks for everything that follows.

---

# Variables

A **variable** is a name that refers to a value stored in memory.

```python
x = 10
y = 3.5
name = "geneA"
```

You can print the contents of a variable:

```python
print(x)
print(name)
```

In Jupyter notebooks, simply writing the variable name as the last statement of the cell will also show its value:

```python
x
```

What would happen if you have another statement after the variable name?

Try:

```python
x
a = 14
```

---

# Comments

Comments are notes for humans. Python ignores them when running code.

```python
x = 5        # this is a comment
y = x + 2    # add 2 to x
```

Use comments to explain *why* something is done, not just *what* is done.

---

# Python lists

A **list** is a collection of values:

```python
genes = ["Gata1", "Runx1", "Spi1"]
genes
```

You can access elements using square brackets `[]`:

```python
print(genes[0])
print(genes[1])
```

Python starts counting at **0**, not 1.

---

# NumPy arrays

In bioinformatics we usually work with numerical vectors and matrices.  
For this, we use **NumPy arrays**.

First import NumPy:

```python
import numpy as np
```

This means:
- Import the `numpy` library
- Make it available as `np`

Later we can call functions like:

```python
np.array(x)
```

---

# Creating arrays

Create a numeric vector:

```python
x = np.array([1, 2, 3, 4, 5])
x
```

NumPy arrays are designed for fast numerical computation.

---

# Lists vs NumPy arrays

Compare this list:

```python
lst = [1, 2, 3, 4, 5]
```

with this array:

```python
arr = np.array([1, 2, 3, 4, 5])
```

Try adding 1:

```python
lst + 1
```

This fails, because lists do not support mathematical operations.

But with NumPy:

```python
arr + 1
```

This adds 1 to every element.  
This is called **vectorized computation**.

It is much faster than Python loops because the operations are executed in optimized compiled code (mostly written in C) and work on entire blocks of memory at once.

---

# Creating numeric sequences

NumPy provides functions to generate sequences easily.

```python
np.arange(1, 11)
```

This creates numbers from 1 to 10.

You can control the step size:

```python
np.arange(0, 21, 5)
```

You can also generate evenly spaced values using `linspace`:

```python
np.linspace(0, 100, 11)
```

---

# Getting help

Python has built-in documentation.

```python
help(np.arange)   # or in Jupyter: ?np.arange
```

Look at:
- what arguments the function takes
- what it returns

---

# Vectorized mathematics

Create a vector:

```python
v = np.arange(1, 11)
v
```

Multiply all values:

```python
2 * v
```

Take the reciprocal:

```python
1 / v
```

Compute the mean and standard deviation:

```python
np.mean(v)
np.std(v)
```

---

# Exercise

1. Create a NumPy vector from 0 to 100 in steps of 10.
2. Compute its mean and standard deviation.
3. Try the same with a Python list. What happens?

---

## Mathematical background

For a vector with values:

\[
x_1, x_2, x_3, \dots, x_n
\]

### Mean

The **mean** (average) is:

\[
\mu = \frac{1}{n} \sum_{i=1}^{n} x_i
\]

This means:
- Add up all values
- Divide by the number of values

---

### Standard deviation

The **standard deviation** measures how much the values vary around the mean.

Population standard deviation:

\[
\sigma = \sqrt{
\frac{1}{n}
\sum_{i=1}^{n} (x_i - \mu)^2
}
\]

Steps:
1. Subtract the mean from each value
2. Square the differences
3. Compute the mean of those squared differences
4. Take the square root

---

# Logical comparisons

We can compare vectors to values:

```python
x = np.arange(1, 11)

print(x == 5)
print(x < 5)
print(x >= 5)
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

This is a very common pattern in data analysis. But would that also work with a list? Try it!


---


# Working with two string vectors

Create some gene vectors:

```python
marker_genes = np.array(["Gata1", "Spi1", "Runx1"])
detected_genes = np.array(["Runx1", "Actb", "Gata1"])
```

Find common values:

```python
common = np.intersect1d(marker_genes, detected_genes)
common
```

Find values in `marker_genes` that are NOT in `detected_genes`:

```python
result = marker_genes[~np.isin(marker_genes, detected_genes)]
result
```

This would once again not work with Python lists.

---

# Exercise

1. Create a vector from 10 to 50 in steps of 2.
2. Select only values larger than 25.
3. Create a second vector from 30 to 70.
4. Find the values common to both vectors.
5. Find the values in the first vector that are not in the second.

---

# Why this matters for bioinformatics

Gene expression data is usually stored as:

- rows = genes
- columns = samples

These are large numerical tables.  
NumPy arrays allow us to:

- store them efficiently
- compute statistics quickly
- filter genes and samples
- find overlaps between gene sets
- prepare data for plotting and clustering

These operations are performed thousands of times in real workflows.
Using NumPy vectorized operations and logical indexing is the fastest and clearest way to do this.

In the next section, we will learn how to plot data in python.
