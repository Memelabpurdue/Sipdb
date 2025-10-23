# Frequently Asked Questions (FAQ)

Common questions about the SIP Meta-Analysis Pipeline

---

## General Questions

### What is SIP?

**Stable Isotope Probing (SIP)** is a technique used to identify microorganisms that actively incorporate specific substrates. Heavy isotopes (typically ¹³C or ¹⁵N) are provided in growth substrates, and organisms that consume the substrate incorporate the heavy isotope into their DNA, making it denser. This heavy DNA can be separated via density gradient centrifugation and sequenced.

### What does this pipeline do?

The SIP Meta-Analysis Pipeline:
- Identifies taxa with significantly enriched heavy DNA
- Uses multiple statistical methods for robust detection
- Combines results using meta-analytic approaches
- Provides confidence scores and reproducibility metrics
- Creates integrated databases with sequences
- Generates publication-quality outputs

### Who should use this pipeline?

This pipeline is for researchers working with:
- SIP-seq data (16S)
- Density gradient fractionation data
- Isotope-labeled substrate experiments
- Multi-study meta-analyses

---

## Installation & Setup

### What R version do I need?

R ≥ 4.0.0 is required. R ≥ 4.3.0 is recommended for best compatibility.

### Do I need RStudio?

No, but RStudio is recommended for easier interaction and visualization.

### How long does installation take?

- R packages: 15-45 minutes
- BLAST+: 5-10 minutes
- Total: ~1 hour for first-time setup

### Can I use Docker?

Yes! A Dockerfile is provided. This ensures all dependencies are correctly installed and avoids system-specific issues.

### Do I need BLAST+?

BLAST+ is optional. It's only needed if you want to create searchable sequence databases. All other features work without it.

---

## Data Requirements

### What format should my data be in?

Data should be in **phyloseq objects** containing:
- OTU/ASV count table
- Sample metadata with `Density` and `Isotope` columns
- Taxonomy table (optional but recommended)
- Reference sequences (optional)

### Can I use QIIME2 outputs?

Yes! Convert QIIME2 outputs to phyloseq using the `qiime2R` package:

```r
library(qiime2R)
ps <- qza_to_phyloseq(
  features = "table.qza",
  taxonomy = "taxonomy.qza",
  metadata = "metadata.txt"
)
```

### Can I use mothur outputs?

Yes! Import mothur data with the `phyloseq` package:

```r
import_mothur(mothur_list_file, mothur_group_file, mothur_tree_file)
```

### How many samples do I need?

**Minimum**: 2 labeled + 2 unlabeled samples per comparison  
**Recommended**: 3+ replicates per group for robust statistics

### What density range should I use?

**Standard CsCl gradients**: 1.65-1.80 g/ml  
The pipeline validates this range automatically.

---

## Running the Pipeline

### How long does analysis take?

Depends on:
- Number of studies: 1-10 minutes per study
- Number of taxa: More taxa = longer runtime
- Number of cores: More cores = faster
- ALDEx2 inclusion: Adds significant time

**Typical**: 10-30 minutes for a single study with 1000 taxa

### How many cores should I use?

```r
# Check available cores
parallel::detectCores()

# Use 75% of available cores
n_cores <- ceiling(parallel::detectCores() * 0.75)
```

**Note**: More cores = more memory usage

### What if I run out of memory?

Solutions:
1. Reduce `n_cores`
2. Process studies individually
3. Set `include_aldex2 = FALSE`
4. Pre-filter low-abundance taxa
5. Use a machine with more RAM

### Can I pause and resume?

Not directly, but you can:
1. Save intermediate results with `save(results, file = "checkpoint.RData")`
2. Process studies individually
3. Use the saved RData files to continue

---

## Results & Interpretation

### What is the "heavy threshold"?

The density value above which samples are considered "heavy" (containing ¹³C-labeled DNA). Common values:
- **1.680-1.710**: Relaxed (more sensitive)
- **1.700**: Standard
- **1.720-1.750**: Stringent (more specific)

### What's the difference between relaxed and stringent thresholds?

- **Relaxed**: Lower density threshold, more taxa detected, higher false positive rate
- **Stringent**: Higher density threshold, fewer taxa, lower false positive rate

Use both to assess robustness!

### What is a "consensus incorporator"?

Taxa identified as enriched:
- Across multiple studies
- With high reproducibility
- By multiple statistical methods
These are high-confidence results.

### How do I interpret log2FC?

**Log2 Fold Change** indicates enrichment magnitude:
- `log2FC = 1`: 2-fold increase
- `log2FC = 2`: 4-fold increase
- `log2FC = 3`: 8-fold increase

Higher values = stronger incorporation.

### What is reproducibility score?

Measures consistency across density windows:
- **0-0.3**: Low reproducibility
- **0.3-0.7**: Medium reproducibility
- **0.7-1.0**: High reproducibility

### No enriched taxa detected - is this normal?

Possible reasons:
1. **Legitimate biological result**: No significant incorporation
2. **Too stringent threshold**: Try relaxing to 1.680
3. **Low sequencing depth**: Check library sizes
4. **High variability**: Check replicates
5. **Experimental issues**: Check controls

