---
title: "Python basics"
---

# Goal of this section

By the end of this section you will be able to:

- Create and use variables
- Understand the difference between Python lists and NumPy arrays
- Add comments to your code
- Use `help()` to look up documentation

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

In Jupyter notebooks, simply writing the variable name will also show its value:

```python
x
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
genes[0]
genes[1]
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
This is called **vectorized computation** and is extremely important for performance.

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
help(np.arange)
```

In Jupyter you can also use:

```python
np.arange?
```

Look at:
- what arguments the function takes
- what it returns

---

# Simple calculations

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

# Why this matters for bioinformatics

Gene expression data is usually stored as:

- rows = genes
- columns = samples

These are large numerical tables.  
NumPy arrays allow us to:
- store them efficiently
- compute statistics quickly
- prepare them for plotting and clustering

In the next section, we will learn how to make plots from these vectors.
