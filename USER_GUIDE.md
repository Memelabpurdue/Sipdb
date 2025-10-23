# User Guide

Comprehensive guide for using the SIP Meta-Analysis Pipeline v1.0

---

## Table of Contents

1. [Getting Started](#getting-started)
2. [Data Preparation](#data-preparation)
3. [Running the Pipeline](#running-the-pipeline)
4. [Understanding Results](#understanding-results)
5. [Database Generation](#database-generation)
6. [Advanced Usage](#advanced-usage)
7. [Best Practices](#best-practices)

---

## Getting Started

### Prerequisites

Before using the pipeline, ensure you have:

1. **R ≥ 4.0.0** installed
2. **All required packages** installed (see [INSTALL.md](../INSTALL.md))
3. **Phyloseq objects** prepared with your SIP data
4. **Basic R knowledge** (data frames, functions, file paths)

### Quick Start Example

```r
# Load the pipeline
source("sip_pipeline_v1.0.R")
source("database_builder_v1.0.R")

# Load your data
load("my_sip_studies.RData")  # Contains 'studies' list

# Run analysis
results <- run_complete_sip_analysis(
  studies = studies,
  heavy_threshold = 1.700,
  run_dual_threshold = TRUE,
  n_cores = 4
)

# View results
View(results$master_relaxed)
View(results$consensus$incorporators)
```

---

## Data Preparation

### Phyloseq Object Requirements

Your phyloseq objects must contain:

#### 1. OTU Table
- ASV/OTU counts (samples × taxa)
- Integer counts (not relative abundances)

#### 2. Sample Data (metadata)
- **Required columns:**
  - `Density`: Numeric density values (g/ml)
  - `Isotope` or `Treatment`: Labeled vs. unlabeled indicator
  
- **Optional columns:**
  - `Substrate`: For substrate-specific analysis
  - `Replicate`: For biological replicates
  - `Fraction`: Fraction number
  - `Study`: Study identifier

#### 3. Taxonomy Table (optional but recommended)
- Standard ranks: Kingdom, Phylum, Class, Order, Family, Genus, Species

#### 4. Reference Sequences (optional)
- DNA sequences for each ASV/OTU

### Example Data Structure

```r
# Check your phyloseq structure
ps <- studies[[1]]

# OTU table
otu_table(ps)[1:5, 1:5]

# Sample data
sample_data(ps)

# Taxonomy table
tax_table(ps)[1:5, ]
```

### Preparing Your Data

#### From DADA2 Pipeline

```r
library(phyloseq)

# Create phyloseq object
ps <- phyloseq(
  otu_table(seqtab.nochim, taxa_are_rows = FALSE),
  sample_data(metadata),
  tax_table(taxa),
  refseq(sequences)
)

# Add to studies list
studies <- list(study1 = ps)
```

#### From QIIME2

```r
library(qiime2R)

# Import from QIIME2
ps <- qza_to_phyloseq(
  features = "table.qza",
  taxonomy = "taxonomy.qza",
  metadata = "metadata.txt"
)

# Add density data
sample_data(ps)$Density <- density_values
sample_data(ps)$Isotope <- isotope_labels
```

#### Column Name Standardization

```r
# Ensure correct column names
colnames(sample_data(ps))

# Rename if needed
sample_data(ps)$Density <- sample_data(ps)$density_g_ml
sample_data(ps)$Isotope <- sample_data(ps)$label
```

---

## Running the Pipeline

### Basic Analysis

#### Single Threshold

```r
results <- run_complete_sip_analysis(
  studies = studies,
  heavy_threshold = 1.700,  # Density threshold for "heavy" fractions
  run_dual_threshold = FALSE,
  n_cores = 4
)
```

#### Dual Threshold (Recommended)

```r
results <- run_complete_sip_analysis(
  studies = studies,
  heavy_threshold = 1.700,      # Relaxed threshold
  run_dual_threshold = TRUE,
  stringent_threshold = 1.725,  # Stringent threshold
  n_cores = 4
)
```

### Parameter Explanations

#### Density Thresholds

```r
# heavy_threshold: Minimum density to consider as "heavy" (labeled)
# Based on CsCl gradient fractionation
# Typical ranges:
# - Relaxed: 1.680-1.710 g/ml (more sensitive, more false positives)
# - Standard: 1.700-1.720 g/ml (balanced)
# - Stringent: 1.720-1.750 g/ml (more specific, fewer false positives)

heavy_threshold = 1.700      # Standard, recommended starting point
stringent_threshold = 1.725  # For high-confidence results
```

#### Statistical Thresholds

```r
# Adjust significance cutoffs
results <- run_complete_sip_analysis(
  studies = studies,
  heavy_threshold = 1.700,
  min_log2fc = 1.0,    # Minimum 2-fold change
  padj_cutoff = 0.05,  # Adjusted p-value threshold
  n_cores = 4
)
```

#### Method Selection

```r
# Control which statistical methods to use
results <- run_complete_sip_analysis(
  studies = studies,
  heavy_threshold = 1.700,
  include_aldex2 = FALSE,  # Skip ALDEx2 (faster)
  n_cores = 4
)
```

#### Parallel Processing

```r
# Check available cores
parallel::detectCores()

# Use fewer cores if memory is limited
results <- run_complete_sip_analysis(
  studies = studies,
  heavy_threshold = 1.700,
  n_cores = 2  # Reduce if out-of-memory errors occur
)
```

### Substrate-Specific Analysis

For multi-substrate experiments:

```r
# Ensure 'Substrate' column exists
sample_data(ps)$Substrate <- c("Glucose", "Glucose", "Cellulose", "Cellulose", ...)

# Run analysis (automatically detects substrates)
results <- run_complete_sip_analysis(
  studies = studies,
  heavy_threshold = 1.700,
  run_dual_threshold = TRUE,
  n_cores = 4
)
```

---

## Understanding Results

### Result Structure

```r
# Main results object contains:
names(results)
# [1] "master_relaxed"      # All results at relaxed threshold
# [2] "master_stringent"    # All results at stringent threshold
# [3] "consensus"           # Consensus incorporators
# [4] "summaries"           # Summary statistics
# [5] "results_relaxed"     # Detailed per-study results
# [6] "results_stringent"   # Detailed stringent results
# [7] "saved_files"         # Paths to output files
```

### Master Table Columns

```r
# View master table
master <- results$master_relaxed

# Key columns:
# - ASV: ASV/OTU identifier
# - study_id: Which study
# - log2FC: Log2 fold-change (labeled vs unlabeled)
# - pvalue: Combined p-value across methods
# - padj: FDR-adjusted p-value
# - is_enriched: TRUE if significantly enriched
# - n_methods: Number of methods detecting enrichment
# - reproducibility_score: Consistency across windows
# - confidence_tier: high/medium/low confidence
# - Kingdom, Phylum, ..., Species: Taxonomy
```

### Filtering Results

```r
# Only enriched taxa
enriched <- master %>%
  filter(is_enriched == TRUE)

# High-confidence incorporators only
high_conf <- master %>%
  filter(confidence_tier == "high_consensus")

# By taxonomy
phylum_of_interest <- master %>%
  filter(Phylum == "Proteobacteria", is_enriched == TRUE)

# By fold-change
strong_incorporators <- master %>%
  filter(is_enriched == TRUE, log2FC > 2)  # >4-fold change
```

### Consensus Results

```r
# High-confidence incorporators identified across studies
consensus <- results$consensus$incorporators

# Columns:
# - ASV: ASV identifier
# - n_studies: Number of studies detecting this ASV
# - mean_log2FC: Average fold-change across studies
# - confidence: "high_consensus"
# - Taxonomy columns
```

### Summary Statistics

```r
summaries <- results$summaries

# Study-level summaries
summaries$study_summary

# Method comparison
summaries$method_comparison

# Taxonomic distribution
summaries$phylum_enrichment
```

---

## Database Generation

### Creating Integrated Database

```r
# Build comprehensive database with FASTA and BLAST
db_result <- build_optimized_sip_database_v7(
  studies = studies,
  pipeline_dir = "~/Documents/sip_pipeline",
  create_blast = TRUE,
  optimize_structure = TRUE,
  include_consensus = TRUE
)
```

### Database Outputs

```r
# Database structure:
# pipeline_dir/
#   database/
#     sip_integrated_v1.sqlite    # SQLite database
#     database_build_report_v1.txt
#   sequence_database_comprehensive/
#     all_sequences_v1.fasta      # All ASV sequences
#     sip_all_sequences_db_v1.*   # BLAST database files
```

### Querying the Database

```r
library(DBI)
library(RSQLite)

# Connect to database
con <- dbConnect(SQLite(), "path/to/sip_integrated_v1.sqlite")

# List tables
dbListTables(con)

# Query enriched taxa
enriched_taxa <- dbGetQuery(con, "
  SELECT 
    a.asv_id,
    t.phylum,
    t.genus,
    r.log2FC,
    r.padj,
    s.sequence
  FROM sip_results_relaxed r
  JOIN taxonomy t ON r.tax_id = t.tax_id
  LEFT JOIN asv_sequences s ON r.ASV = s.asv_id
  WHERE r.is_enriched = 1
  ORDER BY r.log2FC DESC
")

# Close connection
dbDisconnect(con)
```

### Pre-built Database Views

```r
# Use convenient views
view_enriched <- dbGetQuery(con, "SELECT * FROM view_enriched_with_taxonomy")
view_consensus <- dbGetQuery(con, "SELECT * FROM view_consensus_with_sequences")
```

### BLAST Search

```bash
# Search sequences against database
blastn -db sip_all_sequences_db_v1 \
       -query my_query.fasta \
       -out results.txt \
       -outfmt 6
```

---

## Advanced Usage

### Custom Window Parameters

```r
# Modify global parameters before running
BIOLOGICAL_PARAMS$density$window_width <- 0.015  # Narrower windows
BIOLOGICAL_PARAMS$density$overlap <- 0.005       # Less overlap

# Then run analysis
results <- run_complete_sip_analysis(studies = studies, ...)
```

### Processing Individual Studies

```r
# Process one study at a time for memory efficiency
results_list <- list()

for (i in seq_along(studies)) {
  cat("Processing study", i, "\n")
  
  results_list[[i]] <- run_complete_sip_analysis(
    studies = studies[i],
    heavy_threshold = 1.700,
    n_cores = 4
  )
  
  gc()  # Garbage collection
}
```

### Custom Statistical Methods

```r
# Modify which methods are used
# Edit in the pipeline code or:

# Run only specific methods
results_deseq_only <- run_sip_analysis(
  physeq = studies[[1]],
  methods = c("deseq2"),  # Only DESeq2
  ...
)
```

### Memory Management

```r
# Monitor memory usage
report_memory_usage()

# Clear caches
clear_cache()

# Remove large intermediate objects
rm(large_object)
gc()
```

---

## Best Practices

### Experimental Design

1. **Sufficient replication**: ≥3 biological replicates per group
2. **Proper controls**: Unlabeled substrate controls
3. **Good fractionation**: Clear density gradient separation
4. **Adequate sequencing depth**: ≥10,000 reads per sample

### Data Quality

1. **QC before analysis**:
   ```r
   # Check library sizes
   sample_sums(ps)
   summary(sample_sums(ps))
   
   # Check sparsity
   sum(otu_table(ps) == 0) / length(otu_table(ps))
   ```

2. **Remove low-quality samples**:
   ```r
   # Filter low-depth samples
   ps <- prune_samples(sample_sums(ps) >= 5000, ps)
   ```

3. **Pre-filter rare taxa**:
   ```r
   # Remove very rare taxa
   ps <- filter_taxa(ps, function(x) sum(x > 0) >= 2, TRUE)
   ```

### Threshold Selection

1. **Start with standard thresholds** (1.700/1.725)
2. **Run dual thresholds** to assess robustness
3. **Visualize density distributions**:
   ```r
   ggplot(sample_data(ps), aes(x = Density, fill = Isotope)) +
     geom_density(alpha = 0.5) +
     geom_vline(xintercept = 1.700, linetype = "dashed")
   ```

### Result Interpretation

1. **Use consensus results** for high confidence
2. **Check reproducibility scores**
3. **Validate with literature** or database searches
4. **Consider biological plausibility**

### Reproducibility

1. **Set random seed**:
   ```r
   set.seed(42)
   ```

2. **Document versions**:
   ```r
   sessionInfo()
   ```

3. **Save workspace**:
   ```r
   save.image("analysis_workspace.RData")
   ```

4. **Keep analysis scripts**

---

## Example Workflows

### Complete Analysis Pipeline

```r
# 1. Setup
library(phyloseq)
source("sip_pipeline_v1.0.R")
source("database_builder_v1.0.R")

# 2. Load data
load("sip_studies.RData")

# 3. QC
for (name in names(studies)) {
  cat("\n=== Study:", name, "===\n")
  ps <- studies[[name]]
  cat("Samples:", nsamples(ps), "\n")
  cat("Taxa:", ntaxa(ps), "\n")
  cat("Median depth:", median(sample_sums(ps)), "\n")
}

# 4. Run analysis
results <- run_complete_sip_analysis(
  studies = studies,
  heavy_threshold = 1.700,
  run_dual_threshold = TRUE,
  stringent_threshold = 1.725,
  n_cores = 4
)

# 5. Build database
db_result <- build_optimized_sip_database_v7(
  studies = studies,
  create_blast = TRUE,
  include_consensus = TRUE
)

# 6. Explore results
enriched <- results$master_relaxed %>%
  filter(is_enriched == TRUE) %>%
  arrange(desc(log2FC))

# 7. Save
save(results, file = "sip_results_final.RData")
write.csv(enriched, "enriched_taxa.csv", row.names = FALSE)
```

---

## Troubleshooting

For common issues and solutions, see [TROUBLESHOOTING.md](../TROUBLESHOOTING.md).

Quick tips:

- **"No valid density values"**: Check density column exists and is numeric
- **"Sample identification failed"**: Check Isotope/Treatment column
- **Out of memory**: Reduce n_cores or process studies individually
- **No enriched taxa**: Try more relaxed threshold or check data quality

---

## Getting Help

- **Documentation**: Start here!
- **Tutorial**: See [TUTORIAL.md](TUTORIAL.md) for step-by-step examples
- **Issues**: [GitHub Issues](https://github.com/ymemelabpurdue/Sipdb/issues)
- **Discussions**: [GitHub Discussions](https://github.com/memelabpurdue/Sipdb/discussions)

---

**Happy analyzing!** 🔬
