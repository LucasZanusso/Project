# Multi-Omics Profiling of Chromosome 17 in TCGA-BRCA

Self-directed bioinformatics project exploring copy-number variation,
RNA-seq expression, and DNA methylation in TCGA breast cancer data,
with a biological focus on ERBB2, TP53, and BRCA1 on chromosome 17.

The project was developed as part of my transition from wet-lab
molecular biology to computational genomics.

---

## Project structure

```
GDC open-access derived data
        │
        ├── CNV
        │     └── gene-level and chr17 analyses
        │
        ├── RNA-seq
        │     └── DESeq2 differential expression
        │
        └── DNA methylation
              └── differential methylation analysis

Completed as three independent analyses.

Planned extension:
CNV × expression × methylation integration
```

> \* Scripts `02_alignment.sh` and `03_cnvkit.sh` are included for completeness and
> future use with controlled-access FASTQs (e.g. dbGaP / PhD cluster).
> The current open-access workflow starts at `00_download.sh` and proceeds
> directly to the analysis scripts (`04`, `05`, `06`).
## Project scope

Three omics layers were analyzed independently:

- Copy-number variation (CNV)
- RNA-seq gene expression
- DNA methylation

The original goal was to integrate the three modalities across a larger
matched cohort. However, scaling the analysis to a more informative
sample size exceeded the computational resources available on my local
machine. Therefore, the repository presents the completed layer-specific
analyses, while cross-omic integration remains a future extension.

## Current status

- ✅ CNV analysis completed
- ✅ RNA-seq analysis completed
- ✅ DNA methylation analysis completed
- ⏸ Cross-omic integration not completed due to local computational constraints
---

## Pipeline overview

### Current workflow — open-access derived data

```
GDC Portal (open access)
  │
  ├── Masked Copy Number Segment (.txt)     ← pre-computed by GDC/TCGA
  ├── Gene Expression Quantification (.tsv) ← STAR counts, GRCh38
  └── Methylation Beta Value (.txt)         ← Illumina 450k array
        │
        ▼ 00_download.sh
  gdc-client → organized into data/cnv | rna | methylation
        │
        ├──▶ 04_cnv_analysis.R
        │    chr17 segments · ERBB2/TP53/BRCA1 heatmap · Gviz locus plots
        │
        ├──▶ 05_rnaseq.R              
        │    DESeq2 normalization · DEG tumor vs normal · volcano plots
        │
        └──▶ 06_methylation.R         
           minfi QC · beta/M-value · DMR detection · methylation heatmap
        
       
```

### Future workflow — controlled-access FASTQs (dbGaP)

```
FASTQ (SRA / GDC controlled access)
  │
  ▼ 01_qc.sh          fastp → FastQC → MultiQC
  ▼ 02_alignment.sh   BWA-MEM → samtools markdup → .dedup.bam
  ▼ 03_cnvkit.sh      CNVkit batch → .cnr / .cns / .call.cns
  ▼ 04_cnv_analysis.R (same as above)
```

---

## Sample cohort

9 cases selected from the TCGA-BRCA 2012 paper (Supplementary Table 1).
Selection criteria: all three data types available in open access + biological
diversity across expected chr17 alteration patterns.

| Case ID | PAM50 subtype | HER2 status | Expected chr17 event |
|---------|--------------|-------------|----------------------|
| TCGA-B6-A0I9 | HER2-enriched | Positive | ERBB2 amplification |
| TCGA-BH-A18R | HER2-enriched | Positive | ERBB2 amplification |
| TCGA-BH-A0DZ | HER2-enriched | Positive | ERBB2 amplification |
| TCGA-A1-A0SK | Basal-like | Negative | TP53 deletion / BRCA1 loss |
| TCGA-A2-A0CM | Basal-like | Negative | TP53 deletion / BRCA1 loss |
| TCGA-BH-A18V | Basal-like | Negative | TP53 deletion / BRCA1 loss |
| TCGA-BH-A18Q | Basal-like | Negative | TP53 deletion / BRCA1 loss |
| TCGA-A2-A0CU | Luminal A | Negative | Biological control |
| TCGA-AR-A0TR | Luminal A | Negative | Biological control |

## Cohort limitation

The initial proof-of-concept cohort contained nine TCGA-BRCA cases,
with matched solid-tissue normal samples available for only a subset
of RNA-seq and methylation analyses.

This cohort was sufficient for workflow development and exploratory
analysis, but I considered it too limited for a robust integrative
multi-omics interpretation. Expanding the cohort substantially
increased local memory and processing requirements, so the integration
stage was not pursued further on the available hardware.

