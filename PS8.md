### 7/13/26

### Working with Imre, Diya, Rose

# Part 1: STAR
The first step is to log onto a compute node within Talapas. All work is done within this compute node on Talapas.
```
srun -A bgmp -p bgmp --time=2-00:00:00 --pty bash
```
After cloning the repository into the correct location ```/projects/bgmp/aylee/bioinfo/Bi621/PS```, I created the ```dre/``` directory which will store my assembly fasta and my gtf data.

Within PS8/dre I ran the following commands that I got from ensembl.org -> downloads -> ftp downloads looking for the primary assembly fasta and the 116 GTF for Danio_rerio (zebrafish):
```
wget https://ftp.ensembl.org/pub/release-116/fasta/danio_rerio/dna/Danio_rerio.GRCz11.dna.primary_assembly.fa.gz
wget https://ftp.ensembl.org/pub/release-116/gtf/danio_rerio/Danio_rerio.GRCz11.116.gtf.gz
```
Unzipped both files with ```gunzip``` as STAR --genomeFastaFiles flag only takes unzipped input.

I created another directory that included the tool, version, ensembl information, and species information. It is found in the base ```PS/``` directory: ```STAR_2.7.11b-Danio_rerio.GRCz11.dna-ens116```. It will include outputs from STAR

Then, I executed the following commands to check that I am both in pixi and have the required tools for the remainder of this problem set.
```
cd /projects/bgmp/aylee/bioinfo/Bi621/PS/adrianylee-Bi621-PS8/
pixi init
pixi add star
pixi add samtools
pixi shell
STAR --version                        ### should be 2.7.11b
samtools --version                    ### should be 1.23.1
```
Software version info:
STAR: 2.7.11b
samtools: 1.23.1

I created a SLURM script ```starDB.sh``` which I will use to run genomeGenerate via STAR
```
#!/bin/bash
#SBATCH --account=bgmp                    # REQUIRED: which account to use
#SBATCH --partition=bgmp                  # REQUIRED: which partition to use
#SBATCH --cpus-per-task=8                 # optional: number of cpus, default is 1
#SBATCH --job-name=star                   # optional: job name


/usr/bin/time -v pixi run STAR --runThreadN 8 --runMode genomeGenerate \
--genomeDir STAR_2.7.11b-Danio_rerio.GRCz11.dna-ens116 \
--genomeFastaFiles dre/Danio_rerio.GRCz11.dna.primary_assembly.fa \
--sjdbGTFfile dre/Danio_rerio.GRCz11.116.gtf
```
This SLURM bash script copies the format of previous examples. I made it executeable ```chmod 755 starDB.sh``` and ran with ```sbatch starDB.sh``` within Talapas

On the first run I ran out of memory, so I removed that line from the SLURM script. The example above already doesn't include the --mem line, so it defaults to 4 GB per core so 32 GB.

Jason came by and helped me resolve the above issue and taught me that I can use a calculator inside the terminal via ```bc``` . ```scale=x``` when you launch where x is the number of decimals. This allowed for quick calculations to see how much data the job actually took (displated in kb, want GB)

SLURM.out stored: ```/projects/bgmp/aylee/bioinfo/Bi621/PS/adrianylee-Bi621-PS8/slurm-45207541.out```
Summary of statistics:
The process completed in 8 minutes and 41 seconds with a maxumum memory usage of ~27.61 GB (used ```bc``` to calculate within the terminal). The job utilized 371% CPU (which is around 3.7 cores).

The STAR outputs are in the ```STAR_2.7.11b-Danio_rerio.GRCz11.dna-ens116``` which is the genome directory which will be used for the STAR alignment step.

I then created a secondary SLURM script that uses STAR to align reads: ```alignReads.sh```
This is what the script looks like (the format is very similar to above). This is the second part of the STAR run. They are ran separately because this is my first time doing it.

