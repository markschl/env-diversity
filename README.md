# Analysis of environmental diversity using amplicon sequencing

This pipeline system uses [Snakemake](https://snakemake.github.io/) to create and integrate different amplicon pipelines. Their results can be easily compared due to a common file structure of the output directories, which may serve as input e.g. for R-based analyses.

*Note:* The repository was created to fit the needs of the author and is tailored for the analysis of paired-end Illumina data from fungi or arthropods, but it may be extended further (see [below](#still-not-finished)).

*Pipelines*

- [USEARCH](https://www.drive5.com/usearch/manual)/[VSEARCH](https://github.com/torognes/vsearch)-based amplicon pipeline using UNOISE3 for obtaining ASVs (implemented here from scratch with Snakemake)
- [QIIME2](https://qiime2.org) (DADA2 denoising)
- [Amptk](https://github.com/nextgenusfs/amptk) (UNOISE3 and DADA2)
- (more may follow)

*Features*

- Running different pipelines or variations of the same pipeline using different sets of parameters, which are defined in a [`configuration file`](config/config.yaml).
- Simultaneous processing of amplicons generated using different primer sets
- Multiple taxonomic assignment methods can be applied to the denoised dataset using marker-specific reference databases; currently implemented: [UNITE](https://unite.ut.ee) for Eukaryote ITS and [Midori](http://www.reference-midori.info) for mitochondrial markers (more may follow)

The pipeline is tested using amplicon data from a fungal mock community ([`test` directory](test/)) and the results of the different pipelines can be compared [using a script](#comparison-of-denoisingclustering-pipelines).

The current rule graph (see also [description of target rules](Commands.md)):

![rule graph](https://raw.githubusercontent.com/markschl/env-diversity/main/rulegraph.png)


## Installing

The pipeline makes use of the [Conda package manager](https://conda.io), the installation is thus pretty straightforward ([see instructions here](INSTALL.md)).

## Configuring

The easiest is to copy the contents of the [config](config/) directory into the future analysis directory and then modify the files according to your needs. [`config.yaml`](config/config.yaml) contains all settings, while the taxonomic databases can be configured in [`taxonomy.yaml`](config/taxonomy.yaml). Please refer to the comments in these config files.

## Running

As visible in the rulegraph above, there are a few Snakemake target rules, which can be run independently (even though they also partly depend on each other). A complete list of commands is [found here](Commands.md).

### On a local computer

The following command runs the test pipeline (FASTQ files from sequencing fungal mock comunities in the [`test` directory](test/)) on a local computer.

```sh
conda activate snakemake
snakemake -c6 --use-conda --conda-prefix ~/conda -d test denoise cmp taxonomy ITS
```

Note that the `~/conda` directory is used for the installation of all additional necessary software. This allows reusing the installed software across analyzes of different datasets.

### On a computer cluster

A complete denoising run of a dataset on a HPCC with the SLURM job scheduler may look like this:

```sh
outdir=~/path/to/analysis  # must contain a config directory
conda activate snakemake
snakemake -j10 -c20 \
    --use-conda --conda-prefix ~/conda \
    --slurm \
    -d $outdir \
    --rerun-incomplete \
    denoise cmp taxonomy ITS \
    --default-resources runtime=180
```


## Analyzing in R

The R source file [`R/read_amplicon.R`](R/read_amplicon.R) provides code for reading all data from a results directory. See also the small [example analysis](test/R_example/example.md).


## Comparison of denoising/clustering pipelines

There is a separate bash script `scripts/compare_results.sh`, which creates an Excel file comparing the number of reads assigned to each 98% cluster by each pipeline. A separate workbook is created for each sample. The script requires VSEARCH, as well as R with the following packages: `ggplot2`, `tidyverse`, `data.table` and `openxlsx`.


## Still not finished...

A list of possible next steps includes:

- Enable single-end analyses and other platforms than Illumina
- Integrate more pipelines / clustering methods
- Integrate more taxonomy databases
- Testing deployment on different systems
- Improve configuration of job resources (memory, CPUs)
- The USEARCH pipeline may be moved into an extra repository to be used independently. This repo has a strong focus on the comparison of a variety of different pipelines as well as handling complex cases (e.g. several primer combinations).
- ...
