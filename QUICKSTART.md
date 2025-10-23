# Quick Start Guide

Get up and running with SIP Meta-Analysis Pipeline in 10 minutes

---

## Prerequisites

✅ R ≥ 4.0.0 installed  
✅ 30 minutes for package installation  
✅ Your SIP data in phyloseq format  

---

## 1. Install (One-Time Setup)

### Install R Packages

```r
# Install Bioconductor manager
if (!requireNamespace("BiocManager", quietly = TRUE))
    install.packages("BiocManager")

# Install Bioconductor packages
BiocManager::install(c(
  "phyloseq", "Biostrings", "DESeq2", "edgeR", 
  "limma", "ALDEx2", "ComplexHeatmap"
))

# Install CRAN packages
install.packages(c(
  "dplyr", "tidyr", "DBI", "RSQLite", "ggplot2",
  "data.table", "openxlsx", "parallel", "foreach", "doParallel"
))
```

### Verify Installation

```r
source("check_installation.R")  # Provided in repository
```

---

## 2. Download Pipeline

```bash
git clone https://github.com/memelabpurdue/Sipdb.git
cd Sipdb
```

---

## 3. Prepare Your Data

### Required Data Structure

Your data must be **phyloseq objects** with:

```r
# Example structure
library(phyloseq)

# Your phyloseq object should have:
# 1. OTU table (counts)
# 2. Sample metadata with these REQUIRED columns:
#    - Density: numeric (g/ml)
#    - Isotope: "12C" or "13C" (or "unlabeled"/"labeled")
# 3. Taxonomy (optional but recommended)

# Check your data
sample_data(your_phyloseq)
# Must see "Density" and "Isotope" columns
```

### Quick Data Check

```r
# Load your phyloseq
ps <- readRDS("your_data.rds")  # or however you load it

# Verify required columns
required_cols <- c("Density", "Isotope")
has_cols <- required_cols %in% colnames(sample_data(ps))

if (all(has_cols)) {
  cat("✅ Data structure looks good!\n")
} else {
  cat("❌ Missing columns:", required_cols[!has_cols], "\n")
}

# Create studies list
studies <- list(study1 = ps)
```

---

## 4. Run Analysis (3 Lines of Code!)

```r
# Load pipeline
source("sip_pipeline_v1.0.R")

# Run analysis
results <- run_complete_sip_analysis(
  studies = studies,
  heavy_threshold = 1.700,
  run_dual_threshold = TRUE,
  n_cores = 4
)

# View results
View(results$master_relaxed)
```

**That's it!** You now have results.

---

## 5. Quick Results Check

### View Enriched Taxa

```r
# Filter to significant results only
enriched <- results$master_relaxed %>%
  filter(is_enriched == TRUE) %>%
  arrange(desc(log2FC))

# How many enriched?
nrow(enriched)

# Top 10
head(enriched, 10)
```

### Quick Visualization

```r
library(ggplot2)

# Volcano plot
ggplot(results$master_relaxed, 
       aes(x = log2FC, y = -log10(padj), color = is_enriched)) +
  geom_point() +
  theme_minimal()
```

---

## 6. Export Results

### Results are automatically saved in:

```
substrate_specific_pipeline_v1.0/
├── raw_data/
│   ├── master_table_relaxed_TIMESTAMP.csv    ← Main results
│   └── master_table_stringent_TIMESTAMP.csv
├── tables/
│   └── summary_tables_TIMESTAMP.xlsx          ← Excel report
└── plots/
    ├── volcano_plots/
    └── heatmaps/
```

### Manual Export

```r
# Export to CSV
write.csv(enriched, "my_enriched_taxa.csv", row.names = FALSE)

# Save workspace
save(results, file = "my_results.RData")
```

---

## Common Adjustments

### Change Density Threshold

```r
# More relaxed (more sensitive)
results <- run_complete_sip_analysis(
  studies = studies,
  heavy_threshold = 1.680,  # Lower threshold
  n_cores = 4
)

# More stringent (more specific)
results <- run_complete_sip_analysis(
  studies = studies,
  heavy_threshold = 1.725,  # Higher threshold
  n_cores = 4
)
```

### Faster Analysis

```r
# Skip ALDEx2 (slowest method)
results <- run_complete_sip_analysis(
  studies = studies,
  heavy_threshold = 1.700,
  include_aldex2 = FALSE,  # Skip ALDEx2
  n_cores = 4
)
```

### Use Fewer Cores

