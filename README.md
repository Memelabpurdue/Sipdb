# SIP Meta-Analysis Pipeline v1.0

[![R Version](https://img.shields.io/badge/R-%E2%89%A5%204.0.0-blue)](https://www.r-project.org/)

**A comprehensive R pipeline for Stable Isotope Probing (SIP) meta-analysis with integrated database generation**

---

## 🌟 Overview

The SIP Meta-Analysis Pipeline is a production-ready, comprehensive toolkit for analyzing Stable Isotope Probing experiments. It combines multiple statistical methods through meta-analytic approaches to identify isotope-incorporating microorganisms with high confidence.

### Key Features

- **🔬 Multi-Method Analysis**: Integrates DESeq2, edgeR, limma-voom, and ALDEx2
- **📊 Meta-Analytic Consensus**: Combines results across methods for robust identification
- **🎯 Dual Threshold Support**: Relaxed and stringent thresholds for sensitivity analysis
- **🧬 Sliding Window Analysis**: Optimized for density gradient data
- **💾 Integrated Database**: Automatic SQLite, FASTA, and BLAST database generation
- **⚡ High Performance**: Parallel processing and smart caching
- **📈 Quality Control**: Comprehensive validation and reproducibility scoring
- **📖 Extensively Documented**: Biologically-informed parameters with citations

---

## 🚀 Quick Start

```r
# Load the pipeline
source("sip_pipeline_v1.0.R")
source("database_builder_v1.0.R")

# Run complete analysis
results <- run_complete_sip_analysis(
  studies = studies,                    # List of phyloseq objects
  heavy_threshold = 1.700,              # Relaxed threshold
  run_dual_threshold = TRUE,            # Enable stringent analysis
  stringent_threshold = 1.725,          # Stringent threshold
  n_cores = 4                           # Parallel processing cores
)

# Build integrated database
db_result <- build_optimized_sip_database_v7(
  studies = studies,
  pipeline_dir = "~/path/to/pipeline",
  create_blast = TRUE,
  include_consensus = TRUE
)

# Access results
master_table <- results$master_relaxed
consensus_incorporators <- results$consensus$incorporators
summaries <- results$summaries
```

---

## 📋 Table of Contents

- [Installation](#installation)
- [Requirements](#requirements)
- [Usage](#usage)
- [Pipeline Components](#pipeline-components)
- [Output Files](#output-files)
- [Documentation](#documentation)
- [Citation](#citation)
- [Contributing](#contributing)
- [License](#license)
- [Authors](#authors)
- [Support](#support)

---

## 💻 Installation

### Prerequisites

- **R** ≥ 4.0.0
- **BLAST+** (optional, for BLAST database creation)
- Sufficient RAM (≥8 GB recommended for large datasets)

### Step 1: Install R Dependencies

```r
# Install Bioconductor packages
if (!requireNamespace("BiocManager", quietly = TRUE))
    install.packages("BiocManager")

BiocManager::install(c(
  "phyloseq",
  "Biostrings",
  "DESeq2",
  "edgeR",
  "limma",
  "ALDEx2",
  "ComplexHeatmap"
))

# Install CRAN packages
install.packages(c(
  "dplyr", "tidyr", "tibble", "readr", "stringr",
  "data.table", "DBI", "RSQLite",
  "ggplot2", "ggrepel", "pheatmap",
  "openxlsx", "parallel", "foreach", "doParallel",
  "mgcv", "MASS", "vegan", "RColorBrewer"
))
```

### Step 2: Install BLAST+ (Optional)

**macOS:**
```bash
brew install blast
```

**Ubuntu/Debian:**
```bash
sudo apt-get install ncbi-blast+
```

**Windows:**
Download from [NCBI BLAST+](https://ftp.ncbi.nlm.nih.gov/blast/executables/blast+/LATEST/)

### Step 3: Clone Repository

```bash
git clone https://github.com/memelabpurdue/Sipdb.git
cd Sipdb
```

### Step 4: Source the Scripts

```r
source("sip_pipeline_v1.0.R")
source("database_builder_v1.0.R")
```

See [INSTALL.md](INSTALL.md) for detailed installation instructions and troubleshooting.

---

## 📦 Requirements

### R Packages

| Package | Version | Purpose |
|---------|---------|---------|
| phyloseq | ≥1.36 | Microbiome data handling |
| Biostrings | ≥2.60 | Sequence manipulation |
| DESeq2 | ≥1.32 | Differential abundance |
| edgeR | ≥3.34 | Differential abundance |
| limma | ≥3.48 | Differential abundance |
| ALDEx2 | ≥1.24 | Differential abundance |
| dplyr | ≥1.0 | Data manipulation |
| DBI/RSQLite | ≥1.0 | Database operations |
| ggplot2 | ≥3.3 | Visualization |

See complete list in [INSTALL.md](INSTALL.md)

---

## 🔧 Usage

### Basic Analysis

```r
# Load your phyloseq objects
studies <- list(
  study1 = physeq1,
  study2 = physeq2,
  study3 = physeq3
)

# Run pipeline
results <- run_complete_sip_analysis(
  studies = studies,
  heavy_threshold = 1.700,
  run_dual_threshold = TRUE,
  n_cores = 4
)
```

### Advanced Options

```r
# Custom parameters
results <- run_complete_sip_analysis(
  studies = studies,
  heavy_threshold = 1.700,
  stringent_threshold = 1.725,
  run_dual_threshold = TRUE,
  include_aldex2 = TRUE,
  min_log2fc = 1.0,
  padj_cutoff = 0.05,
  n_cores = 8,
  output_prefix = "my_analysis"
)
```

### Database Generation

```r
# Build comprehensive database
db_result <- build_optimized_sip_database_v7(
  studies = studies,
  pipeline_dir = "~/Documents/sip_pipeline",
  create_blast = TRUE,
  optimize_structure = TRUE,
  only_master_asvs = TRUE,
  include_consensus = TRUE
)
```

See [docs/USER_GUIDE.md](docs/USER_GUIDE.md) for detailed usage examples.

---

## 🧩 Pipeline Components

### 1. Statistical Analysis Module
- **DESeq2**: Negative binomial model for count data
- **edgeR**: Empirical Bayes estimation
- **limma-voom**: Linear modeling with precision weights
- **ALDEx2**: Compositional data analysis (optional)

### 2. Sliding Window Analysis
- Biologically-informed window width (0.020 g/ml)
- 50% overlap for smooth transitions
- Density range validation (1.650-1.800 g/ml)

### 3. Meta-Analytic Combination
- Fisher's method for p-value combination
- Stouffer's method with weights
- Heterogeneity assessment (I² statistic)

### 4. Consensus Calling
- Reproducibility scoring across windows
- Confidence tier classification
- High-confidence incorporator identification

### 5. Database Generation
- Normalized SQLite schema
- FASTA file creation
- BLAST database building
- Optimized indexing

---

## 📁 Output Files

The pipeline generates organized outputs in multiple directories:

```
pipeline_output/
├── raw_data/
│   ├── master_table_relaxed_YYYYMMDD_HHMMSS.csv
│   ├── master_table_stringent_YYYYMMDD_HHMMSS.csv
│   └── complete_results_YYYYMMDD_HHMMSS.RData
├── consensus_results/
│   └── consensus_incorporators_YYYYMMDD_HHMMSS.csv
├── database/
│   ├── sip_integrated_v1.sqlite
│   └── database_build_report_v1.txt
├── sequence_database_comprehensive/
│   ├── all_sequences_v1.fasta
│   └── sip_all_sequences_db_v1.*
├── plots/
│   ├── volcano_plots/
│   ├── heatmaps/
│   └── qc_plots/
└── tables/
    ├── summary_tables/
    └── taxonomic_summaries/
```

### Key Output Files

| File | Description |
|------|-------------|
| `master_table_relaxed_*.csv` | Complete results at relaxed threshold |
| `master_table_stringent_*.csv` | Complete results at stringent threshold |
| `consensus_incorporators_*.csv` | High-confidence incorporators |
| `sip_integrated_v1.sqlite` | Integrated SQLite database |
| `all_sequences_v1.fasta` | All ASV sequences |
| `sip_all_sequences_db_v1.*` | BLAST database files |

---

## 📚 Documentation

- **[User Guide](docs/USER_GUIDE.md)** - Comprehensive usage instructions
- **[Installation Guide](INSTALL.md)** - Detailed installation steps
- **[Troubleshooting](TROUBLESHOOTING.md)** - Common issues and solutions
- **[API Reference](docs/API_REFERENCE.md)** - Function documentation
- **[Tutorial](docs/TUTORIAL.md)** - Step-by-step walkthrough
- **[Biological Background](docs/BIOLOGICAL_BACKGROUND.md)** - SIP methodology
- **[Database Schema](docs/DATABASE_SCHEMA.md)** - Database structure

---

## 📖 Citation

If you use this pipeline in your research, please cite:

```bibtex
@software{sip_pipeline_2025,
  author = {SIP Pipeline Team},
  title = {SIP Meta-Analysis Pipeline: Comprehensive Stable Isotope Probing Analysis},
  year = {2025},
  version = {1.0},
  url = {https://github.com/memelabpurdue/Sipdb}
}
```

### Related Publications

This pipeline implements methods from:

- Neufeld, J.D., et al. (2007). "DNA stable-isotope probing." *Nature Protocols*
- Lueders, T., et al. (2004). "Enhanced sensitivity of DNA-based stable-isotope probing"

See [CITATION.md](CITATION.md) for more citation formats.

---

## 🤝 Contributing

We welcome contributions! Please see [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

### Ways to Contribute
- 🐛 Report bugs
- 💡 Suggest new features
- 📝 Improve documentation
- 🔧 Submit pull requests

---

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

## 👥 Authors

See [AUTHORS.md](AUTHORS.md) for the full list of contributors.

---

## 💬 Support

- **Issues**: [GitHub Issues](https://github.com/memelabpurdue/Sipdb/issues)
- **Discussions**: [GitHub Discussions](https://github.com/memelabpurdue/Sipdb/discussions)
- **Email**: your.email@institution.edu

---

## 🙏 Acknowledgments

- Bioconductor community for excellent packages
- All contributors and users who provided feedback

---

## 📊 Statistics

![GitHub stars](https://img.shields.io/github/stars/yourusername/sip-metanalysis-pipeline?style=social)
![GitHub forks](https://img.shields.io/github/forks/yourusername/sip-metanalysis-pipeline?style=social)
![GitHub issues](https://img.shields.io/github/issues/yourusername/sip-metanalysis-pipeline)

---

**Made with ❤️ for the microbial ecology community**
