---
title: "Flow control (loops)"
---

# Goal of this section

By the end of this section you will be able to:

- Use `for` loops to repeat operations
- Use `while` loops for conditional repetition
- Store results produced inside loops
- Understand why loops can be slow for large datasets

---

# Import libraries

```python
import pandas as pd
import numpy as np
```

---

# For loops

A `for` loop repeats an action for each value in a sequence.

```python
for i in range(5):
    print(i)
```

This prints numbers from 0 to 4.

---

# Looping over data

Print the first 5 rows of a DataFrame:

```python
for i in range(5):
    print(hspc_data.iloc[i, :])
```

---

# Step size in loops

Print every second row from 0 to 9:

```python
for i in range(0, 10, 2):
    print(hspc_data.iloc[i, :])
```

---

# Storing results from loops

Data created inside a loop disappears unless we store it.
We use containers (usually lists).

```python
values = []

for i in range(10):
    values.append(i * 2)

values
```

---

# Example: row means using a loop

```python
gmp_row_means = []

for i in range(gmp_data.shape[0]):
    gmp_row_means.append(gmp_data.iloc[i, :].mean())

len(gmp_row_means)
```

This works, but it is slow for large tables.

---

# While loops

A `while` loop runs until a condition becomes false.

```python
w = 0
while w <= 5:
    print(w)
    w = w + 1
```

---

# When to use loops

Loops are useful for:

- printing values
- building algorithms step by step
- teaching how algorithms work

They are NOT ideal for:

- numerical operations on large matrices
- statistics on big datasets

---

# Exercise

1. Use a `for` loop to print the first 10 rows of `lthsc_data`.
2. Use a loop to compute the mean of each row in `gmp_data`.
3. Store the results in a list called `row_means`.
4. Print the length of `row_means`.

---

# Why this matters for bioinformatics

Many algorithms (clustering, optimisation, simulations) are based on loops.
Understanding flow control lets you:

- follow algorithm logic
- debug code
- write your own simple algorithms

In the next section, we will turn loop-based code into reusable functions.
