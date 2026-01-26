---
title: "Plotting data"
---

# Goal of this section

By the end of this section you will be able to:

- Create simple line and scatter plots
- Label axes and add titles
- Understand how plotting helps interpret biological data

We will use **matplotlib** and **seaborn** only. These are the most widely used plotting tools in Python for scientific work.

---

# Import plotting libraries

```python
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
```

---

# A simple line plot

Create some data:

```python
x = np.arange(1, 11)
y = 1 / x
```

Plot it:

```python
plt.plot(x, y)
plt.show()
```

---

# Adding labels and a title

```python
plt.plot(x, y)
plt.xlabel("x")
plt.ylabel("1 / x")
plt.title("Simple line plot")
plt.show()
```

Good plots always have:
- a title
- labeled axes

---

# Changing the style

You can change how points and lines look:

```python
plt.plot(x, y, marker="o", color="red")
plt.xlabel("x")
plt.ylabel("1 / x")
plt.title("Line plot with markers")
plt.show()
```

---

# Scatter plot

Scatter plots are useful to show relationships between variables.

```python
plt.scatter(x, y)
plt.xlabel("x")
plt.ylabel("1 / x")
plt.title("Scatter plot")
plt.show()
```

---

# Using seaborn (built on matplotlib)

Seaborn works directly with tables (DataFrames).

```python
import pandas as pd

df = pd.DataFrame({"x": x, "y": y})
df
```

Plot with seaborn:

```python
sns.lineplot(data=df, x="x", y="y")
plt.title("Line plot with seaborn")
plt.show()
```

Scatter plot with seaborn:

```python
sns.scatterplot(data=df, x="x", y="y")
plt.title("Scatter plot with seaborn")
plt.show()
```

---

# Plotting real biological data (Iris example)

Load the iris dataset:

```python
from sklearn.datasets import load_iris

iris_data = load_iris(as_frame=True)
iris = iris_data.frame
iris.head()
```

Add species names:

```python
iris["species"] = iris_data.target_names[iris_data.target]
iris.head()
```

Make a scatter plot:

```python
sns.scatterplot(data=iris,
                x="sepal length (cm)",
                y="sepal width (cm)",
                hue="species")

plt.title("Sepal length vs width")
plt.show()
```

Each point is one flower.  
Color shows the species.

---

# Exercise

1. Make a scatter plot of:
   - x = `petal length (cm)`
   - y = `petal width (cm)`
   - color = species

2. Change the plot title and axis labels.

---

# Why this matters for bioinformatics

Plots help us:
- see patterns
- detect outliers
- compare groups

In gene expression analysis, we use plots to:
- compare samples
- detect clusters
- visualize trends

In the next section, we will learn how to work with matrices and tables of data.
