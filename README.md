# README
# Acute Myeloid Leukemia Heatmap

This repository contains an analysis of Acute Myeloid Leukemia (AML) using a heatmap. The analysis was adapted from the refine.bio-examples notebook.

## Dataset

The data used in this analysis is an AML sample dataset obtained from [refine.bio](https://www.refine.bio/experiments/SRP070849). The dataset contains RNA-sequencing results for types of AML under controlled treatment conditions. The data is pre-processed and quantile normalized.

## Setup

Before running the analysis, you need to set up the analysis folders. The following R code can be used to create the necessary directories:

```r
# Create the data folder if it doesn't exist
if (!dir.exists("data")) {
  dir.create("data")
}

# Define the file path to the plots directory
plots_dir <- "plots"

# Create the plots folder if it doesn't exist
if (!dir.exists(plots_dir)) {
  dir.create(plots_dir)
}

# Define the file path to the results directory
results_dir <- "results"

# Create the results folder if it doesn't exist
if (!dir.exists(results_dir)) {
  dir.create(results_dir)
}
```

## Libraries

The analysis uses the `pheatmap` library for clustering and creating a heatmap. This library can be installed and attached to the environment using the following R code:

```r
if (!("pheatmap" %in% installed.packages())) {
  # Install pheatmap
  install.packages("pheatmap", update = FALSE)
}

# Attach the `pheatmap` library
library(pheatmap)

# We will need this so we can use the pipe: %>%
library(magrittr)

# Set the seed so our results are reproducible:
set.seed(12345)
```

## Data Import and Preparation

The R code below reads in both the metadata and data TSV files and adds them as data frames to your environment. It also ensures that the metadata and data are in the same sample order.

```r
# Read in metadata TSV file
metadata <- readr::read_tsv(metadata_file)

# Read in data TSV file
expression_df <- readr::read_tsv(data_file) %>%
  # Here we are going to store the gene IDs as row names so that
  # we can have only numeric values to perform calculations on later
  tibble::column_to_rownames("Gene")

# Make the data in the order of the metadata
expression_df <- expression_df %>%
  dplyr::select(metadata$refinebio_accession_code)

# Check if this is in the same order
all.equal(colnames(expression_df), metadata$refinebio_accession_code)
```

## Clustering Heatmap

The analysis uses the `pheatmap` package to create a heatmap of the data. The heatmap is created by selecting genes in the upper quartile of variance. The code for this is as follows:

```r
# Calculate the variance for each gene
variances <- apply(expression_df, 1, var)

# Determine the upper quartile variance cutoff value
upper_var <- quantile(variances, 0.75)

# Filter the data choosing only genes whose variances are in the upper quartile
df_by_var <- data.frame(expression_df) %>%
  dplyr::filter(variances > upper_var)
```

The heatmap is then saved as a PNG file:

```r
# Open a PNG file
png(file.path(
  plots_dir,
  "aml_heatmap.png" # Replace with a relevant file name
))

# Print the heatmap
heatmap_annotated

# Close the PNG file:
dev.off()
```

## Session Info

The R session info can be printed using the following R code:

```r
# Print session info
sessioninfo::session_info()
```

## References

- [refine.bio-examples notebook](https://alexslemonade.github.io/ref