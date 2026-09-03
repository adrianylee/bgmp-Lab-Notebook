Lab Notebook for Project2
## Summary
- 

Main Files (input/output)

Part 2  
- [python plotting script for length of distributions](https://github.com/adrianylee/Project-2-Electric-organ-RNA-seq-analysis/blob/master/Project2_Part2/plot.py)
- [SRR25630306 bar graph distribution](https://github.com/adrianylee/Project-2-Electric-organ-RNA-seq-analysis/blob/master/Project2_Part2/SRR25630306.png)
- [SRR25630396 bar graph distribution](https://github.com/adrianylee/Project-2-Electric-organ-RNA-seq-analysis/blob/master/Project2_Part2/SRR25630396.png)
- [fastqc script performed on post-trimmed files](https://github.com/adrianylee/Project-2-Electric-organ-RNA-seq-analysis/blob/master/Project2_Part2/fastqc.sh)
- [cutadapt and trimmomatic run](https://github.com/adrianylee/Project-2-Electric-organ-RNA-seq-analysis/blob/master/Project2_Part2/cutandtrim.sh)
- [answer to bonus](https://github.com/adrianylee/Project-2-Electric-organ-RNA-seq-analysis/blob/master/Project2_Part2/Project2_Part2_answers.txt)


Part 3  
- [star database SLURM](https://github.com/adrianylee/Project-2-Electric-organ-RNA-seq-analysis/blob/master/Project2_Part3/stardb.sh)
- [star align SLURM](https://github.com/adrianylee/Project-2-Electric-organ-RNA-seq-analysis/blob/master/Project2_Part3/staralign.sh)
- [mapped python](https://github.com/adrianylee/Project-2-Electric-organ-RNA-seq-analysis/blob/master/Project2_Part3/mapped.py)
- [mapped slurm](https://github.com/adrianylee/Project-2-Electric-organ-RNA-seq-analysis/blob/master/Project2_Part3/mapped.sh)
- [htseq slurm script](https://github.com/adrianylee/Project-2-Electric-organ-RNA-seq-analysis/blob/master/Project2_Part3/htseq.sh)
- [answers](https://github.com/adrianylee/Project-2-Electric-organ-RNA-seq-analysis/blob/master/Project2_Part3/part3answers.md)
- 

**Software Versions**  
PART1  
fastqc = ">=0.12.1,<0.13"  
cutadapt = ">=5.2,<6"  
trimmomatic = ">=0.41,<0.42"  

PART2  
trimmomatic = ">=0.41,<0.42"  
cutadapt = ">=5.2,<6"  
matplotlib = ">=3.11.1,<4"  
fastqc = ">=0.12.1,<0.13" 

PART3  
✔ Added star >=2.7.11b,<3  
✔ Added samtools >=1.23.1,<2  
✔ Added numpy >=2.5.2,<3  
✔ Added matplotlib >=3.11.1,<4  
✔ Added htseq >=2.1.2,<3  
✔ Added gffread >=0.12.9,<0.13

### 9/2/26
#### Working alone

## Part 2 - Plotting the distribution of lengths from cutadapt + trimmomatic pipeline

Initially tried to use a histogram, storing all lengths in a list, as I used gzip to loop over the entire file.

HISTOGRAM DOES NOT WORK. It worked initially but kept getting "Killed" when I tried to further modify the plots. Based on online forums, this is because matplotlib ran out of memory. 
Do not use histogram for these type of calculations (hundreds of thousands of data points, logged, etc) as there are too many calculations. In the future must preprocess the data for the histogram (pre-bin) and that would more likely work.

Instead rewrote the code to store all length values in a dictionary. {length: count}. Ordered this dictionary, and put the keys as x values, the values as y values. Used a bar graph. Used alpha = 0.5 to make bars 50% transparent so I could plot them over each other. In the future can use width to displace the bars so they are sitting side-by-side. However, alpha worked very well with matplotlib bar. Logged the y axis so the data is a bit easier to interpret. 

Used the skeleton SLURM file from part 1, and reran fastqc on the trimmed paired end files. HTML files attached in Github.

## Part 3 - downloading packages
Created a new folder as outlined by the instructions. Did the following to set up my environment:
```
pixi init
pixi add star samtools numpy matplotlib htseq
```
Got the following:
```
✔ Added star >=2.7.11b,<3  
✔ Added samtools >=1.23.1,<2  
✔ Added numpy >=2.5.2,<3  
✔ Added matplotlib >=3.11.1,<4  
✔ Added htseq >=2.1.2,<3
```

Will be working within this directory within ```pixi shell``` and ```srun -A bgmp -p bgmp --time=2:00:00 --pty bash```. This has been standard practice for multiple assignments but wanted to explicitly lay it out here. 

Attempted to download via the following two commands:
```
wget https://datadryad.org/downloads/file_stream/2058657
--2026-09-02 20:28:08--  https://datadryad.org/downloads/file_stream/2058657
Resolving datadryad.org (datadryad.org)... 32.186.204.172, 32.187.32.25, 54.213.72.218, ...
Connecting to datadryad.org (datadryad.org)|32.186.204.172|:443... connected.
HTTP request sent, awaiting response... 403 Forbidden
2026-09-02 20:28:09 ERROR 403: Forbidden.

[aylee@login2 Project2_Part3]$ wget https://datadryad.org/downloads/file_stream/2058656
--2026-09-02 20:28:55--  https://datadryad.org/downloads/file_stream/2058656
Resolving datadryad.org (datadryad.org)... 54.213.72.218, 100.23.147.45, 32.187.32.25, ...
Connecting to datadryad.org (datadryad.org)|54.213.72.218|:443... connected.
HTTP request sent, awaiting response... 403 Forbidden
2026-09-02 20:28:55 ERROR 403: Forbidden.
```
Luckily the data is already in Talapas here:   
`/projects/bgmp/shared/Bi623/Project2/campylomormyrus.fasta`  
`/projects/bgmp/shared/Bi623/Project2/campylomormyrus.gff`  

Added gffread to pixi  

Since the data is a gff file, it must be converted to a gtf file for STAR to run properly.
```pixi run gffread campylomormyrus.gff -T -o campylomormyrus.gtf``` in the terminal  

Then used the format of PS8 to create a STAR database and align.  

On initial database creation got a warning: ```!!!!! WARNING: --genomeSAindexNbases 14 is too large for the genome size=862592683, which may cause seg-fault at the mapping step. Re-run genome generation with recommended --genomeSAindexNbases 13``` which can be found in ```slurm-47036765.out```  

So reran using the suggestion, new SLURM output found here: ```slurm-47036809.out```. Run statistics: 4 minutes 24 seconds, 433% CPU usage, 23.16851 GB RAM usage. Calculated using ```bc``` and ```scale=5```  

Again, based on PS8 created an alignReads SLURM script for STAR. Two runs, one for each SRA that I'm responsible for. Check above for link to the script for more specific details. Output SLURM: ```slurm-47036847.out```  

STAR run alignment for SRR25630306 run statistics: 8 minutes 26 seconds. 750% CPU, 9.7776 GB RAM  
STAR run alignment for SRR25630396 run statistics: 20 minutes 7 seconds, 742% CPU, 9.7797 GB RAM

Slightly updated the mapping script from PS8 to include a printout of current file. Ran via SLURM script. SLURM output: ```slurm-47037011.out```.  

SRR25630306  
Mapped: 68406912  
Unmapped: 5385616  
Run stats: 55 seconds, 91% CPU usage, 0.0601 GB RAM

SRR25630396  
Mapped: 209805206  
Unamppaed: 9957574  
Run stats: 2 minutes 19 seconds, 99% CPU usage, 0.0601 GB RAM

Next, read through the ht-seq documentation and wrote another SLURM script (linked above). Used standard run with -i. Two runs for each. First stranded, second reverse for each alignment file. Initially ran with gff file and -i gene_id. Realized that this wasn't going to work since the gff file doesn't have those identifiers. Documentation states that I can just use a valid gtf file, using that instead. Reran again since I had to specify a standard out. 

SLURM output: ```slurm-47037187.out```  

Run statistics: 
SRR25630306 --> both around ~15 minutes, 99% CPU usage, 0.14607 GB RAM
SRR25630396 --> both around ~45 minutes, 99% CPU usage, 0.14813 GB RAM

### 9/3/26
#### Working 
## Part 3 (cont)

Found out that my gtf file using gffread has significantly fewer lines than an AGAT-converted gtf file. Reconverted a gft file using AGAT and reran STAR to check for differences between the runs. Also did another htseq-count run using ```-i``` Reran entire pipeline with AGAT to see if the original differences found in between the gffread converted file gtf and and gff would impact downstream analyses. **it does not**. Using gffread is a perfectly fine (and faster) method to convert a file to gtf. Mapped/Unmapped are exactly the same. Comparing outcomes.

Did a full run with AGAT (since there were massive differences between the gtf and gff files converted by gffread). The only statistics that matter for this pipeline are exon and gene counts. As long as they match, gffread is faster and works better with downstream analyses. During the AGAT test, all values and genes are exactly the same without any issues. If AGAT is used, ```-i Parent``` needs to be used to track by gene rather than exon. If gffread is used, the default flag ```-i gene_id``` works perfectly fine. 

As outlined by the run statistics above, htseq will only run on a single core, despite using the ```-n``` flag. Use multiple scripts to run if you want it faster. 

All answers were calculated using a simple counts script taken from ICA4. These are the commands used to find the percentage (manually calculated)
```

grep -v '^__' SRR25630306_counts_stranded | awk '{sum += $2} END {print sum}'
awk '{sum += $2} END {print sum}' SRR25630306_counts_stranded

grep -v '^__' SRR25630306_counts_reverse | awk '{sum += $2} END {print sum}'
awk '{sum += $2} END {print sum}' SRR25630306_counts_reverse

grep -v '^__' SRR25630396_counts_stranded | awk '{sum += $2} END {print sum}'
awk '{sum += $2} END {print sum}' SRR25630396_counts_stranded

grep -v '^__' SRR25630396_counts_reverse | awk '{sum += $2} END {print sum}'
awk '{sum += $2} END {print sum}' SRR25630396_counts_reverse
```

Results:
## Stranded/Reverse
**SRR25630306 - stranded**
HTSeq Counts: 942195
Total: 36896264
Percentage: 2.553%

**SRR25630306 - reverse**
HTSeq Counts: 16185491
Total: 36896264
Percentage: 43.867%

**SRR25630396 - stranded**
HTSeq Counts: 3378953
Total: 109881390
Percentage: 3.075%

**SRR25630396 - reverse**
HTSeq Counts: 60747263
Total: 109881390
Percentage: 55.284%

Further data can be found in the attached answers.md file (above).

This lab notebook was updated 9/3/26.