### How many enriched taxa is normal?

Highly variable:
- **Complex substrates** (e.g., cellulose): 10-50+ taxa
- **Simple substrates** (e.g., glucose): 5-20 taxa
- **Specialist substrates**: 1-10 taxa

---

## Troubleshooting

### "No valid density values" error

**Solutions**:
1. Check density column exists: `colnames(sample_data(ps))`
2. Check density range: `range(sample_data(ps)$Density)`
3. Ensure column is named exactly "Density"

### "Sample identification failed" error

**Solutions**:
1. Check Isotope column: `table(sample_data(ps)$Isotope)`
2. Ensure labeled samples contain: "13C", "labeled", or "heavy"
3. Ensure unlabeled contain: "12C", "unlabeled", or "control"

### Pipeline runs but no results files

Check:
1. Output directory permissions
2. Disk space
3. Check for error messages in console

### Excel file won't open

Try:
1. Open CSV file instead
2. Reinstall `openxlsx` package
3. Check file wasn't corrupted during creation

---

## Performance & Optimization

### How can I speed up analysis?

1. **Use more cores**: Increase `n_cores`
2. **Skip ALDEx2**: Set `include_aldex2 = FALSE`
3. **Pre-filter data**: Remove rare taxa before analysis
4. **Reduce windows**: Decrease overlap in parameters

### How much RAM do I need?

- **Small datasets** (<100 samples, <1000 taxa): 8 GB
- **Medium datasets**: 16 GB
- **Large datasets**: 32+ GB

### Can I run this on a cluster/HPC?

Yes! The pipeline supports:
- Parallel processing
- Batch job submission
- Resource management

Submit as R script to your scheduler.

---

## Database Questions

### What's in the SQLite database?

- **Tables**: Studies, taxonomy, sequences, results
- **Views**: Pre-joined queries for easy access
- **Indexes**: Optimized for fast searches

### Can I use the database without R?

Yes! It's standard SQLite:
- Query with any SQLite client
- Use in Python, Perl, etc.
- Use command-line sqlite3 tool

### What's the BLAST database for?

Search your identified ASVs against:
- New experimental sequences
- Database sequences
- Related study ASVs

### How do I add new data to existing database?

Currently, you must rebuild. Feature planned for v1.1.

---

## Statistical Questions

### Why use multiple methods?

Different methods have different:
- Assumptions
- Strengths
- Weaknesses

Combining methods increases confidence in results.

### What if methods disagree?

This indicates:
- Biological complexity
- Borderline significance
- Need for more replicates

Check `n_methods` and `reproducibility_score` columns.

### What p-value correction is used?

**False Discovery Rate (FDR)** via Benjamini-Hochberg correction, appropriate for multiple testing.

### Can I use my own statistical method?

Yes! The pipeline is modular. You can:
1. Add your method function
2. Include it in the methods list
3. Results will be meta-analytically combined

---

## Customization

### Can I change thresholds?

Yes! Modify `BIOLOGICAL_PARAMS`:

```r
BIOLOGICAL_PARAMS$density$heavy_threshold <- 1.680
BIOLOGICAL_PARAMS$statistics$log2fc_threshold <- 1.5
```

### Can I add custom metadata?

Yes! Add any columns to sample_data. They'll be preserved in outputs.

### Can I modify plots?

Yes! Plot functions return ggplot objects - customize as needed.

---

## Citation & Publication

### How do I cite this pipeline?

See [CITATION.md](../CITATION.md) for multiple formats.

### Can I use this in publications?

Yes! It's MIT licensed - free for any use including commercial.

### Should I cite the statistical methods too?

Yes! Cite DESeq2, edgeR, limma, and ALDEx2 papers. See [CITATION.md](../CITATION.md).

---

## Support & Community

### Where can I get help?

1. **This FAQ** - Start here
2. **[User Guide](USER_GUIDE.md)** - Detailed usage
3. **[Troubleshooting](../TROUBLESHOOTING.md)** - Common issues
4. **GitHub Issues** - Bug reports
5. **GitHub Discussions** - Questions
6. **Email** - Collaborative inquiries

### How can I contribute?

See [CONTRIBUTING.md](../CONTRIBUTING.md)! We welcome:
- Code contributions
- Documentation improvements
- Bug reports
- Feature suggestions

### Is there a mailing list?

Currently: GitHub Discussions  
Planned: Dedicated mailing list in v1.1

---

## Future Development

### What's planned for future versions?

See [CHANGELOG.md](../CHANGELOG.md) "Unreleased" section:
- Shiny app for visualization
- Additional statistical methods
- Time-series SIP analysis
- Network analysis integration

### Can I request features?

Yes! Open a GitHub issue with:
- Feature description
- Use case
- Why it's important

### When is the next release?

- **v1.1**: Q2 2025 (planned)
- **v1.2**: Q3 2025 (tentative)

---

## Still Have Questions?

- **Search GitHub Issues**: Your question may be answered
- **Ask in Discussions**: Community support
- **Email**: batista2@purdue.edu

---

**Last Updated**: 2025
**Version**: 1.0

Help us improve this FAQ by suggesting new questions!
