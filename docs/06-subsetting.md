---
title: "Subsetting and filtering data"
---

# Goal of this section

By the end of this section you will be able to:

- Select rows and columns by position using `iloc`
- Select rows and columns by name using `loc`
- Slice tables
- Filter data using logical conditions

Subsetting is one of the most common operations in data analysis.

---

# Import libraries

```python
import numpy as np
import pandas as pd
```

---

# Create example data

```python
m = np.arange(1, 51).reshape((10, 5))

df = pd.DataFrame(
    m,
    index=["A","B","C","D","E","F","G","H","I","J"],
    columns=["cat", "dog", "pig", "cow", "chicken"]
)

df
```

---

# Selecting values with iloc (by position)

`iloc` uses **row and column numbers** (starting at 0).

First row:

```python
df.iloc[0, :]
```

Third column:

```python
df.iloc[:, 2]
```

Specific elements:

```python
df.iloc[2, 3]
```

Slice rows 3 to 7 (7 is excluded):

```python
df.iloc[2:7, :]
```

---

# Selecting values with loc (by name)

`loc` uses **row and column labels**.

Row named "SPI1":

```python
df.loc["SPI1", :]
```

Value at row "SPI1", column "LTHSC_2":

```python
df.loc["SPI1", "LTHSC_2"]
```

Multiple rows and columns:

```python
df.loc[["GATA1", "LYZ"], [ "LTHSC_2", "MEP_1", "MEP_2" ]]
```

---

# Filtering with conditions

Select rows where values in column "MEP_1" are greater than 20:

```python
df[df["MEP_1"] > 20]
```

This creates a logical mask:

```python
df["MEP_1"] > 20
```

which is then used to select rows.

---

# Subsetting columns

Select specific columns:

```python
df[["LTHSC_2", "MEP_1"]]
```

---

# Combining conditions

Select rows where:
- pig > 20
- AND cow < 40

```python
df[(df["MEP_1"] > 20) & (df["MEP_2"] < 40)]
```

Use:
- `&` for AND
- `|` for OR

Always use parentheses.

---

# Sorting values

Sort rows by a column:

```python
df_sorted = df.sort_values("MEP_2")
df_sorted
```

---

# Exercise

1. Select the first 5 rows using `iloc`.
2. Select the column called `"GMP_1"` using `loc`.
3. Select rows where `"LTHSC_2"` is greater than 30.
4. Select rows where `"LTHSC_1"` is between 10 and 30.
5. Sort the table by `"MEP_2"`.

---

# Biological interpretation

In gene expression data, you will often:

- select specific genes
- select specific samples
- filter genes based on expression thresholds
- split data by experimental groups

All of these use the same subsetting patterns shown here.

In the next section, we will learn how to store multiple objects together using dictionaries.
