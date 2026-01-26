---
title: "Functions and errors"
---

# Goal of this section

By the end of this section you will be able to:

- Define your own functions
- Pass arguments into functions
- Return results from functions
- Handle incorrect input using errors

Functions let us reuse code and make programs easier to read.

---

# Defining a function

A function is a named block of code.

```python
def square(x):
    return x * x
```

Call the function:

```python
square(4)
```

---

# Functions with multiple arguments

```python
def add_numbers(a, b):
    return a + b

add_numbers(3, 5)
```

---

# Using functions with data

```python
import numpy as np

def vector_mean(v):
    return np.mean(v)

x = np.arange(1, 11)
vector_mean(x)
```

---

# Returning multiple results

You can return more than one value using a dictionary.

```python
def calc_mean_and_var(mat):
    means = []
    variances = []
    
    for i in range(mat.shape[0]):
        row = mat.iloc[i, :]
        means.append(np.mean(row))
        variances.append(np.var(row))

    return {
        "means": means,
        "variances": variances
    }
```

---

# Using the function

```python
result = calc_mean_and_var(gmp_data)

result["means"][:5]
result["variances"][:5]
```

---

# Conditional logic in functions

```python
def animal_maths(value1, value2, animal="pig"):
    if animal == "pig":
        return value1 / value2
    elif animal == "cow":
        return value1 * value2
```

```python
animal_maths(10, 5, "pig")
animal_maths(10, 5, "cow")
```

---

# Handling unexpected input

What happens here?

```python
animal_maths(10, 5, "dog")
```

It returns nothing. This can cause bugs later.

Better approach: raise an error.

```python
def animal_maths(value1, value2, animal="pig"):
    operations = {
        "pig": lambda x, y: x / y,
        "cow": lambda x, y: x * y
    }

    if animal not in operations:
        ## be kind and also state what you would expect here!
        raise ValueError(f"Unknown animal: '{animal}'' I only know about {join(operations.keys())}")

    return operations[animal](value1, value2)
```

```python
animal_maths(1, 4, "elephant")
```

---

# Why raise errors?

Raising errors:

- stops incorrect calculations
- shows exactly where something went wrong
- makes debugging easier

---

# Exercise

1. Write a function `row_mean(mat)` that returns row means of a DataFrame using a loop.
2. Modify it so it raises an error if the input is not a DataFrame.
3. Test it on `gmp_data`.

---

# Why this matters for bioinformatics

Bioinformatics scripts often:

- take user input
- process large tables
- write output files

Functions make it possible to:

- structure code
- test small pieces
- reuse analysis steps

In the next section, we will learn how to make this kind of calculation fast using built-in functions.
