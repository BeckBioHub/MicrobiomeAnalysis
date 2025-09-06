# Microbiome Analysis Workflow 
A compact, reproducible R Markdown workflow for microbiome analysis of different diversity and abundance metrices using phyloseq, vegan, ggplot2, and DESeq2. The data is synthesized for the project and do not mimic real life data.

<img width="6000" height="4500" alt="Combi" src="https://github.com/user-attachments/assets/3cfcd5f5-d504-414a-a5f5-17d73021b407" />



## [Link: Rpubs for Microbiome Analysis](https://rpubs.com/BeckBioHub/1339198)



📌 What this repo demonstrates
```
- Build a phyloseq object from TSVs (OTU table, taxonomy, metadata)

- Rarefaction and plotting of sample counts 

- QC & filtering (depth + prevalence)

- Alpha diversity (Observed, Shannon, Simpson) + stats & plots

- Beta diversity (Bray–Curtis) + PCoA, ellipses, PERMANOVA, dispersion check

- Differential abundance analysis utilizing DESeq2 with LFC shrinkage (apeglm)

- Final patchwork panel combining key plots
```
📦 Required inputs (for reproducing repo output) - adjust parameters and names for own data
```
Place these TSV files at the repo root (or adjust the paths in the script):

otu_table.tsv — rows = taxa/ASVs/OTUs, cols = samples, integer counts

taxonomy.tsv — rows = taxa, cols = ranks (e.g., Phylum Class Order Family Genus Species)

metadata.tsv — rows = samples, must contain a SampleID column and factors:

Diet (e.g., Vegan / Carnivore / Vegetarian)

Activity (low / medium / high)

AgeGroup (four intervals such as 18–29, 30–44, 45–59, 60–80)

Tip: keep real project data private; swap in synthetic/demo data with the same schema for public repos.
```
🔧 Software & packages
```
Tested with R ≥ 4.3.

# CRAN
install.packages(c(
  "tidyverse","ggpubr","patchwork","vegan"
))
# Bioconductor
install.packages("BiocManager")
BiocManager::install(c("phyloseq","DESeq2","apeglm"))


If DESeq2 installation complains about generics or other namespaces: restart R, update generics, broom (and friends), then reinstall DESeq2 with BiocManager. Keep R/Bioconductor versions current.
```
🚀 How to run
```
Clone the repo and open the R project / RStudio.

Put otu_table.tsv, taxonomy.tsv, metadata.tsv at repo root (or change otu_path, tax_path, meta_path).

(Optional) Create an output folder:

dir.create("figures", showWarnings = FALSE)


Knit the R Markdown (or source the chunks interactively).

The Rmd currently contains setwd("C:/Users/x").
For a portable repo, remove that line and rely on project working directory, or use here::here() for paths.
```
🗂️ Script overview (by section)
```
1) Palette (consistent across plots)

Defines a reusable 15-color earthy palette + ggplot helper scales:

scale_color_earth15()
scale_fill_earth15()

2) Build phyloseq

Reads TSVs, aligns IDs (intersection of taxa and samples), and constructs phyloseq(OTU, TAX, SAM).

3) QC & filtering

Minimum sample depth (default: 1000)

Prevalence filter (keep taxa present in ≥ 5% samples)

4) Alpha diversity

estimate_richness(ps, measures = c("Observed","Shannon","Simpson"))

Boxplots by Diet and AgeGroup + nonparametric tests (ggpubr::stat_compare_means)

Outputs:

figures/alpha_diversityDiet.png

figures/alpha_shannon.png

figures/alpha_diversity.png

5) Beta diversity (Bray–Curtis)

Relative abundance transform (transform_sample_counts)

PCoA (ordinate(..., distance="bray")) with ellipses by Diet

PERMANOVA (vegan::adonis2) and dispersion (betadisper)

Outputs:

figures/Beta_divAge.png

figures/Beta_diet.png (ensure you save the intended object; see note below)

Note: In the script, Beta_diet.png is saved with p_bc instead of p_bA.
Change ggsave("figures/Beta_diet.png", p_bA, ...) to match the filename.

6) Differential abundance (DESeq2)

Agglomeration (example uses tax_glom(ps, taxrank="Species"))

phyloseq_to_deseq2(ps_genus, ~ Diet) → DESeq → contrast Vegan vs Carnivore

LFC shrinkage with apeglm

Volcano-style scatter (LFC vs −log10 padj)

Output:

figures/Differential_abundance.png

You can switch the rank (Genus/Family/Species) or the grouping variable (e.g., Activity) by changing tax_glom rank and the design/contrast.

7) Patchwork panel

Combines alpha, beta, and DA plots into a single figure. Legends are collected to the right; per-panel legends are removed where appropriate.

Output:

figures/Combi.png

```

✅ Reproducibility tips
```
Set a seed early: set.seed(42)

Consider initializing renv to pin package versions:

install.packages("renv")
renv::init()

```

🧪 Troubleshooting
```
“cannot open the connection” → path is wrong. Check getwd() and file.exists("otu_table.tsv").

tax_glom not found → load phyloseq (library(phyloseq)) or reinstall via BiocManager. Also ensure the taxonomy has a Species/Genus column to glom to.

DESeq2 install errors (generics cannot be unloaded, etc.) → Restart R, install.packages("generics"), update broom/recipes if you use them, then reinstall DESeq2 with BiocManager.

Patchwork + heatmap: If you later use pheatmap, wrap it before combining:

DEs_wrapped <- patchwork::wrap_elements(full = DEs_ph$gtable)
```


