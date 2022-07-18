
<!-- README.md is generated from README.Rmd. Please edit that file -->

# ggcoverage - Visualize and annotate genomic coverage with ggplot2

<img src = "man/figures/ggcoverage.png" align = "right" width = "200"/>

## Introduction

The goal of `ggcoverage` is simplify the process of visualizing genomic
coverage. It contains three main parts:

-   **Load the data**: `ggcoverage` can load bam, bigwig (.bw), bedgraph
    file from various NGS data, including RNA-seq, ChIP-seq, ATAC-seq,
    et al.
-   **Create genomic coverage plot**
-   **Add annotaions**: `ggcoverage` supports four different annotaions:
    -   **gene annotaion**: Visualize genomic coverage across whole gene
    -   **transcription annotion**: Visualize genomic coverage across
        different transcripts
    -   **ideogram annotation**: Visualize the region showing on whole
        chromosome
    -   **peak annotation**: Visualize genomic coverage and peak
        identified.

`ggcoverage` utilizes `ggplot2` plotting system, so its usage is
ggplot2-style!

## Installation

`ggcoverage` is an R package distributed as part of the
[Bioconductor](http://bioconductor.org) project and
[CRAN](https://cran.r-project.org/). To install the package, start R and
enter:

``` r
# install via Bioconductor
if (!requireNamespace("BiocManager", quietly=TRUE))
  install.packages("BiocManager")
BiocManager::install("ggcoverage")

# install via CRAN
install.package("ggcoverage")

# install via Github
# install.package("remotes")   #In case you have not installed it.
remotes::install_github("showteeth/ggcoverage")
```

Once `ggcoverage` is installed, it can be loaded by the following
command.

``` r
library("rtracklayer")
library("ggcoverage")
```

## RNA-seq data

### Load the data

The RNA-seq data used here are from [Transcription profiling by high
throughput sequencing of HNRNPC knockdown and control HeLa
cells](https://bioconductor.org/packages/release/data/experiment/html/RNAseqData.HNRNPC.bam.chr14.html),
we select four sample to use as example: ERR127307\_chr14,
ERR127306\_chr14, ERR127303\_chr14, ERR127302\_chr14, and all bam files
are converted to bigwig file with
[deeptools](https://deeptools.readthedocs.io/en/develop/).

Load metadata:

``` r
# load metadata
meta.file <- system.file("extdata", "RNA-seq", "meta_info.csv", package = "ggcoverage")
sample.meta = read.csv(meta.file)
sample.meta
#>        SampleName    Type Group
#> 1 ERR127302_chr14 KO_rep1    KO
#> 2 ERR127303_chr14 KO_rep2    KO
#> 3 ERR127306_chr14 WT_rep1    WT
#> 4 ERR127307_chr14 WT_rep2    WT
```

Load track files:

``` r
# track folder
track.folder = system.file("extdata", "RNA-seq", package = "ggcoverage")
# load bigwig file
track.df = LoadTrackFile(track.folder = track.folder, format = "bw",
                         meta.info = sample.meta)
# check data
head(track.df)
#>   seqnames    start      end score    Type Group
#> 1    chr14 21572751 21630650     0 KO_rep1    KO
#> 2    chr14 21630651 21630700     1 KO_rep1    KO
#> 3    chr14 21630701 21630800     4 KO_rep1    KO
#> 4    chr14 21630801 21657350     0 KO_rep1    KO
#> 5    chr14 21657351 21657450     1 KO_rep1    KO
#> 6    chr14 21657451 21663550     0 KO_rep1    KO
```

Prepare mark region:

``` r
# create mark region
mark.region=data.frame(start=c(21678900,21732001,21737590),
                       end=c(21679900,21732400,21737650),
                       label=c("M1", "M2", "M3"))
# check data
mark.region
#>      start      end label
#> 1 21678900 21679900    M1
#> 2 21732001 21732400    M2
#> 3 21737590 21737650    M3
```

Load GTF file:

``` r
gtf.file = system.file("extdata", "used_hg19.gtf", package = "ggcoverage")
gtf.gr = rtracklayer::import.gff(con = gtf.file, format = 'gtf')
```

### Basic coverage

``` r
basic.coverage = ggcoverage(data = track.df, color = "auto", 
                            mark.region = mark.region, range.position = "out")
basic.coverage
```

<img src="man/figures/README-basic_coverage-1.png" width="100%" style="display: block; margin: auto;" />

You can also change Y axis style:

``` r
basic.coverage = ggcoverage(data = track.df, color = "auto", 
                            mark.region = mark.region, range.position = "in")
basic.coverage
```

<img src="man/figures/README-basic_coverage_2-1.png" width="100%" style="display: block; margin: auto;" />

### Add gene annotation

``` r
basic.coverage + 
  geom_gene(gtf.gr=gtf.gr)
```

<img src="man/figures/README-gene_coverage-1.png" width="100%" style="display: block; margin: auto;" />

### Add transcript annotation

``` r
basic.coverage + 
  geom_transcript(gtf.gr=gtf.gr,label.vjust = 1.5)
```

<img src="man/figures/README-transcript_coverage-1.png" width="100%" style="display: block; margin: auto;" />

### Add ideogram

``` r
basic.coverage +
  geom_gene(gtf.gr=gtf.gr) +
  geom_ideogram(genome = "hg19",plot.space = 0)
#> [1] "hg19"
#> Loading ideogram...
#> Loading ranges...
#> Scale for 'x' is already present. Adding another scale for 'x', which will
#> replace the existing scale.
```

<img src="man/figures/README-ideogram_coverage_1-1.png" width="100%" style="display: block; margin: auto;" />

``` r
basic.coverage +
  geom_transcript(gtf.gr=gtf.gr,label.vjust = 1.5) +
  geom_ideogram(genome = "hg19",plot.space = 0)
#> [1] "hg19"
#> Loading ideogram...
#> Loading ranges...
#> Scale for 'x' is already present. Adding another scale for 'x', which will
#> replace the existing scale.
```

<img src="man/figures/README-ideogram_coverage_2-1.png" width="100%" style="display: block; margin: auto;" />

## ChIP-seq data

The ChIP-seq data used here are from
[DiffBind](https://bioconductor.org/packages/release/bioc/html/DiffBind.html),
I select four sample to use as example: Chr18\_MCF7\_input,
Chr18\_MCF7\_ER\_1, Chr18\_MCF7\_ER\_3, Chr18\_MCF7\_ER\_2, and all bam
files are converted to bigwig file with
[deeptools](https://deeptools.readthedocs.io/en/develop/).

Create metadata:

``` r
# load metadata
sample.meta = data.frame(SampleName=c('Chr18_MCF7_ER_1','Chr18_MCF7_ER_2','Chr18_MCF7_ER_3','Chr18_MCF7_input'),
                         Type = c("MCF7_ER_1","MCF7_ER_2","MCF7_ER_3","MCF7_input"),
                         Group = c("IP", "IP", "IP", "Input"))
sample.meta
#>         SampleName       Type Group
#> 1  Chr18_MCF7_ER_1  MCF7_ER_1    IP
#> 2  Chr18_MCF7_ER_2  MCF7_ER_2    IP
#> 3  Chr18_MCF7_ER_3  MCF7_ER_3    IP
#> 4 Chr18_MCF7_input MCF7_input Input
```

Load track files:

``` r
# track folder
track.folder = system.file("extdata", "ChIP-seq", package = "ggcoverage")
# load bigwig file
track.df = LoadTrackFile(track.folder = track.folder, format = "bw",
                         meta.info = sample.meta)
# check data
head(track.df)
#>   seqnames    start      end   score      Type Group
#> 1    chr18 76799701 76800000 439.316 MCF7_ER_1    IP
#> 2    chr18 76800001 76800300 658.974 MCF7_ER_1    IP
#> 3    chr18 76800301 76800600 219.658 MCF7_ER_1    IP
#> 4    chr18 76800601 76800900 658.974 MCF7_ER_1    IP
#> 5    chr18 76800901 76801200   0.000 MCF7_ER_1    IP
#> 6    chr18 76801201 76801500 219.658 MCF7_ER_1    IP
```

Prepare mark region:

``` r
# create mark region
mark.region=data.frame(start=c(76822533),
                       end=c(76823743),
                       label=c("Promoter"))
# check data
mark.region
#>      start      end    label
#> 1 76822533 76823743 Promoter
```

### Basic track

``` r
basic.coverage = ggcoverage(data = track.df, color = "auto", region = "chr18:76822285-76900000", 
                            mark.region=mark.region, show.mark.label = FALSE)
basic.coverage
```

<img src="man/figures/README-basic_coverage_chip-1.png" width="100%" style="display: block; margin: auto;" />

### Add annotations

Add **gene**, **ideogram** and **peak** annotaions. To create peak
annotaion, we first **get consensus peaks** with
[MSPC](https://github.com/Genometric/MSPC), you can also use
[DEbChIP’s](https://github.com/showteeth/DEbChIP) `GetConsensusPeak`
(`MSPC`’s wrapper) to do this.

``` r
# get consensus peak file
peak.file = system.file("extdata", "ChIP-seq", "consensus.peak", package = "ggcoverage")

basic.coverage +
  geom_gene(gtf.gr=gtf.gr) +
  geom_peak(bed.file = peak.file) +
  geom_ideogram(genome = "hg19",plot.space = 0)
#> [1] "hg19"
#> Loading ideogram...
#> Loading ranges...
#> Scale for 'x' is already present. Adding another scale for 'x', which will
#> replace the existing scale.
```

<img src="man/figures/README-peak_coverage-1.png" width="100%" style="display: block; margin: auto;" />

## Code of Conduct

Please note that the `ggcoverage` project is released with a
[Contributor Code of
Conduct](https://contributor-covenant.org/version/2/0/CODE_OF_CONDUCT.html).
By contributing to this project, you agree to abide by its terms.