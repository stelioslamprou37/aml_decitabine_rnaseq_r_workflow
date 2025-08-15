# HL60 Decitabine RNA-seq (Salmon + tximport + DESeq2)

**Paper**: Ge AY, Arab A, et al. *A multiomics approach reveals RNA dynamics promote cellular sensitivity to DNA hypomethylation*. Scientific Reports (2024). doi:10.1038/s41598-024-77314-9

**Data**: RNA‑seq (Homo sapiens, HL‑60) decitabine vs DMSO. The paper’s Data Availability points to GEO accession **GSE222886** (RNA‑seq; meRIP‑seq; Ribo‑seq). For decitabine/DMSO bulk RNA‑seq across AML cell lines (including HL‑60) see **GSE222822** (public Sept 11, 2023). You will fetch specific **SRR** run accessions from the GEO/SRA pages (Run Selector).

**Workflow summary** (matches the paper’s described approach):
- Quantification: **Salmon** (quasi‑mapping) against human **GENCODE** transcriptome
- Import to R: **tximport**
- Differential expression: **DESeq2**
- Miniaturization: deterministic subsampling to **50,000 paired fragments** per sample (fixed seed)

---

## 0) What you need installed (locally or in Taiga runner)
- `sra-tools` (for `prefetch`, `fasterq-dump`) — or you can download FASTQs via EBI FTP.
- `seqtk` (for subsampling)
- `salmon` (>=1.10)
- `gzip`, `bash`, `R` (>=4.2) and these R packages: `tximport`, `DESeq2`, `readr`, `yaml`, `jsonlite`, `optparse`, `BiocManager`

> Tip (R): if tximport/DESeq2 aren’t present, the R script will install them via BiocManager.

---

## 1) Get the run accessions (SRR)
1. Open the GEO page for **GSE222886** (paper’s data) or **GSE222822** (decitabine AML panel).
2. Click **“SRA Run Selector”** and download an accession list.
3. Pick **two replicates** of **DMSO** and **two** of **decitabine** for **HL‑60** (4 runs total).
4. Put the chosen **SRR** IDs into `data/sra_runs.tsv` like this (tab‑separated):
   ```
   sample_id	condition	srr
   DMSO_rep1	DMSO	SRRXXXXXXXXX
   DMSO_rep2	DMSO	SRRXXXXXXXXX
   DAC_rep1	DAC	SRRXXXXXXXXX
   DAC_rep2	DAC	SRRXXXXXXXXX
   ```

If you already have direct FASTQ URLs (e.g., from EBI/ENA), you can skip SRA tools and just set the `fastq_1` and `fastq_2` paths in `data/fastq_paths.tsv` (see below).

---

## 2) Run the workflow (fetch → subsample → index → quantify → DE → answers)
From the `workflow/` folder run:

```bash
bash run_all.sh
```

This does the following:
- **01_fetch_fastq.sh** — downloads FASTQs for the SRR IDs (or uses ENA/FTP links you provide).
- **02_subsample.sh** — subsamples to **50,000 pairs** with **seed=42** (deterministic).
- **03_build_index.sh** — downloads **GENCODE** (version set in the script) and builds a Salmon index.
- **04_quantify.sh** — runs Salmon for each sample (paired‑end assumed; edit if single‑end).
- **05_tximport_deseq2.R** — imports Salmon -> gene counts, runs DESeq2 (formula `~ condition`), writes results.
- **06_make_answers.R** — **auto‑generates `answers.yaml`** for your rubric from the run outputs.

Outputs land in `workflow/out/` and summaries in `workflow/out/answers/`.

---

## 3) What to upload to Taiga
You will create **two Taskforge problems**:

### A) *with‑workflow* problem
- ZIP/upload this whole folder (the `with_workflow` package) including:
  - `workflow/` (all scripts)
  - `data/` **with your subsampled FASTQs** under `data/fastq_subsampled/` (<=2GB each; gzip OK)
  - `questions.yaml`, `metadata.yaml`, `README.md`
  - **Do NOT** include `answers.yaml` in the upload.
- Task prompt: “Using the data in the `\data` folder **and** provided workflows in the `\workflow` folder, answer the following questions…” (paste the 5 Qs from `questions.yaml`).

### B) *no‑workflow* problem
- ZIP/upload the **same**, but **exclude** the `workflow/` folder.
- For the rubric, open your *with‑workflow* attempt transcript or the file `workflow/out/answers/answers.yaml` produced by `06_make_answers.R` and **paste those values** into Taiga’s “complex criterion breakdown”.

---

## 4) Folder contents
- `questions.yaml` — the 5 objective questions
- `metadata.yaml` — citation, tools/versions, data source (GEO/SRA), reference genome
- `answers.yaml` — **auto‑generated** by `workflow/06_make_answers.R`
- `workflow/` — executable pipeline
- `data/` — your FASTQs go here
   - `fastq_raw/` (downloaded)
   - `fastq_subsampled/` (50k pairs each; what Salmon uses)

---

## 5) Notes & troubleshooting
- If your FASTQs are single‑end, see comments inside `04_quantify.sh` and adjust `salmon quant` flags.
- If you can’t use SRA tools on Taiga, you can supply **FTP/HTTP** FASTQ URLs via `data/fastq_paths.tsv`.
- Keep each gzipped FASTQ under **2GB** for Taiga upload. If larger, lower `SUBSAMPLE_PAIRS` in `02_subsample.sh`.
- R installs missing packages on first run; if your runner blocks installs, pre‑install locally and re‑zip.

---

## 6) License
MIT for scripts here. Reference data (GENCODE) and GEO sequences follow their own licenses.