### Data availability per case

| Case ID | CNV tumor | CNV normal | RNA tumor | RNA normal | Meth tumor | Meth normal |
|---------|-----------|------------|-----------|------------|------------|-------------|
| TCGA-A1-A0SK | ✅ | ✅ | ✅ | — | ✅ | — |
| TCGA-A2-A0CM | ✅ | ✅ | ✅ | — | ✅ | — |
| TCGA-A2-A0CU | ✅ | ✅ | ✅ | — | ✅ | — |
| TCGA-AR-A0TR | ✅ | ✅ | ✅ | — | ✅ | — |
| TCGA-B6-A0I9 | ✅ | ✅ | ✅ | — | ✅ | — |
| TCGA-BH-A0DZ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| TCGA-BH-A18Q | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| TCGA-BH-A18R | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| TCGA-BH-A18V | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |

> Normal for CNV = Peripheral Blood (sample suffix `-10A`).  
> Normal for RNA-seq and methylation = Solid Tissue Normal (`-11A`),
> available only for 4 cases. For the remaining 5, tumor-only analysis applies.

### Manifest filtering

The original GDC cart contained 48 files. Four were removed before download:

- **3 files** from `TCGA-BH-A18V-06A` (metastatic sample) — kept only Primary Tumor `-01A`
- **1 file** from `TCGA-BH-A0DZ-11A` (duplicate CNV normal, solid tissue) — kept Peripheral Blood `-10A`

---

## Key genes

| Gene | Chr17 locus (hg38) | Expected alteration in BRCA |
|------|--------------------|-----------------------------|
| ERBB2 | 17q12 (39.7 Mb) | Amplification (~15–20% of tumors) |
| TP53 | 17p13.1 (7.7 Mb) | Deletion / LOH (~35%) |
| BRCA1 | 17q21.31 (43.0 Mb) | Deletion / LOH (~15%) |

---

## TCGA data access

| Data type | Format | Access | Download |
|-----------|--------|--------|----------|
| Masked Copy Number Segment | TXT | Open ✅ | `gdc-client` |
| Gene Expression Quantification | TSV | Open ✅ | `gdc-client` |
| Methylation Beta Value | TXT | Open ✅ | `gdc-client` |
| WES / RNA-seq BAM (unaligned) | BAM | Controlled (dbGaP) | `gdc-client` + token |

All files were obtained from the [GDC Data Portal](https://portal.gdc.cancer.gov/)
under project **TCGA-BRCA**, open access tier.

---

## Quickstart

```bash
# 1. Create and activate conda environment
conda env create -f environment.yml
conda activate tcga-brca

# 2. Place manifest and sample table in data/raw/
cp manifest_clean.txt master_sample_table.tsv data/raw/

# 3. Download and organize files (~300 MB total)
bash scripts/00_download.sh

# 4. CNV analysis
Rscript scripts/04_cnv_analysis.R

# 5. RNA-seq analysis  [future]
Rscript scripts/05_rnaseq.R

# 6. Methylation analysis  [future]
Rscript scripts/06_methylation.R

# 7. Multi-omics integration  [future]
Rscript scripts/07_integration.R
```

---

## Environment

All dependencies are managed via Conda. See `environment.yml` for the full
specification covering all three projects plus the future integration module.

```bash
# Create
conda env create -f environment.yml

# Export locked versions after creation (for full reproducibility)
conda env export > environment_locked.yml

# Recreate on a new machine
conda env create -f environment_locked.yml
```

---

## Notes

- **Reference genome**: all GDC open-access files are aligned to **GRCh38/hg38**. Coordinates in the TCGA 2012 paper use hg19 — do not mix liftover positions with these files.
- **CNV data type**: `Masked Copy Number Segment` is derived from SNP array (Affymetrix GenomeWideSNP_6), not WES. It provides genome-wide segmented log2 ratios suitable for gene-level CNV analysis without requiring alignment.
- **RNA-seq counts**: files use the **STAR augmented** format with four count columns (unstranded, stranded_first, stranded_second, TPM). Use `unstranded` counts as input to DESeq2.
- **Scalability**: scripts iterate over files in `data/` directories. Adding more samples requires only updating the manifest and re-running — no script changes needed.

---
## Skills demonstrated

- R-based analysis of biological datasets
- DESeq2 differential-expression analysis
- CNV analysis and visualization
- DNA methylation analysis
- TCGA/GDC data organization
- Reproducible project structure with Git and Conda
- Biological interpretation of multi-layer molecular data
---

