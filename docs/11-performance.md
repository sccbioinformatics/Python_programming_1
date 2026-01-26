---
title: "Performance: vectorization vs loops (and why apply is usually slow)"
---

# Goal of this section

By the end of this section you will be able to:

- Explain why Python loops are slow for large numerical data
- Use fast built-in pandas/NumPy methods instead of loops
- Measure runtime using a simple timing helper
- Understand when `apply` is acceptable and when it should be avoided

In bioinformatics, datasets can be large.  
Writing code that *runs* is not enough — it must also run *fast enough*.

---

# Import libraries

```python
import numpy as np
import pandas as pd
```

---

# A timing helper

We'll use this helper function to time any function call.

```python
def timed(func, *args, **kwargs):
    import time
    start = time.perf_counter()
    result = func(*args, **kwargs)
    end = time.perf_counter()
    return result, end - start
```

- `*args` collects extra positional arguments into a tuple
- `**kwargs` collects extra named arguments into a dictionary

---

# A slow approach: looping over rows

This function computes row means and variances using a loop.

```python
def calc_mean_and_var_slow(mat):
    means = []
    variances = []

    for i in range(mat.shape[0]):
        row = mat.iloc[i, :]
        means.append(np.mean(row))
        variances.append(np.var(row, ddof=1))

    return {"means": means, "variances": variances}
```

Time it:

```python
(_, t_slow) = timed(calc_mean_and_var_slow, gmp_data)
print("slow:", t_slow)
```

---

# A fast approach: vectorized pandas methods

Pandas can do these operations in compiled code (fast).

```python
def calc_mean_and_var_fast(mat):
    return {
        "means": mat.mean(axis=1),
        "variances": mat.var(axis=1, ddof=1)
    }
```

Time it:

```python
(_, t_fast) = timed(calc_mean_and_var_fast, gmp_data)
print("fast:", t_fast)
```

You should usually see a big speedup.

---

# The key lesson

For numerical work:

✅ Prefer **built-in** pandas/NumPy methods  
❌ Avoid Python loops over rows/columns

---

# What about pandas apply?

In R, `apply()` can be a speed trick.  
In Python, `DataFrame.apply()` usually still runs a Python function once per row/column.

That means it often behaves like a loop (and can be slow).

Example task:

﻿For each row, compute:

```
(mean * std) / sum
```

---

# Fast (vectorized) solution

```python
means = hspc_data.mean(axis=1)
stds  = hspc_data.std(axis=1, ddof=1)
sums  = hspc_data.sum(axis=1)

result_vectorized = (means * stds) / sums
```

Time it:

```python
(_, t_vec) = timed(lambda: (hspc_data.mean(axis=1) * hspc_data.std(axis=1, ddof=1)) / hspc_data.sum(axis=1))
print("vectorized:", t_vec)
```

---

# Slower solution using apply

```python
def example_func(v):
    return (np.mean(v) * np.std(v, ddof=1)) / np.sum(v)

(_, t_apply) = timed(hspc_data.apply, example_func, axis=1)
print("apply:", t_apply)
```

---

# Interpreting the result

Usually:

- vectorized version is very fast
- apply is much slower

Why? Because `apply` calls Python code repeatedly.

---

# When is apply OK?

`apply` can be reasonable when:

- there is no clean vectorized solution
- the dataset is small
- readability matters more than speed

But for large biological matrices (genes x samples), prefer vectorization.

---

# Exercise

1. Time these two operations on `gmp_data`:

   - `calc_mean_and_var_slow(gmp_data)`
   - `calc_mean_and_var_fast(gmp_data)`

2. Write down the speedup factor:

   ```python
   t_slow / t_fast
   ```

3. Explain in one sentence why the fast method is faster.

---

# Why this matters for bioinformatics

Bioinformatics data matrices can contain:

- 20,000 genes
- 10,000+ cells or samples

A slow method can take minutes or hours.  
A vectorized method can take seconds.

Understanding this difference is one of the most valuable skills in scientific programming.

In the next section, we will apply these ideas to real expression data: selecting variable genes and scaling (z-scores).
