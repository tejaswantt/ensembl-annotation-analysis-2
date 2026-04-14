# Ensembl Genome Annotation Analysis Framework

A Python-based pipeline for genome annotation quality assessment across multiple species. The framework retrieves annotation metrics from Ensembl core databases, computes derived features, groups species by taxonomy, detects anomalous annotation patterns, and generates structured quality control reports.

---

## Background

Genome annotation projects produce large volumes of structured data describing genes, transcripts, exons, and associated metrics across hundreds of species. As the number of annotated genomes grows, manually inspecting raw metrics for quality issues becomes impractical.

This project addresses that by building an automated analysis layer on top of Ensembl's existing tracking system. All statistical comparisons are performed **within taxonomic groups** (e.g., mammals, birds, fish) rather than globally, ensuring biologically meaningful results.

---

## Features

- SQL-based extraction of annotation statistics from Ensembl core databases
- Computation of derived features: coding ratio, gene density, transcript density, average gene/transcript lengths
- Taxonomy-based grouping of species using `species.classification` metadata
- Multi-layer outlier detection: z-score, quantile, and PCA-based methods
- Composite QC scoring with status labels: `HEALTHY`, `WARNING`, `NEEDS REVIEW`
- Per-genome reports with metrics, rankings, and anomaly flags
- Visualization outputs: histograms, boxplots, PCA scatter plots, comparative charts

---

## Project Structure

```
ensembl-annotation-analysis/
├── data/
│   └── extraction.py          # Ensembl DB connection and SQL queries
├── features/
│   └── engineering.py         # Derived feature computation
├── taxonomy/
│   └── grouping.py            # Taxonomy-based species classification
├── analysis/
│   ├── zscore.py              # Z-score outlier detection
│   ├── quantile.py            # Quantile-based outlier detection
│   └── pca.py                 # PCA multivariate analysis
├── scoring/
│   └── qc.py                  # QC scoring and status classification
├── reporting/
│   └── report.py              # Per-genome report generation
├── visualization/
│   └── plots.py               # Plot generation
├── genome_analysis.ipynb      # Exploratory analysis notebook
└── README.md
```

---

## Installation

```bash
pip install pymysql sqlalchemy pandas numpy scikit-learn matplotlib seaborn
```

---

## Usage

### Connect to Ensembl

```python
import pymysql

conn = pymysql.connect(
    host='ensembldb.ensembl.org',
    user='anonymous',
    password=''
)
```

### Run the Pipeline

```python
from data.extraction import get_species_data
from features.engineering import compute_features
from taxonomy.grouping import assign_taxonomy
from analysis.zscore import detect_zscore_outliers
from analysis.quantile import detect_quantile_outliers
from analysis.pca import detect_pca_outliers
from scoring.qc import classify_qc_status
from reporting.report import generate_report

df = get_species_data(species_list, conn)
df = compute_features(df)
df = assign_taxonomy(df)
df = detect_zscore_outliers(df)
df = detect_quantile_outliers(df)
df = detect_pca_outliers(df)
df = classify_qc_status(df)

generate_report(df, output_dir='reports/')
```

---

## Computed Features

| Feature | Formula |
|---|---|
| `coding_cnt` | `transcript_count - noncoding_cnt` |
| `coding_ratio` | `coding_cnt / transcript_count` |
| `gene_density` | `gene_count / ref_length` |
| `transcript_density` | `transcript_count / ref_length` |
| `avg_gene_length` | `total_gene_length / gene_count` |
| `avg_transcript_length` | `total_transcript_length / transcript_count` |

---

## Outlier Detection

Detection is performed within each taxonomy group to avoid cross-group bias.

**Z-Score** — Flags genomes where a feature deviates more than 3 standard deviations from the group mean.

**Quantile** — Flags genomes in extreme percentile ranges. Robust to non-normal distributions.

**PCA** — Reduces the feature space to principal components and flags genomes that are distant from the group centroid in the transformed space. Captures multivariate anomalies not visible in individual metrics.

Results from all three methods are combined into a single `final_outlier` label per genome.

---

## QC Classification

| Status | Condition |
|---|---|
| `HEALTHY` | No anomalies detected across methods |
| `WARNING` | Flagged by one detection method |
| `NEEDS REVIEW` | Flagged by two or more detection methods |

---

## Taxonomy Groups

| Group | Examples |
|---|---|
| Mammalia | Human, mouse, dog, horse, gorilla |
| Aves | Chicken, zebra finch, falcon, crow |
| Actinopterygii | Zebrafish, pufferfish, salmon, tilapia |
| Reptilia | Lizard, snake, turtle, crocodile |
| Amphibia | Frog, salamander |
| Insecta | *Drosophila*, mosquito |

---

## Notebook

`genome_analysis.ipynb` contains the full exploratory analysis:

- Data extraction across 100+ species from Ensembl core databases
- Feature engineering and taxonomy grouping
- Outlier detection experiments using all three methods
- Distribution and PCA visualizations per taxonomic group

---

## Status

Active development — being built as part of a GSoC 2026 proposal for the Ensembl project under the Open Bioinformatics Foundation.

Repository: [ensembl-annotation-analysis-2](https://github.com/tejaswantt/ensembl-annotation-analysis-2)
