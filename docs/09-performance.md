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

url = "https://raw.githubusercontent.com/shambam/R_programming_1/main/Mouse_HSPC_reduced.txt"
hspc_data = pd.read_csv(
    url,
    sep="\t",
    header=0,
    index_col=0
)

hspc_data
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

This function computes the eucledian distance for two arrays.

Eucledian distance is defined like that:

\[
d_{ij} = \sqrt{\sum_t (g^i_t - g^j_t)^2}
\]


```python
import math

def euclid_distance_by_hand(v1, v2):
    if len(v1) != len(v2):
        raise ValueError(f"Length mismatch: {len(v1)} != {len(v2)}")

    s = 0.0
    for k in range(len(v1)):
        diff = float(v1[k]) - float(v2[k])
        s += diff * diff

    return math.sqrt(s)


def distance_matrix_df(df, func):
    X = np.asarray(df)
    names = df.index
    n = X.shape[0]

    D = np.zeros((n, n))

    for i in range(n):
        for j in range(i,n):
            D[i, j] = func(X[i], X[j])
            D[j, i] = D[i, j] # symmetry

    return pd.DataFrame(D, index=names, columns=names)
```

Time it:

```python
(_, t_slow) = timed(distance_matrix_df, hspc_data.iloc[range(200)], euclid_distance_by_hand )# first 200 genes of the whole data only
print("slow:", t_slow)
```

---

# A fast approach: vectorized pandas methods

Pandas can do these operations in compiled code (fast).

```python
def euclid_distance_vec(v1, v2):
    v1 = np.asarray(v1, dtype=float)
    v2 = np.asarray(v2, dtype=float)

    return np.sqrt(np.sum((v1 - v2) ** 2))

```

Time it:

```python
(_, t_faster) = timed(distance_matrix_df, hspc_data.iloc[range(200)], euclid_distance_vec )
print("faster:", t_faster)
```

You should usually see a big speedup.

---

But there are also specialized function that operate on the full matrix:

```python
from scipy.spatial.distance import pdist, squareform

def vectorized_dist_mat(mat):
    """
    mat: pandas DataFrame or numpy array (rows = observations, cols = features)
    returns: full (n x n) Euclidean distance matrix as numpy array
    """
    X = np.asarray(mat, dtype=float)

    # upper triangle (condensed form)
    upper = pdist(X, metric="euclidean")

    # convert to full symmetric matrix
    D = squareform(upper)

    return D
```

I am sure by now you can check the run time without my help.

Would it be feasable to process all 4170 rows with this fastest function?


## The key lesson

For numerical work:

✅ Prefer **built-in** pandas/NumPy methods  
❌ Avoid Python loops over rows/columns

If not 100% sure you know all about the possible libraries you could use I recommend asking an AI tool for the max speed up in your function.

---

# What about pandas apply?

In R, `apply()` can be a speed trick.  
In Python, `DataFrame.apply()` usually still runs a Python function once per row/column.

That means it often behaves like a loop (and can be slow). But eucledian distance is probably the worst example for apply as it works on one row/column only.
So for this we switch to a simple mean calculation.

```python
def mean_vec(vec1):
    return vec1.mean() # very fast here
```

```python
(_, t_vec) = timed( hspc_data.apply, mean_vec , axis=1)
print("apply time:", t_vec)
```

This is rather fast, but using the inbuild pandas mean function is even faster:


```python
(_, t_vec) = timed( hspc_data.mean,  axis=1)
print("pandas time:", t_vec)
```

If we would have our data as a numpy array instead we could get this even faster using numpy's vectorization:


```python
arr = np.array(hspc_data)
(c, t_fast) = timed(arr.mean, axis=1 )
print("numpy:", t_fast)
```

**Take Home** If numpy has a function for your problem use that!

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

Just keep the ``timed`` function and apply it later on whenever you like ;-)

In the next section, we will apply these ideas to real expression data: selecting variable genes and scaling (z-scores).
