# Changelog

All notable changes to the SIP Meta-Analysis Pipeline will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

---

## [1.0.0] - 2025

### 🎉 Initial Release

First production release of the SIP Meta-Analysis Pipeline with comprehensive database integration.

### Added

#### Core Pipeline Features
- Multi-method differential abundance analysis (DESeq2, edgeR, limma-voom, ALDEx2)
- Sliding window analysis for density gradient data
- Meta-analytic consensus calling with heterogeneity assessment
- Reproducibility scoring across analysis windows
- Confidence tier classification for incorporators
- Dual threshold support (relaxed and stringent)
- Substrate-specific analysis capabilities
- Parallel processing with configurable core usage
- Comprehensive quality control and validation

#### Database System
- Normalized SQLite database structure
- Automatic FASTA file generation from ASV sequences
- BLAST database creation for sequence searches
- Optimized indexing for fast queries
- Multiple pre-built views for common queries
- Support for consensus incorporator data
- Study deduplication and relationship tracking
- Sequence metrics (length, GC content)

#### Analysis Features
- Biologically-informed density thresholds (Neufeld et al. 2007, Lueders et al. 2004)
- Adaptive parameter selection based on data quality
- Smart method selection (ALDEx2 only when beneficial)
- Robust error handling with fallback options
- Quality-based study type detection
- Cached taxonomy for improved performance

#### Output and Reporting
- Excel workbooks with multiple sheets
- CSV files for all major results
- Comprehensive text summaries
- Publication-quality plots (volcano, heatmaps, QC)
- Build reports for database generation
- Detailed logging system with color-coded messages

#### Documentation
- Comprehensive README with quick start guide
- Detailed installation instructions
- User guide with examples
- API reference documentation
- Biological background and methodology
- Database schema documentation
- Troubleshooting guide

### Technical Details

#### Biological Parameters
- Heavy threshold: 1.725 g/ml (CsCl density for 13C-DNA separation)
- Light threshold: 1.710 g/ml (upper bound for unlabeled DNA)
- Density range: 1.650-1.800 g/ml
- Window width: 0.020 g/ml
- Window overlap: 0.010 g/ml (50%)
- Log2FC threshold: 1.0 (2-fold change)
- Adjusted p-value: 0.05

#### Performance
- Parallel processing support
- Taxonomy caching
- Validation caching
- Memory usage reporting
- Optimized database queries

#### Quality Control
- Density value validation
- Sample size validation
- Data quality checks
- Study compatibility assessment
- Sparsity filtering
- Minimum count thresholds

### Dependencies

#### R Version
- R ≥ 4.0.0

#### Bioconductor Packages
- phyloseq ≥ 1.36
- Biostrings ≥ 2.60
- DESeq2 ≥ 1.32
- edgeR ≥ 3.34
- limma ≥ 3.48
- ALDEx2 ≥ 1.24
- ComplexHeatmap ≥ 2.8

#### CRAN Packages
- dplyr ≥ 1.0
- tidyr ≥ 1.0
- DBI ≥ 1.0
- RSQLite ≥ 2.0
- ggplot2 ≥ 3.3
- data.table ≥ 1.14
- openxlsx ≥ 4.2
- parallel (base R)
- foreach ≥ 1.5
- doParallel ≥ 1.0

#### External Tools (Optional)
- BLAST+ (for sequence database creation)

### Known Issues
- None at release

### Notes
- This is the first production release
- Replaces all beta and development versions
- Full backward compatibility not guaranteed with pre-1.0 versions

---

## [Unreleased]

### Planned for Future Releases

#### Version 1.1.0 (Q2 2025)
- [ ] Interactive Shiny application for visualization
- [ ] Additional statistical methods (ANCOM-BC2, MaAsLin2)
- [ ] Automated report generation
- [ ] Extended database queries and views
- [ ] Performance benchmarking suite
- [ ] Additional export formats

#### Version 1.2.0 (Q3 2025)
- [ ] Time-series SIP analysis
- [ ] Network analysis integration
- [ ] Machine learning-based classification
- [ ] Multi-substrate comparison tools
- [ ] Enhanced visualization options

### Under Consideration
- Integration with other microbiome analysis pipelines
- Cloud computing support (AWS, GCP)
- Web-based interface
- API for programmatic access
- R package format (submit to CRAN/Bioconductor)

---

## Version History

### Development Versions (Pre-1.0)

**Note**: Development versions were internal and not publicly released.

#### [0.9.0] - Internal Development
- Core pipeline functionality implemented
- Basic database structure
- Initial testing with real datasets

#### [0.8.0] - Internal Development
- Multi-method analysis framework
- Sliding window implementation
- Meta-analysis functions

#### [0.7.0] - Internal Development
- Initial phyloseq integration
- Basic statistical methods
- Output formatting

---

## How to Update

### From Pre-1.0 Versions

If you were using development versions:

```r
# Remove old versions
rm(list = ls())

# Clear cache
if (exists(".sip_taxonomy_cache")) {
  rm(.sip_taxonomy_cache, envir = .GlobalEnv)
}

# Source new version
source("sip_pipeline_v1.0.R")
source("database_builder_v1.0.R")
```

### Future Updates

```bash
# Pull latest version
git pull origin main

# Check for new dependencies
source("check_installation.R")

# Update R packages if needed
BiocManager::install(update = TRUE, ask = FALSE)
```

---

## Deprecation Notices

### Version 1.0.0
- **Deprecated**: None (initial release)
- **Removed**: All pre-1.0 development functions

---

## Migration Guide

### Upgrading to 1.0.0

If you have scripts using internal development versions, update function calls:

**Old (pre-1.0):**
```r
run_sip_analysis(...)
build_database(...)
```

**New (1.0.0):**
```r
run_complete_sip_analysis(...)
build_optimized_sip_database_v7(...)
```

See [docs/MIGRATION_GUIDE.md](docs/MIGRATION_GUIDE.md) for detailed migration instructions.

---


**Legend**:
- 🎉 Major release
- ✨ New features
- 🐛 Bug fixes
- 📝 Documentation
- ⚡ Performance improvements
- 🔧 Maintenance
- ⚠️ Breaking changes
