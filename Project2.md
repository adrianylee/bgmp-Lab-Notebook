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
- [mapped slurm]()
- []()
- []()
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
STAR run alignment for SRR25630396 run statistics: 




