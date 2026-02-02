---
title: "K-means (concept) and the final project"
---

# Goal of this section

By the end of this section you will:

- Understand what k-means clustering does (conceptually)
- Load a small time-course expression dataset
- Run k-means using scikit-learn (reliable, fast)
- Visualize cluster patterns
- Start the final project: implement a clustering method using simulated annealing

Important note:
This course is about learning to think programmatically. We use library functions when they are the *right tool*,
and we write algorithms “by hand” when it helps you learn.

---

# Import libraries

```python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
```

---

# Load the yeast cell-cycle dataset

We will use a small dataset where expression has been measured across timepoints.
This dataset has clear clusters, which makes it good for learning.

```python
url = "https://raw.githubusercontent.com/shambam/R_programming_1/main/Spellman_Yeast_Cell_Cycle.tsv"
ycc = pd.read_csv(url, sep="\t", index_col=0)
ycc.head()
```

---

# Z-score each gene (row)

We scale each gene to mean 0 and sd 1 so patterns are comparable.

```python
def zscore_rows(mat, ddof=1):
    m = mat.mean(axis=1)
    s = mat.std(axis=1, ddof=ddof)
    return mat.sub(m, axis=0).div(s, axis=0)

ycc_z = zscore_rows(ycc, ddof=1)
ycc_z.head()
```

---

# What does k-means do?

k-means tries to group genes into **K clusters** such that:

- genes in the same cluster have similar expression patterns
- each cluster is represented by a “centroid” (an average pattern)

Algorithm idea:

1. Choose K initial centroids
2. Assign each gene to the closest centroid
3. Update centroids as the mean of assigned genes
4. Repeat until stable

---

# Run k-means using scikit-learn

This gives a correct implementation you can trust.

```python
from sklearn.cluster import KMeans

K = 8
km = KMeans(n_clusters=K, n_init=10, random_state=0)
clusters = km.fit_predict(ycc_z.values)
clusters[:10]
```

Add cluster labels to the table:

```python
ycc_with_cluster = ycc_z.copy()
ycc_with_cluster["cluster"] = clusters
ycc_with_cluster.head()
```

---

# Visualize cluster patterns

We want to see the typical “time-course shape” of each cluster.

Approach:
- for each cluster, plot all genes (faint lines)
- optionally plot the average centroid (thicker line)

```python
timepoints = np.arange(ycc_z.shape[1])

fig, axes = plt.subplots(2, 4, figsize=(16, 8), sharey=True)
axes = axes.flatten()

for k in range(K):
    ax = axes[k]
    members = ycc_z.values[clusters == k, :]

    # plot individual genes
    for i in range(members.shape[0]):
        ax.plot(timepoints, members[i, :], color="black", alpha=0.15)

    # plot centroid
    ax.plot(timepoints, km.cluster_centers_[k, :], color="red", linewidth=2)

    ax.set_title(f"Cluster {k} (n={members.shape[0]})")
    ax.set_xlabel("Time")
    ax.set_ylabel("Z-score")

plt.tight_layout()
plt.show()
```

---

# Exercise (k-means)

Here we try to replicate what scipy does internally using simple Python.

1. Choose K initial centroids (import random)
2. Assign each gene to the closest centroid
3. Update centroids as the mean of assigned genes
4. Repeat until stable

The most important calculation will be the eucledian distance.
Implement your own ``dist( vec1, vec2)`` function.

??? example "You can compare yours to this after you are finished"
    
    ```python
    def dist(vec1, vec2):
        """
        Calculates a single gene <-> gene or gene <-> centroid eucledian distance
        """
        diff = vec1 - vec2
        return np.sum(diff**2) ** 0.5
    ```

We need to repeatetly get the eucledian distances of one centroid against all genes.
Implement a ``dist_to_centroid (centroid, mat )`` function.


??? example "Again only peak after you finished yours"
    
    ```python
    def dist_to_centroid (centroid, mat ):
        """
        Calculates eucledian distance between one centroid and a matrix of genes
        """"
        dists = []
        for i in range(mat.shape[0]):
            dists.append( dist( centroid, mat[i] ) )
        return dists
    ```
And finally we need to get randomness into our scripts.
For this you normall use a random number generator and as we are already using numpy we should probably take the one from there:

```python
rng = np.random.default_rng(seed)
```

With this module we can then e.g. identify a random set of k ids from a range of n ids:

```python
idx = rng.choice(np.arange(1,11), size=k, replace=False)
```

And the loop should contain these steps:
 1. identify k random genes and use them as initial centromers
 2. compare each centromer to each gene and find for each gene the closest centromer
 3. recalculate the new centromers as the mean of the closest genes
 4. if the new centroids look like the old ones - break the loop
 5. use the new centroids and restart the loop
