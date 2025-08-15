Repository description

Decitabine Response RNA-seq — Preprocessing & Analysis

This repository provides an end-to-end R-based RNA-seq workflow replicating the RNA-seq component of the multiomics study “A multiomics approach reveals RNA dynamics promote cellular sensitivity to DNA hypomethylation” (Scientific Reports, 2024). The paper investigates decitabine response mechanisms in AML using integrated multiomics and CRISPR functional genomics; here we focus on the bulk RNA-seq pipeline used for differential expression analysis. 

It includes two main components:

Preprocessing (Clean.sh)
Downloads chosen SRA runs (HL-60 ± decitabine), subsamples reads for reproducible mini-runs, builds a GENCODE (GRCh38) Salmon index, runs Salmon (with --gcBias --validateMappings), and produces gene-level counts via tximport/tximeta.

Analysis (Differential_expression.sh)
Runs DESeq2 (design ~ condition) to identify genes differentially expressed in decitabine (DAC) vs DMSO, and exports tables and simple QC/diagnostic plots.

Data

Primary data source (paper): GEO Series GSE222886 (multiomics; includes RNA-seq related to the paper). Use the SRA Run Selector to obtain run accessions. 
Nature

Convenient RNA-seq panel: GEO Series GSE222822 (RNA-seq of AML cell line panel ± decitabine). Select HL-60 samples (2× DMSO, 2× DAC) to create a compact, reproducible subset. Use the SRA Run Selector to fetch SRR IDs. 
0-www-ncbi-nlm-nih-gov.brum.beds.ac.uk

Tip: keep each gzipped FASTQ ≤2 GB for Taiga by subsampling (the scripts do this deterministically).

Reference files:

GENCODE GRCh38 (e.g., v41+) transcript FASTA & GTF (used to build the Salmon index)

Salmon index built from the above (created automatically by the scripts)

Directory output

The preprocessing script creates a structured working directory data_pre_processing/ containing:

raw/            # downloaded raw FASTQs (from SRA/ENA)
fastq/          # subsampled FASTQs (one per sample; deterministic seed)
salmon/         # per-sample Salmon quant outputs (quant.sf, meta_info.json)
counts/         # gene-level count matrix (tximport/tximeta)
de/             # DESeq2 results (results table, normalized counts, plots)
qc/             # optional FastQC reports and simple mapping summaries


Citations

Scientific Reports article (Oct 29, 2024): multiomics decitabine study. 
Nature
cancer.ucsf.edu

GEO RNA-seq panel (AML ± DAC) for convenient HL-60 selection (GSE222822). 
0-www-ncbi-nlm-nih-gov.brum.beds.ac.uk

If you want, I can also drop in a ready-to-push README.md, Clean.sh, Differential_expression.sh, and minimal R scripts (Salmon→tximport/tximeta→DESeq2) that mirror this description.

