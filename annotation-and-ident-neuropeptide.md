# Transcriptome annotation and identification of neuropeptide precursors


> Remarks: all following commands are performed on Ubuntu 18.04 LTS 64-bit operating system Dell Server 24 CPU threads and 48GB RAM

### Prerequisites

According to the previously work `assembly-and-post-assembly-process.md`. The base working directory was set to `~/fmtg_rnaseq`, this part will continue to the downstream analysis of transcriptome assembly i.e. transcriptome annotation and identification of the neuropeptide precursors. 

First, create the working directories for each analysis. All directories should be located in `~/fmtg_rnaseq` directory.
```
mkdir 07_blastx 08_ghostkoala 09_signalp 10_deeploc
```


## BLASTX against customized Nr database

```bash
conda create --name blast  -y
conda activate blast

cd ~/fmtg_rnaseq
ln -s 08_Transdecoder/cd-hit_Trinity.fasta.transdecoder.cds 11_BLASTX/
cd 11_BLASTX


# BLASTX
blastx -db nr_arthropod -query cd-hit_Trinity_transdecoder_cds.fasta -out cd-hit.Trinity_transdecoder_cds.outfmt6 -evalue 1e-5 -outfmt "6 std qcovhsp stitle" -max_target_seqs 1 -num_threads 24 & disown


```


## Identification of putative neuropeptide precursors by SignalP and Deeploc

### SignalP

SignalP version 5.0 was used to predict the presence of the signal peptide at the _N_-terminal of a protein sequence. The software can be downloaded from [SignalP 5.0 web server](http://www.cbs.dtu.dk/services/SignalP/) for academic users. 

```sh
./signalp -batch 10000 -fasta ~/fmtg_rnaseq/08_Transdecoder/cd-hit_Trinity.fasta.transdecoder.pep -format short -gff3 -mature -org euk -stdout
```

### Deeploc

In this study, DeepLoc version 1.0 was used to identify subcellular localization of secretory proteins that resulted from the SignalP prediction. This prediction aimed to ensured that the secretory proteins are really secrete outside the cell. This software can be downloaded from [DeepLoc 1.0 web server](http://www.cbs.dtu.dk/services/DeepLoc-1.0/index.php) for academic users.

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
 
