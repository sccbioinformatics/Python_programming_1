# Python Programming I

This repository contains the material for the course **Python Programming I**, aimed at biomedical and life science researchers who want to learn practical Python for data analysis and bioinformatics.

The full rendered tutorial is available here:

👉 **https://sccbioinformatics.github.io/Python_programming_1/**

---

## 📘 About this course

This course introduces:

- Basic Python syntax and data structures
- Numerical computing with **NumPy**
- Data handling with **pandas**
- Plotting with **matplotlib** and **seaborn**
- Flow control and functions
- Performance considerations (vectorisation vs loops)
- Clustering and heatmaps for gene expression data
- Linear models and ANOVA
- A programming project (k-means clustering)
- A final project on simulated annealing

The focus is on:
- learning how to *think like a programmer*
- writing readable and correct code
- understanding what common bioinformatics tools do internally

---

## 🌐 Website version

The tutorial is automatically built and deployed using GitHub Actions:

**https://sccbioinformatics.github.io/Python_programming_1/**

You do not need to build it locally unless you want to modify the content.

---

## 🗂 Repository structure

```
.
├── docs/                 # Tutorial source files (Markdown)
│   ├── index.md
│   ├── 01-setup.md
│   ├── 02-basics.md
│   ├── ...
│   └── 16-Simulated_annealing_final_project_introduction.md
├── mkdocs.yml            # MkDocs configuration
├── requirements.txt      # Python dependencies for building the site
└── .github/workflows/    # GitHub Actions workflow
```

---

## 🛠 Building locally (optional)

If you want to build the website locally:

```bash
pip install -r requirements.txt
mkdocs serve
```

Then open:

```
http://127.0.0.1:8000
```

---

## 🎯 Target audience

This course is designed for:
- biomedical laboratory scientists
- life science students
- researchers entering bioinformatics
- anyone with basic programming exposure (R helpful but not required)

No prior Python experience is assumed.

---

## 📜 License

This material is intended for teaching and academic use.
You may reuse and adapt it with attribution.

---

## ✉️ Contact

Maintained by the SCC Bioinformatics group.

If you find errors or have suggestions, please open an issue or pull request.
