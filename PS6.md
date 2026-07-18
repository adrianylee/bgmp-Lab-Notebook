# Lab Notebook for PS6 --> SPAdes and Raven
## Summary
- Commands used to run SPAdes and Raven and calculate contig distribution (and other statistics) including runtime statistics for future usage.
- Locations and names of input and output files
- Tracked when things got done and who I worked on them with

Main Files (input/output)
- [contigDistribution.py](https://github.com/2026-BGMP/adrianylee-Bi621-PS6/blob/main/contigDistribution.py) - takes in an input fasta file and calculates statistics, outputting them to a table. Works with both raven and spades outputs, but must specify. Also generates a kmer distribution tsv and graph for each input file. Statistics also output as standard out (as well as within the results table).
- [calculations.py](https://github.com/2026-BGMP/adrianylee-Bi621-PS6/blob/main/calculations.py) - takes in a contigs.fasta file and outputs base coverage and kmer coverage statistics. Outputs as standard out. 
- Scripts used to run python scripts or bioinformatic tools: [spadesSummary.sh](https://github.com/2026-BGMP/adrianylee-Bi621-PS6/blob/main/spadesSummary.sh), [runRaven.sh](https://github.com/2026-BGMP/adrianylee-Bi621-PS6/blob/main/runRaven.sh), [populate_table.sh](https://github.com/2026-BGMP/adrianylee-Bi621-PS6/blob/main/populate_table.sh), [runCheckM2.sh](https://github.com/2026-BGMP/adrianylee-Bi621-PS6/blob/main/runCheckM2.sh)
- [answers.md](https://github.com/2026-BGMP/adrianylee-Bi621-PS6/blob/main/answers.md) - answers to questions

Software Versions
- spades version = 4.3.0
- raven-assembler = 1.8.3
- matplotlib = 3.11.0
- checkm2 = 1.1.0

---

## 7/9/2026
### Working with Zach
### Part 2 - testing a run on SPAdes

Cloned the repository into Talapas

Logged onto a node inside Talapas (DO NOT RUN ON LOGIN NODE): 
```
srun -A bgmp -p bgmp --time=2-00:00:00 --pty bash
```

This is the command used to test spades on a node within Talapas. This is within a bash script with everything we want to run. 
```
/usr/bin/time -v pixi run spades.py -k 21 -1 $R1 -2 $R2 -o spades.k21/ -t 8
```
 - gives output information (exit status/command used etc) via the -v information
 - other command flags are Spades specific

Command used to run the script on the node (I am logged in): format is sbatch [script]
```
sbatch spadesTest.sh 
```
-- spadesTest.sh formatted based on github

To check the run:
- cd spades.k21/ - in the directory where I ran sbatch
- squeue -u aylee - to check if somethings running

## 7/13/26
### Working alone
### Part 1 - Contig Length Distributions

Wrote ```contigDistribution.py``` which uses Python regex to search for the length of each contig and the kmer coverage and stores them in two lists (for SPAdes formatting). It does this by parsing a target .fasta file and looking for those matches via Python regex. Using these values for these contigs, calculates the number of contigs, the total length of the genome assembly, the maximum/minimum contig length, the mean contig length, the mean kmer coverage, the weighted mean coverage (takes into account the contig length to give a weighted mean), the N50, a distribution (.tsv) of 100-bin contig length sizes. It then uses the distribution to plot a graph that is logged on both axes and the x-axis is divided by 100. This allows the actual bars of the graph to be seen (otherwise with so much data, the bars will be 0 pixels wide, and thus invisible). Included the ```fasta_one_liner``` function from ```bioinfo.py``` but realized that it's not relevant to use in this assignment. It's just commented out. The exact descriptions of each function can be found within the file itself: [contigDistribution.py](https://github.com/2026-BGMP/adrianylee-Bi621-PS6/blob/main/contigDistribution.py). 

Also wrote out my test files (which includes expected output in a comment): ```Unit_test.fa``` and ```testoutput``` to use to test my ```contigDistribution.py```

## 7/16/26
### Working with Zach, Imre, Collin
### Part 2 - Running All SPAdes
All SPAdes scripts were run individually. There is a SLURM script for each individual run that is in each individual run folder in ```runs/```. This is the basic format for each SPAdes run. 
```
#!/bin/bash
#SBATCH --account=bgmp                    # REQUIRED: which account to use
#SBATCH --partition=bgmp                  # REQUIRED: which partition to use
#SBATCH --ntasks=12
#SBATCH --cpus-per-task=8                 # optional: number of cpus, default is 1
#SBATCH --mem=16GB                        # optional: amount of memory, default is 4GB per cpu
#SBATCH --job-name=testSpades             # optional: job name

DATA=/projects/bgmp/shared/Bi621/PS6/cjejuni/reads
R1=$DATA/illumina-nextseq2k/SRR26899120_1.fastq.gz
R2=$DATA/illumina-nextseq2k/SRR26899120_2.fastq.gz

/usr/bin/time -v pixi run spades.py -k 21 -1 $R1 -2 $R2 -o runs/spades.k21/ -t 8
```
The reason why all of the SPAdes runs are partitioned into indvidual scripts and run from their run folder is two-fold. The first reason is so outputs populate their run folder (there would be way too many outputs to organize otherwise). The second is so I can queue all the SPAdes runs and have them run essentially parallel. Since each run takes around 20 minutes, this allows them to all be done at the same time (rather than have them queued one after another. These are the summary statistics of my spadesk21 run: The run took 17 minutes and 1 second, using 683% CPU (ran on ~6.8 of the 8 cores/threads max allocation), and used 5.50 GB RAM. All other SPAdes (with only kmer filter) ran between 15 and 20 minutes, used +/- 50% CPU and used approximately the same amount of RAM. The SPAdes runs that had the coverage cutoff ran ~5-10 minutes longer on average (20-25 minutes) and had memory and CPU usages in the same range. Each of these SLURM output files are stored in their respective ```runs/``` folder. The directories for each of these runs was created manually (SPAdes doesn't do it for you) and the naming schematic was based off of instructions on GitHub PS6.

Each SPAdes output generates many files but the most important ones for this project are:
- ```contigs.fasta``` which is a list of contigs used for assembly (no gaps)
Actually pretty much all other output files are irrelevant for this PS but,
- ```scaffold.fasta``` - which is the primary assembly file (includes gaps)
- ```assembly_graph.fastg``` and ```assembly_graph_with_scaffolds.gfa``` which store the information for the de Brujin graph
- and ```spades.log``` which includes a detailed output of the spades run.

### Part 2 - Completing Calculations
Before SPAdes finishes running, I completed calculations to predict base and kmer coverage. I then checked my calculations to the actual SPAdes outputs for those same parameters. The script I wrote to calculate this which can be found in ```calculations.py```. In order to do this on the zipped files, I used gzip to open the files (as rt) so I didn't have to copy or unzip anything to do these calculations. The calculations were based off of the base coverage and kmer coverage equations. 

After SPAdes completed I compared calculations to the statistics. With the exception of the k127, other statistics were very similar to the SPAdes output (found in ```spades.log```). 

## 7/17/26
### Working with Imre
### Part 3 Raven

Using Raven on 5 different read sets. 2 from different pore chemistries (r.9.4.1 and r.10.4.1 nanopore) and then another 3 that used the R.10.4.1 pore chemistry which used different basecaller models. I ran raven using a script which can be found at ```runRaven.sh```
```
#!/bin/bash
#SBATCH --account=bgmp                    # REQUIRED: which account to use
#SBATCH --partition=bgmp                  # REQUIRED: which partition to use
#SBATCH --cpus-per-task=8                 # optional: number of cpus, default is 1
#SBATCH --mem=16GB                        # optional: amount of memory, default is 4GB per cpu
#SBATCH --job-name=runRaven               # optional: job name

DATA=/projects/bgmp/shared/Bi621/PS6/cjejuni/reads
R1=$DATA/ont-R10.4.1/SRR27638397.150x.fastq.gz
R2=$DATA/ont-R9.4.1/SRR10342325.150x.fastq.gz
R3=$DATA/ont-R10.4.1-basecallers/SRR27638397.sup-v4.3.0.150x.fastq.gz
R4=$DATA/ont-R10.4.1-basecallers/SRR26899119.fast-v4.2.0.150x.fastq.gz
R5=$DATA/ont-R10.4.1-basecallers/SRR26899121.sup-v3.5.1.150x.fastq.gz

/usr/bin/time -v pixi run raven --threads 8 $R1 > runs/raven.r10/assembly.fasta
/usr/bin/time -v pixi run raven --threads 8 $R2 > runs/raven.r9/assembly.fasta
/usr/bin/time -v pixi run raven --threads 8 $R3 > runs/raven.sup4/assembly.fasta
/usr/bin/time -v pixi run raven --threads 8 $R4 > runs/raven.fast4/assembly.fasta
/usr/bin/time -v pixi run raven --threads 8 $R5 > runs/raven.sup3/assembly.fasta

adrianylee-Bi621-PS6/runs/raven_R10.4.1-basecallers_SRR26899121.sup-v3.5.1.150x
```
Raven SLURM output stored in ```slurm-45540915.out```. Each run took approximately 1 minute to 1.5 minutes to run, 680 - 700% CPU usage, and used between 3-4 GB RAM. 

### Part 4 - CheckM2 (completeness and contamination)
I copied all of my contigs.fasta and assembly.fasta files from SPAdes and raven runs and copied them into a new directory that I created called ```checkm2_in/```. I created a script to do all of this and run check M2 over everything. This SLURm script is called ```runCheckM2.sh```. Command run for CheckM2:
```
DB=/projects/bgmp/shared/Bi621/PS6/checkm2_db/uniref100.KO.1.dmnd

/usr/bin/time -v pixi run -e env-checkm2 checkm2 predict \
    --threads 8 \
    --database_path "$DB" \
    --input checkm2_in/ \
    -x fasta \
    --output-directory checkm2_out/ \
    --force
```

Then, using these statistics, stored in ```checkm2_out/quality_report.tsv```, I reformatted my original ```contigDistribution.py``` to write out all my statistics for each run (both SPAdes and raven) including contamination and completeness information from the quality_report.tsv. All of these statistics were written out in a new file ```results.md```. This was accomplished by using the append function (and everything run all at once). Subsequent runs using this same script if needing to regenerate anything MUST DELETE ```results.md``` or will append to it. Also added additional code to parse the checkm2 and to be able to work with both spades and raven outputs (and a flag to specify which one).

All answers are written in ```answers.md```. Lab notebook updated 7/17/26.


