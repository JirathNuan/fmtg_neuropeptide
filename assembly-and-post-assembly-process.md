# Transcriptome assembly and post-assembly processing


> Remarks: all following commands are performed on 32 cores CPU and 62 GB RAM (Ubuntu 19.10 server)

Create analysis directories

> current directory `~/fmtg_rnaseq`

```bash
mkdir 00_raw_data 01_quality_check 02_trim 03_normalization 04_assembly_trinity 05_clustering 06_translate
mkdir 01_quality_check/{before,after}

```

## FASTQ quality control and adapter trimming

Creating environments
```bash
# Create and activate environment for FASTQ quality control and read trimming
conda create --name qc fastqc=0.11.8 cutadapt=2.7 multiqc=1.8 -y
conda activate qc
```

Run fastQC
```bash
fastqc -t 32 -o 01_quality_check/before/ 00_raw_data/*
```

FastQC generated 2 output file per once, html and compressed file. Then MultiQC was used to generate nice graphical visualization and combine seperated html file into a single file. This help us to interpret easily at a short time. 

```bash
cd 01_quality_check/before
multiqc --filename before_qc_multiqc_report.html .

cd ../../
```

Next, Cutadapt was performed to trim the sequencing adapters, reads containing quality lower than 20, and preserve reads that long at least 25 bases as follows

```bash
### Quality control
for (( i = 0; i <= 3; i++ )); do
for (( j = 1; j <= 2; j++ )); do
cutadapt \
--cores 32 \
--quality-cutoff 20,20 --minimum-length 25 \
-a AGATCGGAAGAGCACACGTCTGAACTCCAGTCA -A AGATCGGAAGAGCGTCGTGTAGGGAAAGAGTGT \
-o '02_trim/trimmed.TgS'$i'_rep'$j'_1.fq' \
-p '02_trim/trimmed.TgS'$i'_rep'$j'_2.fq' \
'00_raw_data/TgS'$i'_rep'$j'_1.fq' \
'00_raw_data/TgS'$i'_rep'$j'_2.fq' \
> '02_trim/TgS'$i'_rep'$j'.log'
done
done
```

Check quality of FASTQ file after trimming

```bash
### Quality check after QC
fastqc -t 32 -o 01_quality_check/after/ 02_trim/trimmed.TgS*.fq

cd 01_quality_check/after
multiqc --filename after_qc_multiqc_report.html .

cd ../../
```

After finish QC step, deactivate the current environment

```bash
conda deactivate
```


## __*in silico*__ read normalization

Create environment for Trinity

```bash
# Create and activate environment for Trininty
conda create --name trinity trinity=2.8.5 -y
conda activate trinity
```
Run _in silico_ read normalization
```bash
### in silico read normalization before assembly
insilico_read_normalization.pl --CPU 32 --JM 62G --seqType fq \
--max_cov 30 --pairs_together \
--left \
02_trim/trimmed.S0_rep1_1.fq,\
02_trim/trimmed.S0_rep2_1.fq,\
02_trim/trimmed.S1_rep1_1.fq,\
02_trim/trimmed.S1_rep2_1.fq,\
02_trim/trimmed.S2_rep1_1.fq,\
02_trim/trimmed.S2_rep2_1.fq,\
02_trim/trimmed.S3_rep1_1.fq,\
02_trim/trimmed.S3_rep2_1.fq \
--right \
02_trim/trimmed.S0_rep1_2.fq,\
02_trim/trimmed.S0_rep2_2.fq,\
02_trim/trimmed.S1_rep1_2.fq,\
02_trim/trimmed.S1_rep2_2.fq,\
02_trim/trimmed.S2_rep1_2.fq,\
02_trim/trimmed.S2_rep2_2.fq,\
02_trim/trimmed.S3_rep1_2.fq,\
02_trim/trimmed.S3_rep2_2.fq --output 03_normalization/
```

## __*De novo*__ transcriptome assembly

```bash
Trinity \
--CPU 32 --max_memory 60G \
--seqType fq --no_normalize_reads --monitoring \
--left 03_normalization/left.norm.fq \
--right 03_normalization/right.norm.fq \
--output 04_assembly_trinity/
```

## Contig clustering

```bash
conda create --name cluster cd-hit=4.8.1 -y
conda activate cluster

cd-hit-est -T 32 -M 0 -c 0.95 -G 0 -aS 0.99 \
-i 04_assembly_trinity/Trinity.fasta \
-o 05_clustering/cd-hit_Trinity.fasta

### Deactivate environments
conda deactivate
```

## Predict the protein-coding regions from the assembled contigs

The coding regions within the assembled contigs can be extracted by many tools that finds the Open Reading Frames (ORF) within a sequence. In this work, [Transdecoder](https://github.com/TransDecoder/TransDecoder) was used to generate a set of protein-coding sequences. 

Run Transdecoder in the environment named 

```bash
# create and activate conda environment
conda create --name translate transdecoder=5.5.0 -y
conda activate translate

# create a symbolic link of clustered transcript assembly in the current directory
cd 06_translate
ln -s ~/fmtg_rnaseq/05_clustering/cd-hit_Trinity.fasta ./

TransDecoder.LongOrfs -m 66 -t cd-hit_Trinity.fasta
TransDecoder.Predict -t cd-hit_Trinity.fasta --single_best_only
```




