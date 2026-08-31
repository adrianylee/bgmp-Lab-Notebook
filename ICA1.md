Lab Notebook for ICA1: Using Genomic Databases
## Summary
- Working with NCBI and Ensembl Databases to analyze genome annotation files from *Electrophorus electricus, Phycodurus eques, Lepisosteus oculatus*, and *Homo sapiens*.
- Downloading and using sra tools

Main Files (input/output)
- [main repo for ICA1](https://github.com/2026-BGMP/adrianylee-Bi623-ICA1/tree/main)
- [sra.sh](https://github.com/2026-BGMP/adrianylee-Bi623-ICA1/blob/main/sra.sh)

**Software Versions**  
sratoolkit.3.4.1-alma_linux64  
Electrophorus electricus: Genome assembly ASM4190279v1, GCF_041902795.1  
Lepisosteus oculatus: Genome assembly fLepOcu1.hap2, GCF_040954835.1  
Phycodurus eques: Genome assembly UOR_Pequ_1.1, GCF_024500275.1  
Homo sapiens: Genome assembly GRCh38.p14, GCF_000001405.40  

---
## 8/25/2026
### Working alone
### Part 1

General scn4aa analysis. Visited both NCBI and Ensembl to familiarize myself with the website UI and locate and understand the scn4aa gene with E. Electricus. Links used for primary analysis:
NCBI - [link](https://www.ncbi.nlm.nih.gov/gene/113588309)  
Ensembl - [link](https://www.ensembl.org/genome-browser/2fa1cc19-c011-47b7-bf79-741df608d0d6?focus=gene:ENSEEEG00000012825&location=15:16998123-17055646)  

For genome size (same sites, different locations):
NCBI - 833.4 Mb [link](https://www.ncbi.nlm.nih.gov/datasets/genome/GCF_041902795.1/)  
Ensembl - 833427914 [link](https://www.ebi.ac.uk/ena/browser/view/GCA_041902795.1), the GCA links directly to ENA. [original link on ensembl](https://www.ensembl.org/genome-selector/search?query=Electrophorus+electricus)  

### Part 2

Continued scn4aa analysis across further species, using bash to sort through the data.
Located the FTP page for each individual gene for the required species.
```
wget https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/041/902/795/GCF_041902795.1_ASM4190279v1/GCF_041902795.1_ASM4190279v1_genomic.gtf.gz
wget https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/024/500/275/GCF_024500275.1_UOR_Pequ_1.1/GCF_024500275.1_UOR_Pequ_1.1_genomic.gtf.gz
wget https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/040/954/835/GCF_040954835.1_fLepOcu1.hap2/GCF_040954835.1_fLepOcu1.hap2_genomic.gtf.gz 
wget https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/001/405/GCF_000001405.40_GRCh38.p14/GCF_000001405.40_GRCh38.p14_genomic.gtf.gz
```
These specific commands were used via "copy link" from the FTP download page.

Bash analysis used to sift through the data to examine where scn4aa genes are (and which ones are protein coding)

### Part 3: Analyzing FASTQ Files

Downloaded SRR11722023 and SRR1302052 using SRA_toolkit. This is the pipeline used based on the documentation: 
```
wget --output-document sratoolkit.tar.gz https://ftp-trace.ncbi.nlm.nih.gov/sra/sdk/current/sratoolkit.current-alma_linux64.tar.gz
tar -xzf sratoolkit.tar.gz
export PATH=$PWD/sratoolkit.3.4.1-alma_linux64/bin:$PATH

# SLURM script: 

prefetch SRR11722023 --max-size u
prefetch SRR1302052 --max-size u

fasterq-dump SRR11722023
fasterq-dump SRR1302052

gzip SRR11722023*.fastq
gzip SRR1302052*.fastq
```

Note that --max-size u had to be used since the sra files are ~40 gb.

This notebook was not completed on time (started and finished on 8/31) but accurately reflects the timeline and actual steps taken during this assignment. 

