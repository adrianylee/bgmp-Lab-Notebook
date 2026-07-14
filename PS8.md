## 7/13/26

Working with Imre, Diya, Rose
```
srun -A bgmp -p bgmp --time=2-00:00:00 --pty bash
```

Within PS8/dre
```
wget https://ftp.ensembl.org/pub/release-116/fasta/danio_rerio/dna/Danio_rerio.GRCz11.dna.primary_assembly.fa.gz
wget https://ftp.ensembl.org/pub/release-116/gtf/danio_rerio/Danio_rerio.GRCz11.116.gtf.gz
```
Unzipped both files with ```gunzip``` as STAR --genomeFastaFiles flag only takes unzipped input


Then,
```
cd /projects/bgmp/aylee/bioinfo/Bi621/PS/adrianylee-Bi621-PS8/
pixi init
pixi add star
pixi add samtools
pixi shell
STAR --version                        ### should be 2.7.11b
samtools --version                    ### should be 1.23.1
```

created starDB.sh
```
#!/bin/bash
#SBATCH --account=bgmp                    # REQUIRED: which account to use
#SBATCH --partition=bgmp                  # REQUIRED: which partition to use
#SBATCH --cpus-per-task=8                 # optional: number of cpus, default is 1
#SBATCH --mem=16GB                        # optional: amount of memory, default is 4GB per cpu
#SBATCH --job-name=star                   # optional: job name


/usr/bin/time -v pixi run STAR --runThreadN 8 --runMode genomeGenerate \
--genomeDir STAR_2.7.11b-Danio_rerio.GRCz11.dna-ens116 \
--genomeFastaFiles dre/Danio_rerio.GRCz11.dna.primary_assembly.fa \
--sjdbGTFfile dre/Danio_rerio.GRCz11.116.gtf
```
chmod 755 starDB.sh
ran with sbatch starDB.sh

ran out of memory --> increased mem=32GB in the SLURM (default is 4GB per cpu)

you can also use ```bc``` to open a calculator in the terminal. ```scale=x``` when you launch where x is the number of decimals.

SLURM.out stored: ```/projects/bgmp/aylee/bioinfo/Bi621/PS/adrianylee-Bi621-PS8/slurm-45207541.out```
Used ~27.61 GB

Created an alignReads.sh script.
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

output SLURM:
```slurm-45207818.out```

### Samtools
```/usr/bin/time -v pixi run samtools view -b -o zebrafish.bam zebrafishAligned.out.sam```
slurm-45208357.out

commands used to sort, extract, and create BAM file:
```
/usr/bin/time -v pixi run samtools sort zebrafish.bam -o zebrafish.sorted.bam
/usr/bin/time -v pixi run samtools index zebrafish.sorted.bam
/usr/bin/time -v pixi run samtools view zebrafish.sorted.bam 1 -o chr1_sorted_zebrafish.sam
```

Number of alignments on chr1: ```grep -c "^K00337" chr1_sorted_zebrafish.sam``` --> 921737 ```wc -l chr1_sorted_zebrafish.sam ``` yields the same

wrote a python program that iterates through all of the SAM lines and counts mapped and unmapped reads based on two bitwise flags (0x4 or 4) and (0x100 or 256) to check primary alignment and checking to see if they are unmapped to avoid double counts.


All files uploaded, answers.md updated, everything pushed to github.






