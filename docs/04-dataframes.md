---
title: "Matrices and DataFrames"
---

# Goal of this section

By the end of this section you will be able to:

- Create matrices using NumPy
- Understand rows vs columns in biological data
- Create labeled tables using pandas DataFrames
- Transpose matrices and tables

These structures are central to bioinformatics:  
**rows = genes, columns = samples**.

---

# Import libraries

```python
import numpy as np
import pandas as pd
```

---

# Creating a matrix with NumPy

Create a matrix of zeros:

```python
m = np.zeros((10, 5))  # 10 rows, 5 columns
m
```

This represents:
- 10 genes (rows)
- 5 samples (columns)

---

# Transposing a matrix

Transpose means swapping rows and columns:

```python
m_T = m.T
m_T
```

This is important because some functions expect:
- rows = observations
- columns = variables

---

# Why we need labels

NumPy matrices only store numbers.  
They do not store gene or sample names.

For biological data, we need labeled rows and columns.  
This is why we use **pandas DataFrames**.

---

# Creating a DataFrame

Convert the NumPy matrix into a DataFrame with labels:

```python
genes = ["Gata1", "Spi1", "Runx1", "Cebpa", "Tal1",
         "Actb", "Kit", "Cd34", "Lyz", "Il7r"]

samples = ["LTHSC_1", "LTHSC_2", "MEP_1", "MEP_2", "GMP_1"]

df = pd.DataFrame(
    m,
    index=genes,
    columns=samples
)
print(df)
```

Now we have:
- row names (index)
- column names (labels)

---

# Shape of a DataFrame

Check dimensions:

```python
df.shape
```

This returns:

```
(number_of_rows, number_of_columns)
```

Access them separately:

```python
print( f"0 - nrow: {df.shape[0]}")  # rows
print( f"1 - ncol: {df.shape[1]}")  # columns
```

---

# Accessing values

Convert DataFrame back to NumPy if needed:

```python
df.values
```

But in most cases, we work directly with the DataFrame.

---

# Creating a numeric DataFrame

Create a matrix of numbers:

```python
m2 = np.arange(1, 51).reshape((10, 5))

df2 = pd.DataFrame(
    m2,
    index=genes,
    columns=samples
)
print(df2)
```
---

# Exercise

1. Plot the gene "Gata1" for the samples
2. Color the plot by sample type
3. 

---

# Why this matters for bioinformatics

Most tools in bioinformatics expect data in this form:

- genes × samples
- with labels

Understanding how to create and manipulate these tables is essential for:
- filtering
- plotting
- clustering
- statistical testing

In the next section, we will learn how to subset and filter these tables.
