CuratedAtlasQueryR
================

<!-- badges: start -->

[![Lifecycle:maturing](https://img.shields.io/badge/lifecycle-maturing-blue.svg)](https://www.tidyverse.org/lifecycle/#maturing)
<!-- badges: end -->

`CuratedAtlasQuery` is a query interface that allow the programmatic
exploration and retrieval of the harmonised, curated and reannotated
CELLxGENE single-cell human cell atlas. Data can be retrieved at cell,
sample, or dataset levels based on filtering criteria.

<img src="man/figures/logo.png" width="120x" height="139px" />

<img src="man/figures/svcf_logo.jpeg" width="155x" height="58px" /><img src="man/figures/czi_logo.png" width="129px" height="58px" /><img src="man/figures/bioconductor_logo.jpg" width="202px" height="58px" /><img src="man/figures/vca_logo.png" width="219px" height="58px" /><img src="man/figures/nectar_logo.png" width="180px" height="58px" />

[website](https://stemangiola.github.io/CuratedAtlasQueryR)

# Query interface

## Installation

``` r
devtools::install_github("stemangiola/CuratedAtlasQueryR")
```

## Load the package

``` r
library(CuratedAtlasQueryR)
```

## Load and explore the metadata

### Load the metadata

``` r
metadata  = get_metadata()

metadata
#> # Source:   table</stornext/Home/data/allstaff/m/mangiola.s/.cache/R/CuratedAtlasQueryR/metadata.0.2.3.parquet> [?? x 56]
#> # Database: DuckDB 0.7.0 [unknown@Linux 3.10.0-1160.81.1.el7.x86_64:R 4.2.0/:memory:]
#>    cell_ sample_ cell_…¹ cell_…² confi…³ cell_…⁴ cell_…⁵ cell_…⁶ sampl…⁷ _samp…⁸
#>    <chr> <chr>   <chr>   <chr>     <dbl> <chr>   <chr>   <chr>   <chr>   <chr>  
#>  1 AAAC… 689e2f… basal … basal_…       1 <NA>    <NA>    <NA>    f297c7… D17PrP…
#>  2 AAAC… 689e2f… basal … basal_…       1 <NA>    <NA>    <NA>    f297c7… D17PrP…
#>  3 AAAC… 689e2f… lumina… lumina…       1 <NA>    <NA>    <NA>    930938… D17PrP…
#>  4 AAAC… 689e2f… lumina… lumina…       1 <NA>    <NA>    <NA>    930938… D17PrP…
#>  5 AAAC… 689e2f… basal … basal_…       1 <NA>    <NA>    <NA>    f297c7… D17PrP…
#>  6 AAAC… 689e2f… basal … basal_…       1 <NA>    <NA>    <NA>    f297c7… D17PrP…
#>  7 AAAC… 689e2f… basal … basal_…       1 <NA>    <NA>    <NA>    f297c7… D17PrP…
#>  8 AAAC… 689e2f… basal … basal_…       1 <NA>    <NA>    <NA>    f297c7… D17PrP…
#>  9 AAAC… 689e2f… lumina… lumina…       1 <NA>    <NA>    <NA>    930938… D17PrP…
#> 10 AAAC… 689e2f… basal … basal_…       1 <NA>    <NA>    <NA>    f297c7… D17PrP…
#> # … with more rows, 46 more variables: assay <chr>,
#> #   assay_ontology_term_id <chr>, file_id_db <chr>,
#> #   cell_type_ontology_term_id <chr>, development_stage <chr>,
#> #   development_stage_ontology_term_id <chr>, disease <chr>,
#> #   disease_ontology_term_id <chr>, ethnicity <chr>,
#> #   ethnicity_ontology_term_id <chr>, experiment___ <chr>, file_id <chr>,
#> #   is_primary_data_x <chr>, organism <chr>, organism_ontology_term_id <chr>, …
```

### Explore the number of datasets per tissue

``` r
metadata |>
  dplyr::distinct(tissue, dataset_id) |> 
  dplyr::count(tissue)
#> # Source:   SQL [?? x 2]
#> # Database: DuckDB 0.7.0 [unknown@Linux 3.10.0-1160.81.1.el7.x86_64:R 4.2.0/:memory:]
#>    tissue                          n
#>    <chr>                       <dbl>
#>  1 peripheral zone of prostate    10
#>  2 transition zone of prostate    10
#>  3 blood                          47
#>  4 intestine                      18
#>  5 middle temporal gyrus          24
#>  6 heart left ventricle           46
#>  7 apex of heart                  16
#>  8 heart right ventricle          16
#>  9 left cardiac atrium             7
#> 10 interventricular septum        16
#> # … with more rows
```

## Download single-cell RNA sequencing counts

### Query raw counts

``` r

single_cell_counts = 
    metadata |>
    dplyr::filter(
        ethnicity == "African" &
        stringr::str_like(assay, "%10x%") &
        tissue == "lung parenchyma" &
        stringr::str_like(cell_type, "%CD4%")
    ) |>
    get_SingleCellExperiment()
#> ℹ Realising metadata.
#> ℹ Synchronising files
#> ℹ Reading files.
#> ℹ Compiling Single Cell Experiment.

single_cell_counts
#> class: SingleCellExperiment 
#> dim: 35615 1571 
#> metadata(0):
#> assays(2): counts cpm
#> rownames(35615): TSPAN6 TNMD ... LNCDAT HRURF
#> rowData names(0):
#> colnames(1571): ACAGCCGGTCCGTTAA_F02526_1 GGGAATGAGCCCAGCT_F02526_1 ...
#>   TACAACGTCAGCATTG_SC84_1 CATTCGCTCAATACCG_F02526_1
#> colData names(56): sample_ cell_type ... updated_at_y original_cell_id
#> reducedDimNames(0):
#> mainExpName: NULL
#> altExpNames(0):
```

### Query counts scaled per million

This is helpful if just few genes are of interest, as they can be
compared across samples.

``` r
single_cell_counts = 
    metadata |>
    dplyr::filter(
        ethnicity == "African" &
        stringr::str_like(assay, "%10x%") &
        tissue == "lung parenchyma" &
        stringr::str_like(cell_type, "%CD4%")
    ) |>
    get_SingleCellExperiment(assays = "cpm")
#> ℹ Realising metadata.
#> ℹ Synchronising files
#> ℹ Reading files.
#> ℹ Compiling Single Cell Experiment.

single_cell_counts
#> class: SingleCellExperiment 
#> dim: 35615 1571 
#> metadata(0):
#> assays(1): cpm
#> rownames(35615): TSPAN6 TNMD ... LNCDAT HRURF
#> rowData names(0):
#> colnames(1571): ACAGCCGGTCCGTTAA_F02526_1 GGGAATGAGCCCAGCT_F02526_1 ...
#>   TACAACGTCAGCATTG_SC84_1 CATTCGCTCAATACCG_F02526_1
#> colData names(56): sample_ cell_type ... updated_at_y original_cell_id
#> reducedDimNames(0):
#> mainExpName: NULL
#> altExpNames(0):
```

### Extract only a subset of genes

``` r
single_cell_counts = 
    metadata |>
    dplyr::filter(
        ethnicity == "African" &
        stringr::str_like(assay, "%10x%") &
        tissue == "lung parenchyma" &
        stringr::str_like(cell_type, "%CD4%")
    ) |>
    get_SingleCellExperiment(assays = "cpm", features = "PUM1")
#> ℹ Realising metadata.
#> ℹ Synchronising files
#> ℹ Reading files.
#> ℹ Compiling Single Cell Experiment.

single_cell_counts
#> class: SingleCellExperiment 
#> dim: 1 1571 
#> metadata(0):
#> assays(1): cpm
#> rownames(1): PUM1
#> rowData names(0):
#> colnames(1571): ACAGCCGGTCCGTTAA_F02526_1 GGGAATGAGCCCAGCT_F02526_1 ...
#>   TACAACGTCAGCATTG_SC84_1 CATTCGCTCAATACCG_F02526_1
#> colData names(56): sample_ cell_type ... updated_at_y original_cell_id
#> reducedDimNames(0):
#> mainExpName: NULL
#> altExpNames(0):
```

### Extract the counts as a Seurat object

This convert the H5 SingleCellExperiment to Seurat so it might take long
time and occupy a lot of memory depending on how many cells you are
requesting.

``` r
single_cell_counts = 
    metadata |>
    dplyr::filter(
        ethnicity == "African" &
        stringr::str_like(assay, "%10x%") &
        tissue == "lung parenchyma" &
        stringr::str_like(cell_type, "%CD4%")
    ) |>
    get_seurat()
#> ℹ Realising metadata.
#> ℹ Synchronising files
#> ℹ Reading files.
#> ℹ Compiling Single Cell Experiment.
#> Warning: Non-unique features (rownames) present in the input matrix, making
#> unique

single_cell_counts
#> An object of class Seurat 
#> 35615 features across 1571 samples within 1 assay 
#> Active assay: originalexp (35615 features, 0 variable features)
```

## Save your `SingleCellExperiment`

The returned `SingleCellExperiment` can be saved with two modalities, as
`.rds` or as `HDF5`.

### Saving as RDS (fast saving, slow reading)

Saving as `.rds` has the advantage that is very fast, the `.rds` file
occupies very little disk space as it only stored the links for the
files in your ache.

However it has the disadvantage that for big `SingleCellExperiment`
objects, which merge a lot of HDF5 from your `get_SingleCellExperiment`
the display and manipulation is going to be slow.

``` r
single_cell_counts |> saveRDS("single_cell_counts.rds")
```

### Saving as HDF5 (slow saving, fast reading)

Saving as `.rds` has the advantage that rewrites on disk a monolithic
`HDF5` and so displaying and manipulating large `SingleCellExperiment`
objects, which merge a lot of HDF5 from your `get_SingleCellExperiment`,
is going to be fast.

However it has the disadvantage that the files are going to be larger as
they include the count information, and the saving process is going to
be slow for large objects.

``` r
single_cell_counts |> saveHDF5SummarizedExperiment("single_cell_counts")
```

## Visualise gene transcription

We can gather all CD14 monocytes cells and plot the distribution of
HLA-A across all tissues

``` r
library(tidySingleCellExperiment)
library(ggplot2)

metadata |>
  # Filter and subset
  filter(cell_type_harmonised=="cd14 mono") |>

  # Get counts per million for HCA-A gene
  get_SingleCellExperiment(assays = "cpm", features = "HLA-A") |> 
  
  # Plot (styling code have been omitted)
  join_features("HLA-A", shape = "wide") |> 
  ggplot(aes( disease, `HLA.A`,color = file_id)) +
  geom_jitter(shape=".") 
```

<img src="man/figures/HLA_A_disease_plot.png" width="525" />

``` r

metadata |> 
    
  # Filter and subset
  filter(cell_type_harmonised=="nk") |> 

  # Get counts per million for HCA-A gene 
  get_SingleCellExperiment(assays = "cpm", features = "HLA-A") |> 

    # Plot (styling code have been omitted)
  join_features("HLA-A", shape = "wide") |> 
  ggplot(aes( tissue_harmonised, `HLA.A`,color = file_id)) +
  geom_jitter(shape=".") 
```

<img src="man/figures/HLA_A_tissue_plot.png" width="525" />

# Cell metadata

Dataset-specific columns (definitions available at
cellxgene.cziscience.com)

`cell_count`, `collection_id`, `created_at.x`, `created_at.y`,
`dataset_deployments`, `dataset_id`, `file_id`, `filename`, `filetype`,
`is_primary_data.y`, `is_valid`, `linked_genesets`,
`mean_genes_per_cell`, `name`, `published`, `published_at`,
`revised_at`, `revision`, `s3_uri`, `schema_version`, `tombstone`,
`updated_at.x`, `updated_at.y`, `user_submitted`, `x_normalization`

Sample-specific columns (definitions available at
cellxgene.cziscience.com)

`sample_`, `sample_name`, `age_days`, `assay`, `assay_ontology_term_id`,
`development_stage`, `development_stage_ontology_term_id`, `ethnicity`,
`ethnicity_ontology_term_id`, `experiment___`, `organism`,
`organism_ontology_term_id`, `sample_placeholder`, `sex`,
`sex_ontology_term_id`, `tissue`, `tissue_harmonised`,
`tissue_ontology_term_id`, `disease`, `disease_ontology_term_id`,
`is_primary_data.x`

Cell-specific columns (definitions available at
cellxgene.cziscience.com)

`cell_`, `cell_type`, `cell_type_ontology_term_idm`,
`cell_type_harmonised`, `confidence_class`,
`cell_annotation_azimuth_l2`, `cell_annotation_blueprint_singler`

Through harmonisation and curation we introduced custom column, not
present in the original CELLxGENE metadata

- `tissue_harmonised`: a coarser tissue name for better filtering
- `age_days`: the number of days corresponding to the age
- `cell_type_harmonised`: the consensus call identity (for immune cells)
  using the original and three novel annotations using Seurat Azimuth
  and SingleR
- `confidence_class`: an ordinal class of how confident
  `cell_type_harmonised` is. 1 is complete consensus, 2 is 3 out of four
  and so on.  
- `cell_annotation_azimuth_l2`: Azimuth cell annotation
- `cell_annotation_blueprint_singler`: SingleR cell annotation using
  Blueprint reference
- `cell_annotation_blueprint_monaco`: SingleR cell annotation using
  Monaco reference
- `sample_id_db`: Sample subdivision for internal use
- `file_id_db`: File subdivision for internal use
- `sample_`: Sample ID
- `sample_name`: How samples were defined

# RNA abundance

The `raw` assay includes RNA abundance in the positive real scale (not
transformed with non-linear functions, e.g. log sqrt). Originally
CELLxGENE include a mix of scales and transformations specified in the
`x_normalization` column.

The `cpm` assay includes counts per million.

# Installation and getting-started problems

**Problem:** Default R cache path including non-standard characters
(e.g. dash)

``` r
get_metadata()

# Error in `db_query_fields.DBIConnection()`:
# ! Can't query fields.
# Caused by error:
# ! Parser Error: syntax error at or near "/"
# LINE 2: FROM /Users/bob/Library/Cach...
```

**Solution:** Setup custom cache path (e.g. user home directory)

``` r
get_metadata(cache_directory = path.expand('~'))
```

**Problem:** namespace ‘dbplyr’ 2.2.1 is being loaded, but \>= 2.3.0 is
required

**Solution:** Install new dbplyr

``` r
install.packages("dbplyr")
```

------------------------------------------------------------------------

This project has been funded by

- *Silicon Valley Foundation* CZF2019-002443
- *Bioconductor core funding* NIH NHGRI 5U24HG004059-18
- *Victoria Cancer Agency* ECRF21036
- *Australian National Health and Medical Research Council* 1116955
- *The Lorenzo and Pamela Galli Medical Research Trust*
