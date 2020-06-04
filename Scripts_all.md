Use transcripts assembled on 3Dec2019

## Quality control

```bash
# activate conda environment
conda activate qc

# Quality check (raw reads)
fastqc \
-t 32 \
-o 01_quality_check/before_trim/ \
raw_data/*.fq

# Quality control
for (( i = 1; i <= 4; i++ )); do
for (( j = 1; j <= 2; j++ )); do
cutadapt \
--max-n 0.00 --discard-trimmed \
--cores 32 \
-a AGATCGGAAGAGCACACGTCTGAACTCCAGTCA -A AGATCGGAAGAGCGTCGTGTAGGGAAAGAGTGT \
-o '02_trim/trimmed.S'$i'_rep'$j'_1.fq' \
-p '02_trim/trimmed.S'$i'_rep'$j'_2.fq' \
'raw_data/S'$i'_rep'$j'_1.fq' \
'raw_data/S'$i'_rep'$j'_2.fq' \
> '02_trim/S'$i'_rep'$j'.log'
done
done

# Quality check (clean reads)
fastqc \
-t 32 \
-o 01_quality_check/after_trim/ \
02_trim/*.fq

# Summarize QC report
multiqc \
--outdir 01_quality_check .

# Deactivate conda environments
conda deactivate
```

## *De novo* transcriptome assembly

```bash
# activate conda environment
conda activate assembly

# *in silico* read normalization before assembly (duration: 20191201 23:51 --> 20191202 04:08)
insilico_read_normalization.pl \
--seqType fq --max_cov 30 --pairs_together \
--CPU 32 --JM 60G \
--left \
02_trim/trimmed.S1_rep1_1.fq,\
02_trim/trimmed.S1_rep2_1.fq,\
02_trim/trimmed.S2_rep1_1.fq,\
02_trim/trimmed.S2_rep2_1.fq,\
02_trim/trimmed.S3_rep1_1.fq,\
02_trim/trimmed.S3_rep2_1.fq,\
02_trim/trimmed.S4_rep1_1.fq,\
02_trim/trimmed.S4_rep2_1.fq \
--right \
02_trim/trimmed.S1_rep1_2.fq,\
02_trim/trimmed.S1_rep2_2.fq,\
02_trim/trimmed.S2_rep1_2.fq,\
02_trim/trimmed.S2_rep2_2.fq,\
02_trim/trimmed.S3_rep1_2.fq,\
02_trim/trimmed.S3_rep2_2.fq,\
02_trim/trimmed.S4_rep1_2.fq,\
02_trim/trimmed.S4_rep2_2.fq \
--output 03_normalization/ \
> 03_normalization/insilico_norm.log

# De novo transcriptome assembly (duration: 20191202 08:43 --> 20191203 02:27)
Trinity \
--seqType fq --no_normalize_reads --monitoring --no_version_check \
--CPU 32 --max_memory 60G \
--left 03_normalization/left.norm.fq \
--right 03_normalization/right.norm.fq \
--output 04_assembly_trinity/ \
> 04_assembly_trinity/trinity.log

# deactivate conda environment
conda deactivate


# [Optional] Examine server usage from trinity
conda create --name assemmbly2 trinity=2.8.5 r-base -y
conda activate assemmbly2

~/miniconda3/pkgs/trinity-2.8.5-h8b12597_5/opt/trinity-2.8.5/trinity-plugins/COLLECTL/examine_resource_usage_profiling.pl collectl/

# [Optional] Examine the assembly statistic
TrinityStats.pl 04_assembly_trinity/Trinity.fasta > assembly_stats.txt

# [Optional] Move files
mv collectl.* collectl/
mv assembly_stats.txt 04_assembly_trinity/

# Deactivate environments
conda deactivate
```

## Translate to protein 

```sh
# create and activate conda environment
conda create -n transdecoder transdecoder=5.5.0
conda activate transdecoder

# เนื่องจากอยากให้โปรแกรม translate sequence ทุกเส้น, assembly มาได้ contig ยาวไม่ต่ำกว่า 200 bp ก็เลยเปลี่ยน minlength จาก 100 เป็น (200/3)=66 aa
TransDecoder.LongOrfs -m 66 -t ~/paper-2/3dec/04_assembly_trinity/Trinity.fasta
TransDecoder.Predict -t ~/paper-2/3dec/04_assembly_trinity/Trinity.fasta --single_best_only 

```

## BLAST

```sh
# create and activate conda environment
conda create -n ncbi blast=2.9.0
conda activate ncbi

mkdir nr_db lva_db

```
### Retrieve Nr database
Database was retrieved on Feburary 7,2020. 

