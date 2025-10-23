---
layout: home
title: Home
---

## ✨ Features

<div class="feature-grid">
  <div class="feature-card">
    <h3><i class="fas fa-microscope"></i> Multi-Method Analysis</h3>
    <p>Integrates DESeq2, edgeR, limma-voom, and ALDEx2 for robust statistical testing</p>
  </div>
  
  <div class="feature-card">
    <h3><i class="fas fa-chart-line"></i> Meta-Analytics</h3>
    <p>Combines results across methods using Fisher's and Stouffer's approaches</p>
  </div>
  
  <div class="feature-card">
    <h3><i class="fas fa-database"></i> Database Integration</h3>
    <p>Automatic SQLite, FASTA, and BLAST database generation</p>
  </div>
  
  <div class="feature-card">
    <h3><i class="fas fa-bolt"></i> High Performance</h3>
    <p>Parallel processing with multi-core support and smart caching</p>
  </div>
  
  <div class="feature-card">
    <h3><i class="fas fa-check-double"></i> Quality Control</h3>
    <p>Comprehensive validation, reproducibility scoring, and confidence tiers</p>
  </div>
  
  <div class="feature-card">
    <h3><i class="fas fa-book"></i> Well Documented</h3>
    <p>30,000+ words of documentation, tutorials, and troubleshooting guides</p>
  </div>
</div>

---

## 🚀 Quick Example

Get started with just three lines of code:

```r
# Load pipeline
source("sip_pipeline_v1.0.R")

# Run analysis
results <- run_complete_sip_analysis(
  studies = your_phyloseq_list,
  heavy_threshold = 1.700,
  n_cores = 4
)

# View enriched taxa
View(results$master_relaxed %>% filter(is_enriched == TRUE))
```

<div class="alert alert-success">
  <strong><i class="fas fa-check-circle"></i> That's it!</strong> 
  You now have complete SIP analysis with statistical testing, meta-analysis, and publication-ready outputs.
</div>

---

## 📊 What You Get

<table>
  <thead>
    <tr>
      <th>Output</th>
      <th>Description</th>
      <th>Format</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><strong>Statistical Results</strong></td>
      <td>Complete differential abundance analysis</td>
      <td>CSV, Excel, RData</td>
    </tr>
    <tr>
      <td><strong>Enriched Taxa</strong></td>
      <td>High-confidence incorporators with metrics</td>
      <td>CSV, Excel</td>
    </tr>
    <tr>
      <td><strong>Database</strong></td>
      <td>SQLite with optimized queries</td>
      <td>.sqlite</td>
    </tr>
    <tr>
      <td><strong>Sequences</strong></td>
      <td>FASTA file + BLAST database</td>
      <td>.fasta, BLAST</td>
    </tr>
    <tr>
      <td><strong>Visualizations</strong></td>
      <td>Volcano plots, heatmaps, QC plots</td>
      <td>PNG, PDF</td>
    </tr>
  </tbody>
</table>

---

## 🎯 Why Choose This Pipeline?

<blockquote>
  <p><strong>Traditional SIP analysis challenges:</strong></p>
  <ul>
    <li>❌ Single method bias</li>
    <li>❌ Manual integration of results</li>
    <li>❌ No confidence metrics</li>
    <li>❌ Limited database support</li>
  </ul>
</blockquote>

<blockquote class="alert-success">
  <p><strong>Our solution:</strong></p>
  <ul>
    <li>✅ Multi-method meta-analysis</li>
    <li>✅ Automated consensus calling</li>
    <li>✅ Reproducibility scoring</li>
    <li>✅ Integrated database system</li>
  </ul>
</blockquote>

---

## 📚 Documentation

| Guide | Description | Time |
|-------|-------------|------|
| [Quick Start](QUICKSTART) | Get running immediately | 10 min |
| [Installation](INSTALL) | Complete setup guide | 30 min |
| [Tutorial](docs/TUTORIAL) | Step-by-step walkthrough | 45 min |
| [User Guide](docs/USER_GUIDE) | Comprehensive reference | Reference |
| [FAQ](docs/FAQ) | Common questions | As needed |
| [Troubleshooting](TROUBLESHOOTING) | Problem solving | As needed |

---

## 🔬 Scientific Background

**Stable Isotope Probing (SIP)** identifies microorganisms that actively metabolize specific substrates:

1. 🧪 Provide <sup>13</sup>C or <sup>15</sup>N labeled substrates
2. 🧬 Active organisms incorporate heavy isotopes into DNA
3. ⚖️ Density gradient centrifugation separates heavy DNA
4. 🔬 Sequencing identifies metabolically active taxa

**Key Publications:**
- Neufeld et al. (2007) *Nature Protocols* - [DOI:10.1038/nprot.2007.109](https://doi.org/10.1038/nprot.2007.109)
- Lueders et al. (2004) *Environmental Microbiology* - [DOI:10.1046/j.1462-2920.2003.00536.x](https://doi.org/10.1046/j.1462-2920.2003.00536.x)

---

## 💻 System Requirements

<div class="feature-grid">
  <div class="feature-card">
    <h4>Software</h4>
    <ul>
      <li>R ≥ 4.0.0</li>
      <li>BLAST+ (optional)</li>
    </ul>
  </div>
  
  <div class="feature-card">
    <h4>Hardware</h4>
    <ul>
      <li>8 GB RAM (16 GB recommended)</li>
      <li>Multi-core CPU</li>
      <li>5 GB disk space</li>
    </ul>
  </div>
</div>

---

## 📖 Citation

If you use this pipeline in your research, please cite:

```bibtex
@software{sip_pipeline_2025,
  author = {SIPdb Development Team},
  title = {SIP Meta-Analysis Pipeline},
  year = {2025},
  version = {1.0.0},
  url = {https://github.com/meme}
}
```

---

## 🤝 Contributing

We welcome contributions! See our [contributing guidelines](CONTRIBUTING) for:

- 🐛 Bug reports
- 💡 Feature requests  
- 🔧 Code contributions
- 📖 Documentation improvements

---

## 📞 Support

<div class="feature-grid">
  <div class="feature-card">
    <h4><i class="fab fa-github"></i> GitHub Issues</h4>
    <p>Report bugs or request features</p>
    <a href="{{ site.author.github }}/issues" target="_blank" class="btn btn-primary">Open Issue</a>
  </div>
  
  <div class="feature-card">
    <h4><i class="fas fa-comments"></i> Discussions</h4>
    <p>Ask questions or share ideas</p>
    <a href="{{ site.author.github }}/discussions" target="_blank" class="btn btn-primary">Join Discussion</a>
  </div>
  
  <div class="feature-card">
    <h4><i class="fas fa-envelope"></i> Email</h4>
    <p>Direct contact for collaboration</p>
    <a href="mailto:{{ site.author.email }}" class="btn btn-primary">Send Email</a>
  </div>
</div>

---

## ⭐ Show Your Support

If this pipeline helps your research, please:
- ⭐ Star the repository on GitHub
- 📖 Cite it in your publications  
- 🐦 Share with colleagues
- 🤝 Contribute improvements

---

<div class="text-center mt-4">
  <p style="font-size: 1.2rem; color: #7f8c8d;">
    <strong>Made with ❤️ for the microbial ecology community</strong>
  </p>
</div>