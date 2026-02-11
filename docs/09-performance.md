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

## and now force that into our data type
def from_pd(expr):
    genes = pd.DataFrame(index=expr.index)
    samples = pd.DataFrame(index=expr.columns)
    return {"expression": np.array(expr), "genes": genes, "samples": samples}

hspc_data = from_pd(hspc_data )
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


def distance_matrix_df(data, func):
    X = data['expression']
    names = data['genes'].index
    n = X.shape[0]

    D = np.zeros((n, n))

    for i in range(n):
        for j in range(i,n):
            D[i, j] = func(X[i], X[j])
            D[j, i] = D[i, j] # symmetry

    return pd.DataFrame(D, index=names, columns=names)
```

Time it - or better time a subset of the real data:

```python
## actually first get us a subset of our data:
def subset_genes(data, gene_idx):
    gene_idx = np.asarray(gene_idx)

    X2 = data["expression"][ gene_idx,:]
    genes2 = data["genes"].iloc[gene_idx].copy()

    # samples unchanged
    samples2 = data["samples"].copy()

    return {"expression": X2, "genes": genes2, "samples": samples2}
hspc_data_tiny = subset_genes(  hspc_data , np.arange(200) )
check_data_model( hspc_data_tiny )
hspc_data_tiny.shape
```

```python
(_, t_slow) = timed(distance_matrix_df, hspc_data_tiny, euclid_distance_by_hand )# first 200 genes of the whole data only
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
(_, t_faster) = timed(distance_matrix_df, hspc_data_tiny , euclid_distance_vec )
print("faster:", t_faster)
```

You should usually see a big speedup.

---

But there are also specialized function that operate on the full matrix:

```python
from scipy.spatial.distance import pdist, squareform

def vectorized_dist(data):
    """
    data: a dist with "expression" - a numpy array (rows = features, cols = observations)
    returns: full (nrow x nrow) Euclidean distance matrix as numpy array
    """
    # upper triangle (condensed form)
    upper = pdist(data['expression'], metric="euclidean")

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


# Interpreting the result

Usually:

- vectorized version is very fast
- pure python as well as manual for loops are much slower

Why? Because even a for loop calls Python code repeatedly whereas the vectorized function (c or fortran) works on one big memory block.


---

# Exercise

Take the function ``zscore_rows`` and convert it from using a numpy ndarray to using our own data structure.
While doing that change tha action to modifying the data in place. 

**Note:** Mutable objects (like lists, dictionaries, and arrays) can be changed inside a function, while immutable objects (like numbers and strings) cannot. Think of it like "Small objects like numbers or strings can be copied, but putatively large ones like matrices or dictionaries should not be copied". 


In the next section, we will apply these ideas to real expression data: selecting variable genes and scaling (z-scores).
