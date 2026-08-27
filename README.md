# sylamer-analysis
# Sylamer-style miRNA Target Enrichment Analysis

This script tests whether predicted targets of a miRNA are enriched (or depleted) among the most differentially expressed genes in a KO vs WT RNA-seq comparison. It builds a rank-based enrichment curve for each miRNA arm, compares it against an empirical null from random gene sets, and outputs one plot per site-strength tier (6mer, 7mer, 8mer).

## What you need

1. A TargetScan predicted-targets file (.xlsx), with one sheet per mature miRNA arm.
2. A differential expression results file (.xlsm or .xlsx), with gene symbols, a ranking statistic, and p-values.
3. R with `readxl`, `dplyr`, `tidyr`, `ggplot2`, `ggrepel`, `stringr` installed.

## Required edits before running on new data

### 1. File paths
```r
targetscan_path <- "path/to/your/targetscan_file.xlsx"
deg_path  <- "path/to/your/deg_file.xlsm"
deg_sheet <- "sheet_name_in_deg_file"
```
`deg_sheet` must match the actual tab name, not the filename.

### 2. miRNA arm sheets
```r
target_sheets <- c(
  "3p.1" = "3p.1_TargetScanMouse_8.0",
  "3p.2" = "3p.2_TargetScanMouse_8.0",
  "5p"   = "5p_TargetScanMouse_8.0"
)
```
This assumes three arms. Run `readxl::excel_sheets(targetscan_path)` first and rebuild this vector to match what's actually in the file — some miRNAs only have one or two annotated arms. The names on the left also feed the plot legend and colors (`scale_color_manual`), so update those too if the arm count changes.

### 3. DEG column names
```r
mutate(pvalue = ..., padj = ...) %>%
filter(!is.na(external_gene_name), !is.na(stat))
```
Assumes DESeq2 output (`external_gene_name`, `stat`, `pvalue`, `padj`). Different pipelines (limma, edgeR) use different column names and ranking statistics. Check `colnames()` on the DEG file and remap, including deciding whether `stat` is still the right column to rank on.

### 4. TargetScan site-count columns
```r
"6mer" = df[["6mer sites"]]
"7mer" = rowSums(df[, c("Conserved 7mer-m8 sites", ...)])
"8mer" = rowSums(df[, c("Conserved 8mer sites", ...)])
```
These are exact column headers from a specific TargetScan release. A mismatch here fails silently (returns 0/NA instead of an error) and produces a plot that looks fine but is wrong. Always check `colnames(df)` against these strings before trusting the output on a new TargetScan file or species.
