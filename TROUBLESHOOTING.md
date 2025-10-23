# Troubleshooting Guide

Common issues and solutions for the SIP Meta-Analysis Pipeline v1.0

---

## Table of Contents

- [Installation Issues](#installation-issues)
- [Runtime Errors](#runtime-errors)
- [Data Issues](#data-issues)
- [Performance Problems](#performance-problems)
- [Output Issues](#output-issues)
- [Database Problems](#database-problems)
- [Platform-Specific Issues](#platform-specific-issues)
- [Getting Help](#getting-help)

---

## Installation Issues

### Issue: Cannot install BiocManager

**Error Message:**
```
Warning in install.packages : package 'BiocManager' is not available
```

**Solution:**
```r
# Update R to latest version first
# Then try with a specific CRAN mirror
install.packages("BiocManager", repos = "https://cloud.r-project.org/")
```

---

### Issue: Bioconductor packages fail to install

**Error Message:**
```
Installation of package 'DESeq2' had non-zero exit status
```

**Solutions:**

**1. Check R and Bioconductor compatibility:**
```r
# Check current versions
R.version.string
BiocManager::version()

# Update Bioconductor
BiocManager::install(version = "3.17", update = TRUE, ask = FALSE)
```

**2. Install system dependencies (Linux):**
```bash
sudo apt-get update
sudo apt-get install build-essential libxml2-dev libcurl4-openssl-dev libssl-dev
```

**3. Install packages one at a time:**
```r
# Install individually to identify problematic packages
BiocManager::install("phyloseq", force = TRUE)
BiocManager::install("DESeq2", force = TRUE)
# etc.
```

**4. Clear cache and reinstall:**
```r
# Remove problematic package
remove.packages("DESeq2")

# Clear BiocManager cache
unlink(file.path(Sys.getenv("HOME"), ".cache", "R"), recursive = TRUE)

# Reinstall
BiocManager::install("DESeq2")
```

---

### Issue: BLAST not found

**Error Message:**
```
makeblastdb: command not found
```

**Solutions:**

**macOS:**
```bash
# If using Homebrew
brew install blast

# Verify
which makeblastdb
```

**Linux:**
```bash
# Ubuntu/Debian
sudo apt-get install ncbi-blast+

# Add to PATH if needed
export PATH="/usr/local/ncbi/blast/bin:$PATH"
echo 'export PATH="/usr/local/ncbi/blast/bin:$PATH"' >> ~/.bashrc
```

**Windows:**
```cmd
# Add BLAST to PATH in System Environment Variables
# Usually: C:\Program Files\NCBI\blast\bin
```

---

### Issue: Package version conflicts

**Error Message:**
```
package 'xxx' was installed by an R version with different internals
```

**Solution:**
```r
# Update all packages
update.packages(ask = FALSE, checkBuilt = TRUE)

# Reinstall all Bioconductor packages
BiocManager::install(update = TRUE, ask = FALSE)

# If problem persists, remove and reinstall specific package
remove.packages("problematic_package")
BiocManager::install("problematic_package")
```

---

## Runtime Errors

### Issue: "No valid density values in biological range"

**Error Message:**
```
ERROR: No valid density values in biological range
```

**Causes:**
- Density column missing from sample data
- Density values outside 1.650-1.800 g/ml range
- Density values are NA or non-numeric

**Solutions:**

**1. Check density data:**
```r
# View sample data
sample_data(your_phyloseq)

# Check density column
sample_data(your_phyloseq)$Density
summary(sample_data(your_phyloseq)$Density)
```

**2. Verify column name:**
```r
# Density column must be named exactly "Density"
colnames(sample_data(your_phyloseq))

# Rename if needed
sample_data(your_phyloseq)$Density <- sample_data(your_phyloseq)$density_g_ml
```

**3. Check for valid range:**
```r
# Filter valid densities
valid <- sample_data(your_phyloseq)$Density >= 1.650 & 
         sample_data(your_phyloseq)$Density <= 1.800
sum(valid, na.rm = TRUE)  # Should be > 0
```

---

### Issue: "Sample identification failed"

**Error Message:**
```
ERROR: Could not identify labeled and unlabeled samples
```

**Causes:**
- Missing or incorrectly named isotope column
- No clear separation between labeled and unlabeled samples
- Insufficient samples

**Solutions:**

**1. Check required columns:**
```r
# Must have 'Isotope' or 'Treatment' column
colnames(sample_data(your_phyloseq))

# Check values
table(sample_data(your_phyloseq)$Isotope)
```

**2. Ensure proper labeling:**
```r
# Labeled samples should contain: '13C', 'labeled', 'heavy', etc.
# Unlabeled should contain: '12C', 'unlabeled', 'light', 'control', etc.

# View unique values
unique(sample_data(your_phyloseq)$Isotope)
```

**3. Manual sample specification:**
```r
# If automatic detection fails, specify manually
labeled_samples <- c("sample1", "sample2", "sample3")
unlabeled_samples <- c("control1", "control2", "control3")

# Add to sample data
sample_data(your_phyloseq)$is_labeled <- 
  sample_names(your_phyloseq) %in% labeled_samples
```

---

### Issue: "Insufficient labeled samples"

**Error Message:**
```
ERROR: Insufficient labeled samples: X < 2
```

**Cause:**
- Fewer than 2 labeled or unlabeled samples per comparison

**Solution:**
```r
# Check sample counts
sample_info <- identify_labeled_unlabeled_samples(your_phyloseq)
sample_info$n_labeled
sample_info$n_unlabeled

# Each substrate/condition needs ≥2 labeled AND ≥2 unlabeled samples
# If insufficient, cannot run statistical analysis for that group
```

---

### Issue: All taxa filtered out

**Error Message:**
```
WARNING: No taxa remaining after filtering
```

**Causes:**
- Data too sparse
- Filtering criteria too strict

**Solutions:**

**1. Relax filtering parameters:**
```r
# Use more lenient filtering
BIOLOGICAL_PARAMS$filtering$min_prevalence <- 0.0001  # More permissive
BIOLOGICAL_PARAMS$filtering$max_sparsity <- 0.999     # Allow more sparse taxa
```

**2. Check data sparsity:**
```r
# Calculate sparsity
otu_table <- otu_table(your_phyloseq)
sparsity <- sum(otu_table == 0) / length(otu_table)
cat("Data sparsity:", round(sparsity * 100, 1), "%\n")

# If > 95%, data may be too sparse
```

**3. Pre-filter before analysis:**
```r
# Remove very low abundance taxa
library(phyloseq)
your_phyloseq <- filter_taxa(your_phyloseq, function(x) sum(x) > 10, TRUE)
```

---

### Issue: DESeq2 estimation fails

**Error Message:**
```
Error in estimateSizeFactors: every gene contains at least one zero
```

**Solutions:**

**1. Use different size factor estimation:**
```r
# This is handled internally, but you can check:
# Try with type = "poscounts"
dds <- DESeqDataSetFromMatrix(
  countData = counts,
  colData = metadata,
  design = ~ condition
)
dds <- estimateSizeFactors(dds, type = "poscounts")
```

**2. Add pseudocounts:**
```r
# Already handled in pipeline with BIOLOGICAL_PARAMS$filtering$pseudocount
# If still failing, increase pseudocount:
BIOLOGICAL_PARAMS$filtering$pseudocount <- 1.0
```

---

## Data Issues

### Issue: Negative control showing enrichment

**Problem:**
Unlabeled controls show similar or higher enrichment than labeled samples

**Diagnosis:**
```r
# Check abundance patterns
plot_ordination(your_phyloseq, 
                ordinate(your_phyloseq, "PCoA"), 
                color = "Isotope")

# Check density distributions
ggplot(sample_data(your_phyloseq), aes(x = Density, fill = Isotope)) +
  geom_density(alpha = 0.5)
```

**Possible Causes:**
1. **Cross-contamination** during fractionation
2. **Incorrect sample labeling** in metadata
3. **Natural density variation** unrelated to isotope incorporation

**Solutions:**
- Verify experimental procedures
- Check sample metadata for errors
- Review fractionation quality
- Consider running with stringent thresholds only

---

### Issue: No incorporators detected

**Problem:**
Pipeline runs successfully but identifies no enriched taxa

**Diagnosis:**
```r
# Check raw results
table(master_table$is_enriched)

# Check p-value distribution
hist(master_table$pvalue)

# Check log2FC distribution
hist(master_table$log2FC)
```

**Possible Causes:**
1. **Insufficient isotope incorporation** in experiment
2. **Too stringent thresholds**
3. **High biological variability**
4. **Low sequencing depth**

**Solutions:**

**1. Use relaxed threshold:**
```r
results <- run_complete_sip_analysis(
  studies = studies,
  heavy_threshold = 1.680,  # More relaxed
  run_dual_threshold = FALSE
)
```

**2. Check individual methods:**
```r
# Look at results from each method
results$results_relaxed[[1]]$deseq2
results$results_relaxed[[1]]$edger
# See which methods detect signal
```

**3. Review QC plots:**
```r
# Check for batch effects, outliers
# Located in plots/qc_plots/
```

---

## Performance Problems

### Issue: Analysis is very slow

**Problem:**
Pipeline takes hours to run

**Solutions:**

**1. Increase parallel cores:**
```r
# Use more cores (check available cores first)
parallel::detectCores()

results <- run_complete_sip_analysis(
  studies = studies,
  n_cores = 8  # Adjust based on available cores
)
```

**2. Disable ALDEx2:**
```r
# ALDEx2 is slowest method
results <- run_complete_sip_analysis(
  studies = studies,
  include_aldex2 = FALSE  # Skip ALDEx2
)
```

**3. Process fewer windows:**
```r
# Reduce window overlap
BIOLOGICAL_PARAMS$density$overlap <- 0.015  # Less overlap = fewer windows
```

**4. Filter more aggressively:**
```r
# Remove low-abundance taxa before analysis
your_phyloseq <- filter_taxa(your_phyloseq, function(x) sum(x > 3) >= 3, TRUE)
```

---

### Issue: Out of memory errors

**Error Message:**
```
Error: cannot allocate vector of size X Gb
```

**Solutions:**

**1. Reduce parallel cores:**
```r
# Fewer parallel processes = less memory
results <- run_complete_sip_analysis(
  studies = studies,
  n_cores = 2  # Reduce from default
)
```

**2. Process studies individually:**
```r
# Instead of all at once
results_list <- list()
for (i in seq_along(studies)) {
  results_list[[i]] <- run_complete_sip_analysis(
    studies = studies[i],
    n_cores = 4
  )
  gc()  # Garbage collection after each
}
```

**3. Increase system RAM or use HPC:**
```r
# For very large datasets, consider:
# - Using a server with more RAM
# - Cloud computing (AWS, GCP)
# - HPC cluster
```

**4. Clear R environment:**
```r
# Remove large objects not needed
rm(list = setdiff(ls(), c("studies", "results")))
gc()

# Clear caches
clear_cache()
```

---

## Output Issues

### Issue: Excel file won't open

**Error:**
"Excel cannot open the file because the file format is not valid"

**Solutions:**

**1. Check file wasn't corrupted:**
```r
# Verify file exists and has size
file.info(excel_path)
```

**2. Try CSV instead:**
```r
# Excel issues sometimes due to format
# CSVs are more reliable
read.csv(csv_path)
```

**3. Regenerate with different package:**
```r
# If openxlsx fails, try xlsx
install.packages("xlsx")
# Or export to CSV and open in Excel
```

---

### Issue: Plots not generated

**Problem:**
Plot directories are empty

**Solutions:**

**1. Check for plotting errors:**
```r
# Run with verbose output
options(warn = 1)  # Print warnings immediately

# Check if plotting functions completed
```

**2. Generate plots manually:**
```r
# If pipeline plots fail, create manually
library(ggplot2)

# Volcano plot
ggplot(master_table, aes(x = log2FC, y = -log10(padj), color = is_enriched)) +
  geom_point() +
  theme_minimal()
```

**3. Check graphics device:**
```r
# Ensure graphics device works
pdf("test.pdf")
plot(1:10)
dev.off()
```

---

## Database Problems

### Issue: SQLite database won't open

**Error:**
```
Error: database disk image is malformed
```

**Solutions:**

**1. Rebuild database:**
```r
# Remove corrupted database
file.remove(db_path)

# Rebuild
db_result <- build_optimized_sip_database_v7(...)
```

**2. Check disk space:**
```bash
df -h  # Ensure sufficient disk space
```

**3. Verify SQLite installation:**
```r
library(RSQLite)
# Should load without errors
```

---

### Issue: BLAST database creation fails

**Error:**
```
Error: makeblastdb failed
```

**Solutions:**

**1. Check BLAST installation:**
```bash
makeblastdb -version
# Should show version info
```

**2. Skip BLAST if not needed:**
```r
db_result <- build_optimized_sip_database_v7(
  studies = studies,
  create_blast = FALSE  # Skip BLAST creation
)
```

**3. Check FASTA file validity:**
```r
# Ensure FASTA was created correctly
fasta <- readDNAStringSet(fasta_path)
length(fasta)  # Should be > 0
```

---

## Platform-Specific Issues

### macOS Issues

**Issue: Cannot install packages requiring compilation**

```bash
# Install Xcode Command Line Tools
xcode-select --install

# Install gfortran
brew install gcc
```

**Issue: Library not loaded errors**

```bash
# Reinstall problematic libraries
brew reinstall openssl
brew reinstall libxml2
```

---

### Windows Issues

**Issue: Rtools not found**

```r
# Install Rtools from: https://cran.r-project.org/bin/windows/Rtools/
# Make sure to check "Add to PATH" during installation

# Verify installation
Sys.which("make")  # Should return path
```

**Issue: Path length limitations**

```r
# Windows has 260 character path limit
# Use shorter directory names or enable long paths:
# Settings > Update & Security > For developers > Enable long paths
```

---

### Linux Issues

**Issue: Permission denied**

```bash
# Grant write permissions
chmod -R 755 ~/Documents/sip_pipeline

# Or use sudo for system-wide installs (not recommended)
```

**Issue: Missing system libraries**

```bash
# Ubuntu/Debian
sudo apt-get install \
  libcurl4-openssl-dev \
  libxml2-dev \
  libssl-dev \
  libgit2-dev \
  libpng-dev \
  libjpeg-dev

# CentOS/RHEL
sudo yum install \
  libcurl-devel \
  openssl-devel \
  libxml2-devel \
  libgit2-devel
```

---

## Getting Help

### Before Asking for Help

1. **Check this troubleshooting guide**
2. **Search existing GitHub issues**
3. **Verify your installation** with `check_installation.R`
4. **Try with test data** if available
5. **Check package versions** are up to date

### When Reporting Issues

Please include:

```r
# System information
sessionInfo()

# Package versions
packageVersion("phyloseq")
packageVersion("DESeq2")
# etc.

# Error messages (complete traceback)
traceback()

# Minimal reproducible example
# Include small test dataset if possible
```

### How to Get Help

1. **GitHub Issues**: [Submit an issue](https://github.com/memelabpurdue/Sipdb/issues)
   - Use for: Bugs, feature requests, installation problems
   - Response time: Usually within 1-3 days

2. **GitHub Discussions**: [Start a discussion](https://github.com/memelabpurdue/Sipdb/discussions)
   - Use for: Usage questions, best practices, ideas
   - Community-driven support

3. **Email**: batista2@purdue.edu
   - Use for: Collaborative inquiries, private data issues
   - Response time: 1-5 business days

### Creating a Minimal Reproducible Example

```r
# Example of good bug report

# 1. Show what you're trying to do
library(phyloseq)
source("sip_pipeline_v1.0.R")

# 2. Create minimal data (or attach small test file)
data(GlobalPatterns)  # Use example data
ps <- subset_samples(GlobalPatterns, SampleType %in% c("Soil", "Feces"))

# 3. Show the error
results <- run_complete_sip_analysis(ps)
# Error: ...

# 4. Include session info
sessionInfo()
```

---

## Common Warning Messages (Not Errors)

### "ALDEx2 skipped - insufficient samples"
**Meaning**: Not enough samples for ALDEx2's Monte Carlo approach
**Action**: No action needed, other methods still run

### "Some windows had no enriched taxa"
**Meaning**: No significant results in certain density windows
**Action**: Normal for sparse data or edge windows

### "Taxonomy not found for some ASVs"
**Meaning**: Some ASVs lack taxonomic classification
**Action**: Normal if using raw OTU tables

### "Heterogeneity detected in meta-analysis"
**Meaning**: Methods disagree on effect sizes
**Action**: Review consensus results, may indicate biological complexity

---

## Still Having Problems?

If your issue isn't covered here:

1. Check the [User Guide](docs/USER_GUIDE.md) for detailed usage
2. Review the [FAQ](docs/FAQ.md)
3. Open a [GitHub issue](https://github.com/memelabpurdue/Sipdb/issues)

---

**Updated**: 2025
**Version**: 1.0.0

Help us improve this guide by reporting issues or suggesting additions!
