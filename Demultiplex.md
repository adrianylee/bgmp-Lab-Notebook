# Demultiplexing
## Summary
-

Main Inputs/Outputs
- []()
- []()
- []()

Software Versions
- fff

---
### 7/22/26
### Assignment the First: Part 1 Quality Score Distribution per-nucleotide

I used a number of commands to examine the data (and how it was formatted).

```
zcat /projects/bgmp/shared/2017_sequencing/1294_S1_L008_R3_001.fastq.gz | head -2
zcat 1294_S1_L008_R4_001.fastq.gz | head -6
zcat 1294_S1_L008_R1_001.fastq.gz | head -6
zcat 1294_S1_L008_R3_001.fastq.gz | head -6
zcat 1294_S1_L008_R2_001.fastq.gz | head -6
zcat 1294_S1_L008_R1_001.fastq.gz | head -2 | tail -1 | wc
zcat 1294_S1_L008_R2_001.fastq.gz | head -2 | tail -1 | wc
1452986940

```
This initial data exploration showed which files were fastq sequence files (R1 and R4) 
and which ones were fastq index files (R2 and R3). Also checked to verify it was indeed fastq 
format, how long each sequence was, and checked to see which files were linked to each other
(R1/R2 and R3/R4). This information was helpful in filling out the following table:

| File name | label | Read length | Phred encoding |
|---|---|---|---|
| 1294_S1_L008_R1_001.fastq.gz | read1 | 101 | phred33 |
| 1294_S1_L008_R2_001.fastq.gz | index1 | 8 | 33 |
| 1294_S1_L008_R3_001.fastq.gz | index2 | 8 | 33 |
| 1294_S1_L008_R4_001.fastq.gz | read2 | 101 | 33 |

I then created a new script to calculate the per-base nucleotide distribution: ```/projects/bgmp/aylee/bioinfo/Bi622/PS/Demultiplex/Assignment-the-first/qualityScoreDistribution.py```


