## Translate to protein 

```sh
conda create -n transdecoder transdecoder=5.5.0
conda activate transdecoder

# เนื่องจากอยากให้โปรแกรม translate sequence ทุกเส้น, assembly มาได้ contig ยาวไม่ต่ำกว่า 200 bp ก็เลยเปลี่ยน minlength จาก 100 เป็น (200/3)=66 aa
TransDecoder.LongOrfs -m 66 -t ~/paper-2/3dec/04_assembly_trinity/Trinity.fasta


```

## Transcript assembly clustering
```sh
# install CD-HIT
conda create -n cdhit cd-hit -y
conda activate cdhit

# Use CD-HIT
cd-hit -i 3Dec_Trinity.fasta -o 01_clustering/3Dec_cdhit.fasta -c 1 -T 30 -M 0

```

## BLAST

```sh
# Create blast environment
conda create -n ncbi blast=2.9.0
conda activate ncbi

cd 
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

### Make blast database
```sh
makeblastdb -in nr_arthropod_20200207.faa -dbtype prot -title nr_arthropod_20200207 -out nr_arthropod_20200207 -hash_index
```