# miRNA Target Enrichment Analysis (Sylamer-style)

## What this script does

This script checks whether the genes a miRNA is predicted to target are unevenly spread across a ranked gene list, instead of spread out randomly. The gene list is ranked from most upregulated to most downregulated in a KO vs WT comparison. If the miRNA's predicted targets cluster near one end of that list more than random genes would, that supports the miRNA affecting those genes.

The script produces one plot for each of three tiers, which describe how strong the predicted match is between the miRNA and a gene:
- **6mer** = weakest match, includes the most genes
- **7mer** = medium match
- **8mer** = strongest match, includes the fewest genes

## What files you need

1. **A TargetScan file** (.xlsx) listing predicted target genes. It typically has a separate sheet for each mature form ("arm") of the miRNA, such as 3p and 5p.
2. **A differential expression results file** (.xlsm or .xlsx) from your KO vs WT comparison, usually from DESeq2. It needs gene names, a ranking statistic, and p-values.
3. **R**, with these packages installed: `readxl`, `dplyr`, `tidyr`, `ggplot2`, `ggrepel`, `stringr`. Install any missing ones with `install.packages("readxl")`, etc.

## What you need to change for a new dataset

The calculations and plotting code stay the same. What needs to change is the file names, sheet names, and column names that point to your specific data.

### 1. File locations

```r
targetscan_path <- "path/to/your/targetscan_file.xlsx"
deg_path  <- "path/to/your/deg_file.xlsm"
deg_sheet <- "sheet_name_in_deg_file"
```
Replace these with your file locations. `deg_sheet` must match the sheet name inside the file, not the file name itself.

### 2. miRNA arm sheets

```r
target_sheets <- c(
  "3p.1" = "3p.1_TargetScanMouse_8.0",
  "3p.2" = "3p.2_TargetScanMouse_8.0",
  "5p"   = "5p_TargetScanMouse_8.0"
)
```
This lists the sheet name for each arm and the short label used in the plot. Check the actual sheet names in your TargetScan file, since some miRNAs only have one or two arms. Update both the labels and the sheet names to match exactly, including spelling and capitalization.

If the number of arms changes, also update the colors listed under `scale_color_manual` further down so each arm has one.

### 3. Column names in your results file

```r
mutate(pvalue = ..., padj = ...) %>%
filter(!is.na(external_gene_name), !is.na(stat))
```
This assumes column names from DESeq2 output: `external_gene_name`, `stat`, `pvalue`, `padj`. Other tools like limma or edgeR use different column names. Check the column headers in your file and update these names everywhere they appear in the script.

### 4. TargetScan column names

```r
"6mer" = df[["6mer sites"]]
"7mer" = rowSums(df[, c("Conserved 7mer-m8 sites", ...)])
"8mer" = rowSums(df[, c("Conserved 8mer sites", ...)])
```
These are exact column headers from a specific TargetScan release. If they don't match your file, the script won't produce an error — it will quietly treat those values as missing or zero, and the plot will look normal but be wrong. Check the column headers in your TargetScan file before trusting the output.