__Details:__
Database: All non-redundant GenBank CDS translations+PDB+SwissProt+PIR+PRF excluding environmental samples from WGS projects
        257,100,652 sequences; 92,592,102,759 total residues

Date: Jan 30, 2020  12:42 AM    Longest sequence: 74,488 residues

BLASTDB Version: 5  Contains 38 volumes

- Extract only from Arthropod taxonomy ID retrieved from NCBI taxonomy statistics 

```sh
blastdbcmd -db /mnt/WD_SSD/nr_v5_20200206/nr -dbtype prot -taxidlist taxonomy_result.txt -outfmt %f -target_only > nr_arthropod_20200207.faa
```
### Retrieve Pacific white shrimp genome annotation
```sh
cd lva_genome

wget ftp://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/003/789/085/GCF_003789085.1_ASM378908v1/GCF_003789085.1_ASM378908v1_genomic.*
wget ftp://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/003/789/085/GCF_003789085.1_ASM378908v1/GCF_003789085.1_ASM378908v1_protein*
wget ftp://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/003/789/085/GCF_003789085.1_ASM378908v1/GCF_003789085.1_ASM378908v1_rna*
wget ftp://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/003/789/085/GCF_003789085.1_ASM378908v1/GCF_003789085.1_ASM378908v1_cds*
```



### Make blast database
```sh
cd nr_db
makeblastdb -in nr_arthropod_20200207.faa -dbtype prot -title nr_arthropod_20200207 -out nr_arthropod_20200207
cd ..

cd lva_db
makeblastdb -in GCF_003789085.1_ASM378908v1_rna.fna -dbtype nucl -title lva_rna -out lva_rna
```

### BLASTN against lva genome
```sh
blastn -db ~/paper-2/06_blast/lva_db/lva_rna -query ~/paper-2/Trinity.fasta -out ~/paper-2/06_blast/result_blastn_Lva.outfmt6 -evalue 1e-5 -outfmt "6 qseqid sseqid bitscore evalue pident qcovhsp qstart qend sstart send" -max_target_seqs 1 -num_threads 30
 
```


### BLASTP against Nr arthropod (14-3-2020)
```sh
# BLASTP (PID: 16146)
blastp -db nr_db/nr_arthropod_20200207 -query ~/paper-2/Trinity.fasta.transdecoder.pep -out Trinity.fasta.transdecoder.outfmt6 -evalue 1e-5 -outfmt "6 qseqid sseqid bitscore evalue pident qcovhsp qstart qend sstart send" -max_target_seqs 1 -num_threads 30 & disown

blastp -db nr_db/nr_arthropod_20200207 -query ~/paper-2/not-blast-yet-query.fasta -out not-blast-yet-query.fasta.outfmt6 -evalue 1e-5 -outfmt "6 qseqid sseqid bitscore evalue pident qcovhsp qstart qend sstart send" -max_target_seqs 1 -num_threads 30 & disown

```



## Identification of signal peptide by SignalP version 5.0b Linux x86_64

```sh
./signalp -batch 10000 -fasta ~/paper-2/Trinity.fasta.transdecoder.pep -format short -gff3 -mature -org euk -stdout
```

## InterProScan

First, create environment for interproscan and run interproscan through this environment
Put the shortcut of executable program and input in the same working directory

**Remark: Beware of asterisk `*` that indicates stop codon from translating DNA sequences to protein sequences**


```sh
# on server .242
./interproscan.sh -i Trinity.fasta.transdecoder.fasta -cpu 30

# on server .49
./interproscan.sh -i Trinity.fasta.transdecoder.fasta -cpu 22
```

## Deeploc

input: Trinity.fasta.transdecoder.fasta


	Deeploc requirements:
	git+https://github.com/Lasagne/Lasagne.git#egg=Lasagne-0.2.dev1
	numpy==1.14.0
	scipy==1.0.0
	six==1.11.0
	Theano==1.0.1

> **Remarks:**
	I've used to install deeploc's requirements on environment of python 3.7, and it was failed. Because of numpy=1.14.0 is only compatible for version 2.7.x, 3.4.x, 3.5.x, 3.6.x, >=3.5, <3.6.0a0, >=2.7 ,<2.8.0a0, >=3.6, and <3.7.0a0. So, Downgrading python version to 3.6 is worked for me.


```sh
# Create a specific environment for deeploc
conda create -n deeploc python=3.6 -y
conda activate deeploc

# go to deeploc source code directory and install requirements
pip install -r requirements.txt
python setup.py install

# Test DeepLoc by running
deeploc -f test.fasta


# Run Deeploc on working directory
deeploc -f test.fasta
```

Run Deeploc with my data

Current working directory: ~/10_deeploc
```sh
deeploc -f query_deeploc.fasta -o output_deeploc.txt
```