---

??? example "Really - do not peak - use this opportunity to dig into this problem!"
    
    ```python
    def kmeans( data, k, maxiter, seed):
        """
        Clusters rows using the kmeans algorithm
        """
        data_np = np.asarray(data)
        ## define a reproducible 'random' state and get the initial centroids as random genes
        rng = np.random.default_rng(seed)
        idx = rng.choice(np.arange(len(data_np)), size=k, replace=False)
        centroids = data_np[idx]

        ## the main loop
        n = len(data_np)
        for it in range(maxiter):
            # 1) Distance table D: rows=genes, cols=centroids
            D = np.zeros((n, k), dtype=float)
            for c in range(k):
                dists = dist_to_centroid(centroids[c], data_np)
                for i in range(n):
                    D[i, c] = dists[i]

            # 2) Assign each gene to nearest centroid
            labels = np.zeros(n, dtype=int)
            for i in range(n):
                labels[i] = int(np.argmin(D[i, :]))

            # 3) Update centroids (mean of assigned points)
            new_centroids = centroids.copy()
            for c in range(k):
                members = data_np[labels == c]
                if len(members) > 0:                # avoid empty cluster crash
                    new_centroids[c] = np.mean(members, axis=0)
                else:
                    # re-seed empty centroid to a random point (simple, explicit)
                    new_centroids[c] = data_np[rng.integers(0, n)]

            # 4) Stop if centroids no longer move
            if np.allclose(new_centroids, centroids):
                break
            centroids = new_centroids

        return labels, centroids, D
    ```

# The Final Project: Simulated annealing clustering

Your final project will combine everything you learned to implement a clustering algorithm
using a simulated annealing / Metropolis-Hastings style procedure.

This is a great exercise because:
- it forces you to break a problem into smaller functions
- it uses loops and conditionals in a meaningful way
- it links biology (clusters of genes) with a real optimisation method

---

## Optimisation problem

We want clusters to be **compact**: genes in the same cluster should be similar.

We already know Euclidean distance between two genes (vectors) across timepoints:

\[
d_{ij} = \sqrt{\sum_t (g^i_t - g^j_t)^2}
\]

An energy function measures “how bad” a clustering is.
One simple idea is the average within-cluster pairwise distance.

For clusters \(C_1, \ldots, C_K\):

\[
E = \frac{1}{K}\sum_{k=1}^{K} \left( \sum_{i \in C_k} \sum_{j \in C_k, j \neq i} d_{ij} \right)
\]

Lower energy means tighter clusters.

---

## Rescaling step (important)

Before starting simulated annealing, rescale each gene row so values lie between 0 and 1:

```python
def minmax_rows(mat):
    mn = mat.min(axis=1)
    mx = mat.max(axis=1)
    return mat.sub(mn, axis=0).div(mx - mn, axis=0)

ycc_mm = minmax_rows(ycc)
ycc_mm.head()
```

---

## Simulated annealing algorithm (high level)

You will need:

1. Initial temperature `Temp`
2. Cooling factor `cool` (e.g. 0.999 or 0.995)
3. Number of clusters `K`
4. Number of iterations `I`

Pseudo-code:

- Randomly assign each gene to one of K clusters

For iteration 1..I:

1. Compute current energy `E_old`
2. Randomly pick one gene and propose moving it to a different cluster
3. Compute new energy `E_new`
4. Accept if:
   - `E_new < E_old`, or
   - `exp(-(E_new - E_old)/Temp) > U(0,1)`
5. Cool the system: `Temp = Temp * cool`

---

# Things to do (project checklist)

1. Implement an energy function `energy(assignments, dist_matrix, K)`
2. Implement a function that proposes a move (gene -> different cluster)
3. Implement the simulated annealing loop
4. Print:
   - starting energy `E_start`
   - final energy `E_final`
5. Plot the final clusters (like we did for k-means)

---

# Tips

- Break the problem into small functions.
- Start with a small number of genes (e.g. first 50) to debug.
- Only then scale up.

---

# Inspiration (optional reading)

Before you start:
- Read about simulated annealing / Metropolis acceptance
- Remember: the goal is learning, not competing with libraries

You now have the tools to tackle a real algorithmic problem in Python.
Use Google, but no AI tool to learn e.g. how to accept user input in a python script.

# Performance

Once you have your script up and runnig and the results look accetable (plots!) and you still have some juce to program more:

 1. Store the disnatnce matrix and do nor re-calculate
 2. Store the per cluster energies and only re-caulaulte the ones affected by the move
 3. Only calculate a delta energy and not touch the stable genes in the clusters
