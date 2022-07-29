# Meta-scRNA-seq
Author: Michael Wang (fw262@cornell.edu)

![temp](https://user-images.githubusercontent.com/56937181/181630489-f6c5a482-b0ba-472c-8b6e-9471f7026444.png)

## Outline
We developed meta-scRNA-seq, a pipeline for unbiased detection of non-host transcriptomic information from scRNA-seq data. To achieve this, meta-scRNA-seq aligns scRNA-seq data against the host-genome reference using standard approaches, collected single-cell tagged unmapped reads, labeled them based on sequence similarity against a large metagenomic database, and demultiplexed the reads to generate a cell-by-metagenome count matrix in parallel with the standard cell-by-gene (host) matrix.

## Required Software
This workflow requires the following packages listed below. Please ensure that tool can be called from the command line (i.e. the paths to each tool is in your path variable).

### 1. [Snakemake](https://snakemake.readthedocs.io/en/stable/)
### 2. [STAR Aligner](https://github.com/alexdobin/STAR/releases)
```
conda install -c bioconda star
```
### 3. [R, version 3.6 or greater](https://www.r-project.org/)
Please also ensure that you have downloaded the following R packages. They will be used throughout the pipeline.
- [Seurat, version 3](https://satijalab.org/seurat/install.html)
- [dplyr](https://www.r-project.org/nosvn/pandoc/dplyr.html)
- [plyr](https://www.rdocumentation.org/packages/plyr/versions/1.8.7)
- [ggplot2](https://ggplot2.tidyverse.org/)
- [argparse](https://cran.r-project.org/web/packages/argparse/index.html)
### 4. [Samtools](http://www.htslib.org/)
```
conda install -c bioconda samtools
```
### 5. [Kraken2](https://ccb.jhu.edu/software/kraken2/)
Please make sure this tool is available in your working environment. Please also download the reference database.

## Procedure

### 1. Clone this repository.
Run the following command in your command line.
```
git clone https://github.com/fw262/Meta-scRNA-seq.git
```

### 2. Download required software listed above.

Please ensure to include all required software before starting.

### 3. Store or link paired end sequencing files.

Please move raw fastq files for each experiment into one data directory. Please ensure the sequence files end in "{sample}\_R1_001.fastq.gz" and "{sample}\_R1_001.fastq.gz" in your data directory.

### 4. Create the STAR reference of the host genome.

### 5. Edit the config.yaml file for your experiment.

Please change the variable names in the config.yaml as required for your analysis. This includes the following changes:
- **Samples**: Samples prefix (before the \_R1_001.fastq.gz)
- **STAR_IND**: Path to your STAR generated index folder.
- **DATADIR**: Path to where the sequencing samples ({sample}\_R1_001.fastq.gz) are stored.
- **PIPELINE_MAJOR**: Directory where the outputs (expression matrices, plots) are stored.
- **GLOBAL**: Define global variables for pipeline including number of mismatches allowed in STAR, cell barcode base pair range in read 1, and UMI base pair range in read 1.
- **STAREXEC**: Path to STAR.
- **KRAKEN**: Path to Kraken2.
- **KRAKEN_DB**: Path to Kraken2 database.

- **CORES**: Number of cores used in each step of the pipeline. To run multiple samples in parallel, please specify total number of cores in the snakemake command (i.e. "snakemake -j {total cores}").

### 6. Run snakemake with the command "snakemake".

Please ensure the Snakefile and config.yaml files as well as the scripts folder are in the directory where you intend to run the pipeline.

## Output

- RefFlat format of TAR features with and without consideration of directionality stored in "**TAR_reads.bed.gz.withDir.refFlat.refFlat**" and "**TAR_reads.bed.gz.noDir.refFlat.refFlat**".
- Digital expression matrix for gene features is stored in "**results_out/{sample}/{sample}\_gene_expression_matrix.txt.gz**".
- Digital expression matrix for TAR features, without consideration of TAR directionality relative to annotated gene features, is stored in "**results_out/{sample}/{sample}\_TAR_expression_matrix_noDir.txt.gz**".
- Digital expression matrix for TAR features, with consideration of TAR directionality relative to annotated gene features, is stored in "**results_out/{sample}/{sample}\_TAR_expression_matrix_withDir.txt.gz**".
- A list of differentially expressed genes and uTARs in "**results_out/{sample}/{sample}\_diffMarkers.txt**".
- A list of differentially expressed uTARs and their labels based on BLASTn results in "**results_out/{sample}/{sample}\_diffuTARMarkersLabeled.txt**".
- Results of the BLASTn analysis for differentially expressed uTARs in "**results_out/{sample}/{sample}\_blastResults.txt**".

### Format of TAR feature label

TAR features, listed in the refFlat and expression matrix files, are named based on their position, total coverage, and whether they overlap with an existing gene annotation. Examples listed below

- **chr3_40767549_40767699_+\_187_0** means that this TAR feature is located at chr3:40767549-40767699 on the positive strand with a total read coverage of 187. The "\_0" means that this is a **uTAR** feature, no overlap with an existing gene annotation.
- **chr6_42888199_42888349_-\_983_RPS6KA2_+\_1** means that this TAR feature is located at chr6:42888199-42888349 on the negative strand with a total read coverage of 983. This feature overlaps in genomic position with the gene annotated as RPS6KA2, which is annotated on the positive strand. The "\_1" means that this is an **aTAR** feature overlapping an existing gene annotation without considering directionality.

## Data Availability

The following publicly available datasets are used in the manuscript.

- [Human PBMC data](https://support.10xgenomics.com/single-cell-gene-expression/datasets/2.1.0/pbmc4k?)
- [Mouse Atlas](https://tabula-muris.ds.czbiohub.org/), [GSE109774](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE109774)
- Naked mole rat spleen data [GSE132642](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE132642)
- Sea urchin embryo data [GSE134350](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE134350)
- Chicken embryonic heart [GSE149457](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE149457)
- Mouse lemur

## Frequently Asked Questions (FAQs)

### 1. Error in rule calcHMMrefFlat stating "Error in read.table(file = file, header = header, sep = sep, quote = quote,  : no lines available in input".

Please refer to the "SingleCellHMM_Run_combined_bam_HMM_features.log" created in the same directory as your Snakefile. You will likely see the following error:
```
cannot read: chr*_HMM.bed: No such file or directory
```
Please manually install [groHMM](https://bioconductor.org/packages/release/bioc/html/groHMM.html) and [rtracklayer](https://bioconductor.org/packages/release/bioc/html/rtracklayer.html) and make sure you can load these packages in R. This error indicates that the groHMM R script did not finish running to generate groHMM bed files.
