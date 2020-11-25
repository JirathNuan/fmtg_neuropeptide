Transcriptome assembly and post-assembly processing
=====

> Server specification: 32 cores CPU and 62 GB RAM (242 Ubuntu 19.10 server)

Create analysis directory

> current directory `~/paper-2`

```bash
mkdir raw_data 01_quality_check 02_trim 03_normalization 04_assembly_trinity 05_clustering 06_translate
mkdir 01_quality_check/{before after}
```

### FASTQ quality control and adapter trimming

```bash
# Create directory
mkdir 01_quality_check

# Create and activate environment for FASTQ quality control and read trimming
conda create --name qc fastqc=0.11.8 cutadapt=2.7 multiqc=1.8 -y
conda activate qc

```

LoremLorem ipsum dolor sit amet, consectetur adipisicing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.

```bash
fastqc -t 32 -o 01_quality_check/before/ raw_data/*
```

FastQC generated 2 output file per once, html and compressed file. Then MultiQC was used to generate nice graphical visualization and combine seperated html file into a single file. This help us to interpret easily at a short time. 

```bash
cd 01_quality_check/before
multiqc --filename before_qc_multiqc_report.html
```





```bash

```

```bash

```

```bash

```

```bash

```