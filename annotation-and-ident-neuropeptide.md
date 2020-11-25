RNA-Seq data analysis of Fme Fenneropenaeus merguiensis thoracic ganglia
================================

> Remarks: all following commands are performed on Ubuntu 18.04 LTS 64-bit operating system Dell Server 24 CPU threads and 48GB RAM

## Prerequisites

Create root working directory, and analysis subdirectories
```bash
mkdir ~/fmtg_rnaseq

cd ~/fmtg_rnaseq
mkdir 05_CDHIT 06_Evaluate 07_DEG 08_BLASTN 09_BLASTX 10_Translate 11_BLASTP 12_TBLASTN 13_GhostKoala 14_SignalP 15_DeepLoc
```


## Translate contig assembly to protein sequence

```bash
cd 10_Translate
# [optional] create a symbolic link of clustered transcript assembly in the current directory
ln -s ~/fmtg_rnaseq/05_CDHIT/cd-hit_Trinity.fasta ./

# create and activate conda environment
conda create -n transdecoder transdecoder=5.5.0
conda activate transdecoder

# เนื่องจากอยากให้โปรแกรม translate sequence ทุกเส้น, assembly มาได้ contig ยาวไม่ต่ำกว่า 200 bp ก็เลยเปลี่ยน minlength จาก 100 เป็น (200/3)=66 aa
TransDecoder.LongOrfs -m 66 -t cd-hit_Trinity.fasta
TransDecoder.Predict -t cd-hit_Trinity.fasta --single_best_only

```

## BLASTN against Lva genome

### Install BLAST through Bioconda channel
```sh
# create and activate conda environment
conda create -n ncbi blast=2.9.0
conda activate ncbi
```

### Retrieve Pacific white shrimp genome annotation
```sh
cd 08_BLASTN/

mkdir lva_genome && cd lva_genome

wget ftp://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/003/789/085/GCF_003789085.1_ASM378908v1/GCF_003789085.1_ASM378908v1_genomic.*
wget ftp://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/003/789/085/GCF_003789085.1_ASM378908v1/GCF_003789085.1_ASM378908v1_protein*
wget ftp://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/003/789/085/GCF_003789085.1_ASM378908v1/GCF_003789085.1_ASM378908v1_rna*
wget ftp://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/003/789/085/GCF_003789085.1_ASM378908v1/GCF_003789085.1_ASM378908v1_cds*

gunzip *
cd ..
```

### Nucleotide BLAST against Lva RNA
```sh
makeblastdb -in lva_genome/GCF_003789085.1_ASM378908v1_rna.fna -dbtype nucl -title lva_rna -out lva_rna

# Create a symbolic link of a query sequence (CDS output from Transdecoder)
ln -s ~/fmtg_rnaseq/10_Translate/cd-hit_Trinity.fasta.transdecoder.cds ./

# Nucleotide BLAST (BLASTN)
blastn -db lva_rna -query cd-hit_Trinity.fasta.transdecoder.cds -out cd-hit.Trinity_transdecoder_cds.outfmt6 -evalue 1e-5 -outfmt "6 std qcovhsp sstrand stitle" -max_target_seqs 1 -num_threads 24

```

	> qcovs vs qcovhsp vs qcovus
		Your assumption seems to be correct (overlap information is taken in to account). Digging in to the BLAST source code, I stumbled on this part where many parameters are defined. Check line no: 110 in the following page: http://www.ncbi.nlm.nih.gov/IEB/ToolBox/CPP_DOC/lxr/source/include/objects/seqalign/Seq_align.hpp#L54 From what I understand, 'pct_coverage' is the 'qcovs'. It is the percent of no. of bases in the query sequence aligned with the subject sequence (match or mismatch). The bases can be in one HSP or several HSPs (overlap) but they are counted only once. Gaps (in the subject sequence) are treated as mismatches.
		(Retrieved from https://www.biostars.org/p/121972/#122201)

		Whereas, qcovus (Query Coverage Per Unique Subject) is a measure of Query Coverage that counts a position in a subject sequence for this measure only once. The second time the position is aligned to the query is not counted towards this measure. When not provided, the default value is: 'qseqid sseqid pident length mismatch gapopen qstart qend sstart send evalue bitscore', which is equivalent to the keyword 'std'
		(Retrieved from: https://www.ncbi.nlm.nih.gov/books/NBK279684)

### Protein BLAST against Lva Protein

```bash
conda activate blast

cd ~/fmtg_rnaseq/10_BLASTP
ln -s ~/fmtg_rnaseq/08_Transdecoder/cd-hit_Trinity.fasta.transdecoder.pep
ln -s ~/fmtg_rnaseq/09_BLASTN/lva_genome/GCF_003789085.1_ASM378908v1_protein.faa

makeblastdb -in GCF_003789085.1_ASM378908v1_protein.faa -dbtype prot -title lva_prot -out lva_prot

blastp -db lva_prot -query cd-hit_Trinity.fasta.transdecoder.pep -out cd-hit.Trinity_transdecoder_prot.outfmt6 -evalue 1e-5 -outfmt "6 std qcovhsp frames stitle" -max_target_seqs 1 -num_threads 24 & disown

```
### BLASTX against Arthropod Nr database

```bash
conda activate blast

cd ~/fmtg_rnaseq
ln -s 08_Transdecoder/cd-hit_Trinity.fasta.transdecoder.cds 11_BLASTX/
cd 11_BLASTX


# BLASTX
blastx -db nr_arthropod -query cd-hit_Trinity_transdecoder_cds.fasta -out cd-hit.Trinity_transdecoder_cds.outfmt6 -evalue 1e-5 -outfmt "6 std qcovhsp stitle" -max_target_seqs 1 -num_threads 24 & disown


```


## Identification of putative neuropeptide precursors by SignalP and Deeploc

### SignalP
```sh
./signalp -batch 10000 -fasta ~/fmtg_rnaseq/08_Transdecoder/cd-hit_Trinity.fasta.transdecoder.pep -format short -gff3 -mature -org euk -stdout
```

### Deeploc

input: cd-hit_Trinity.fasta.transdecoder.pep

	Deeploc requirements:
	git+https://github.com/Lasagne/Lasagne.git#egg=Lasagne-0.2.dev1
	numpy==1.14.0
	scipy==1.0.0
	six==1.11.0
	Theano==1.0.1

> **Remarks:**
	I've used to install deeploc's requirements on environment of python 3.7, and it was failed. Because of numpy=1.14.0 is only compatible for version 2.7.x, 3.4.x, 3.5.x, 3.6.x, >=3.5, <3.6.0a0, >=2.7 ,<2.8.0a0, >=3.6, and <3.7.0a0. Downgrading python version to 3.6 is worked for me.


```sh
# Create a specific environment for deeploc
conda create -n deeploc python=3.6 -y
conda activate deeploc

# go to deeploc source code directory and install requirements
pip install -r requirements.txt
python setup.py install

# Test DeepLoc by running
deeploc -f test.fasta


# Run Deeploc on working directory [Optional]
deeploc -f test.fasta
```

Run Deeploc with my data
```sh 
deeploc -f query_deeploc.fasta -o output_deeploc.txt
```
 