```
#!/bin/bash
#SBATCH --account=bgmp                    # REQUIRED: which account to use
#SBATCH --partition=bgmp                  # REQUIRED: which partition to use
#SBATCH --cpus-per-task=8                 # optional: number of cpus, default is 1
#SBATCH --job-name=AligningReads          # optional: job name

R1=/projects/bgmp/shared/Bi621/dre_WT_ovar12_R1.qtrim.fq.gz
R2=/projects/bgmp/shared/Bi621/dre_WT_ovar12_R2.qtrim.fq.gz

/usr/bin/time -v pixi run STAR --runThreadN 8 --runMode alignReads \
 --outFilterMultimapNmax 3 \
 --outSAMunmapped Within KeepPairs \
 --alignIntronMax 1000000 --alignMatesGapMax 1000000 \
 --readFilesCommand zcat \
 --readFilesIn $R1 $R2 \
 --genomeDir STAR_2.7.11b-Danio_rerio.GRCz11.dna-ens116 \
 --outFileNamePrefix zebrafish
```
Ran with sbatch

output SLURM:
```slurm-45207818.out```
Summary of statistics:
The process completed in 674.81 seconds with a maxumum memory usage of ~15.12 GB (used ```bc``` to calculate within the terminal). The job utilized 682% CPU (which is around 6.8 cores).
output file: ```zebrafishAligned.out.sam```

# Part 2: Samtools

I then ran samtools on the completed alignment (from STAR) in order to get a bam file:
```/usr/bin/time -v pixi run samtools view -b -o zebrafish.bam zebrafishAligned.out.sam```
slurm-45208357.out
Summary of statistics:
The run took 74.13 seconds and used 99% of CPU. It used .05 GB RAM. 8 threads were used in initial run.

I then ran the following commands separately, in the terminal, in an interactive shell on a Talapas compute node. These commands were used to sort, extract, and create BAM file:
```
/usr/bin/time -v pixi run samtools sort zebrafish.bam -o zebrafish.sorted.bam
/usr/bin/time -v pixi run samtools index zebrafish.sorted.bam
/usr/bin/time -v pixi run samtools view zebrafish.sorted.bam 1 -o chr1_sorted_zebrafish.sam
```
This sorts the bam file, indexes it so samtools can grab data very quickly with view. Sorted to only grab chromosome 1. I reran these 3 commands together in a SLURM script after I got it confirmed to work. 

There are 3 SLURM outputs since usr/bin/time -v was run for all 3 commands. slurm out file: ```slurm-45259478.out```. Summary of statistics: Initial samtools sort took 98.15 seconds, using 97% of the CPU, and used 0.873 GB RAM. Indexing took 4.74 seconds, used 95% of CPU, and 0.057 GB RAM. The final view and output of only chromosome 1 alignments took 0.28 seconds, used 66% of the CPU, and used 0.059 GB RAM.

Number of alignments on chr1: 
```grep -c "^K00337" chr1_sorted_zebrafish.sam```  
```wc -l chr1_sorted_zebrafish.sam ``` 
yields the same. --> 921737 total alignments

wrote a python program ```mapped.py``` that iterates through all of the original SAM (```zebrafishAligned.out.sam```) lines. It checks to make sure the current line is header line and grabs the bitwise flag (the second value when the line is split). Using this binary flag, the program checks the following two bitwise flags: (0x4 or 4) and (0x100 or 256) to check whether an alignment is mapped and if it is a primary alignment, respectively. Since I do not want to count secondary alignments, if the alignment is a primary one (this must be done since pairwise alginments have the exact same name), and increments a mapped or unmapped counter (depending on what the alignment is). These statistics are summarized at the end. 

Number of MAPPED reads: 21851108   
Number of UNMAPPED reads: 1645850

All files uploaded, answers.md updated, everything pushed to github. All analyses and work completed in one day (7/13/26), this lab notebook was finalized on 7/14/26.