```r
# If running out of memory
results <- run_complete_sip_analysis(
  studies = studies,
  heavy_threshold = 1.700,
  n_cores = 2  # Fewer cores = less memory
)
```

---

## Create Database (Optional)

```r
# Load database builder
source("database_builder_v1.0.R")

# Build database
db_result <- build_optimized_sip_database_v7(
  studies = studies,
  create_blast = TRUE,  # FALSE if no BLAST installed
  include_consensus = TRUE
)

# Database created at:
# pipeline_dir/database/sip_integrated_v1.sqlite
```

---

## What's Next?

### Learn More
- **Detailed usage**: [USER_GUIDE.md](docs/USER_GUIDE.md)
- **Step-by-step tutorial**: [TUTORIAL.md](docs/TUTORIAL.md)
- **Common issues**: [TROUBLESHOOTING.md](TROUBLESHOOTING.md)
- **FAQ**: [FAQ.md](docs/FAQ.md)

### Interpret Results
- **log2FC**: Fold-change (1 = 2x, 2 = 4x, 3 = 8x)
- **padj**: Adjusted p-value (< 0.05 = significant)
- **is_enriched**: TRUE = significantly enriched
- **n_methods**: How many methods agreed
- **confidence_tier**: high/medium/low confidence

### Key Result Columns

```r
# Most important columns in results$master_relaxed:
# ASV           - ASV identifier
# log2FC        - Fold change (labeled vs unlabeled)
# padj          - Adjusted p-value
# is_enriched   - TRUE if significant
# Phylum, Genus - Taxonomy
```

---

## Troubleshooting Quick Fixes

### Error: "No valid density values"
```r
# Check density column exists and is numeric
colnames(sample_data(ps))
class(sample_data(ps)$Density)
range(sample_data(ps)$Density, na.rm = TRUE)
```

### Error: "Sample identification failed"
```r
# Check isotope column
table(sample_data(ps)$Isotope)
# Should see "12C" and "13C" (or "unlabeled"/"labeled")
```

### Error: Out of memory
```r
# Reduce cores
n_cores = 2

# Skip ALDEx2
include_aldex2 = FALSE

# Process studies individually
results1 <- run_complete_sip_analysis(studies = studies[1], ...)
```

### No enriched taxa found
```r
# Try more relaxed threshold
heavy_threshold = 1.680

# Check your data has heavy and light samples
table(sample_data(ps)$Density > 1.700)
```

---

## Getting Help

### Quick Reference
1. This quickstart (you are here!)
2. [FAQ.md](docs/FAQ.md) - Common questions
3. [TROUBLESHOOTING.md](TROUBLESHOOTING.md) - Error solutions

### Need More Help?
- GitHub Issues: https://github.com/memelabpurdue/Sipdb/issues
- Discussions: https://github.com/memelabpurdue/Sipdb/discussions
- Email: batista2@purdue.edu

---

## Minimal Working Example

Here's a complete minimal script:

```r
# Complete script - copy and modify
library(phyloseq)
source("sip_pipeline_v1.0.R")

# Load your data
ps <- readRDS("your_phyloseq.rds")
studies <- list(my_study = ps)

# Run analysis
results <- run_complete_sip_analysis(
  studies = studies,
  heavy_threshold = 1.700,
  n_cores = 4
)

# View results
enriched <- results$master_relaxed %>%
  filter(is_enriched == TRUE)

print(paste("Found", nrow(enriched), "enriched taxa"))

# Export
write.csv(enriched, "enriched_taxa.csv", row.names = FALSE)
```

---

## Parameters Cheat Sheet

```r
run_complete_sip_analysis(
  studies = studies,              # List of phyloseq objects
  heavy_threshold = 1.700,        # Density threshold (1.680-1.750)
  run_dual_threshold = TRUE,      # Also run stringent?
  stringent_threshold = 1.725,    # Stringent threshold
  min_log2fc = 1.0,              # Min fold-change (1 = 2x)
  padj_cutoff = 0.05,            # Significance level
  include_aldex2 = TRUE,         # Include ALDEx2? (slow)
  n_cores = 4,                   # Parallel cores
  output_prefix = "my_analysis"  # Output file prefix
)
```

---

**You're ready to go! 🚀**

Questions? Check the [FAQ](docs/FAQ.md) or [User Guide](docs/USER_GUIDE.md).

---

**Last Updated**: 2025
**Pipeline Version**: 1.0.0
