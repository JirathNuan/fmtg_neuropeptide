Use transcripts assembled on 3Dec2019

## Translate to protein 

```sh
conda create -n transdecoder transdecoder=5.5.0
conda activate transdecoder

# เนื่องจากอยากให้โปรแกรม translate sequence ทุกเส้น, assembly มาได้ contig ยาวไม่ต่ำกว่า 200 bp ก็เลยเปลี่ยน minlength จาก 100 เป็น (200/3)=66 aa
TransDecoder.LongOrfs -m 66 -t ~/paper-2/3dec/04_assembly_trinity/Trinity.fasta
TransDecoder.Predict -t ~/paper-2/3dec/04_assembly_trinity/Trinity.fasta --single_best_only 


```

## BLAST

```sh
# Create blast environment
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
```
jiratchaya@poweredge-r440:~/paper-2/07_signalp$ ./signalp -help
Usage of ./signalp:
  -batch int
        Number of sequences that the tool will run simultaneously. Decrease or increase size depending on your system memory. (default 10000)
  -fasta string
        Input file in fasta format.
  -format string
        Output format. 'long' for generating the predictions with plots, 'short' for the predictions without plots. (default "short")
  -gff3
        Make gff3 file of processed sequences.
  -mature
        Make fasta file with mature sequence.
  -org string
        Organism. Archaea: 'arch', Gram-positive: 'gram+', Gram-negative: 'gram-' or Eukarya: 'euk' (default "euk")
  -plot string
        Plots output format. When long output selected, choose between 'png', 'eps' or 'none' to get just a tabular file. (default "png")
  -prefix string
        Output files prefix. (default "Input file prefix")
  -stdout
        Write the prediction summary to the STDOUT.
  -tmp string
        Specify temporary file directory. (default "System default tmpdir")
  -verbose
        Verbose output. Specify '-verbose=false' to avoid printing. (default true)
  -version
        Prints version.
```
