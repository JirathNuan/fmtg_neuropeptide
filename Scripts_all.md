## Transcript assembly clustering
```sh
# install CD-HIT
conda create -n cdhit cd-hit -y
conda activate cdhit

# Use CD-HIT
cd-hit -i 3Dec_Trinity.fasta -o 01_clustering/3Dec_cdhit.fasta -c 1 -T 30 -M 0
```