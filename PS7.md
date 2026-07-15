# Lab Notebook for PS7 --> Working with BLASTp
## __SUMMARY__
- (longestProtein.py)[longestProtein.py]


### 7/8/26
#### Working with Zach today

## Part 1 - Downloading protein fasta files and filtering to retain the longest protein per gene

Cloned the github PS7 repository into my computer and Talapas
While inside the /bioinfo/Bi621/PS directory on my own computer:
```
git clone https://github.com/2026-BGMP/adrianylee-Bi621-PS7.git
```

And similarly while logged into Talapas in the following directory: /projects/bgmp/aylee/bioinfo/Bi621/PS/. This allows me to work inside my directory and inside Talapas, pushing and pulling from github to update my scripts/project.

Copied bioinfo.py from PS5 while into the PS7 directory on Talapas via scp:
```
scp /bioinfo/Bi621/PS/adrianylee-Bi621-PS5 aylee@login.talapas.uoregon.edu:/projects/bgmp/aylee/bioinfo/Bi621/PS/adrianylee-Bi621-PS7
```

Visited Ensembl.org and downloaded the following .pep.all.fa.gz files using wget inside my Talapas PS7 directory. Files were found via downlods -> FTP downloads. The following commands were used within my Talapas directory ```/projects/bgmp/aylee/bioinfo/Bi621/PS/adrianylee-Bi621-PS7```:
```
- wget https://ftp.ensembl.org/pub/release-116/fasta/homo_sapiens/pep/
- wget https://ftp.ensembl.org/pub/release-116/fasta/danio_rerio/pep/
```

Returning to Ensembl -> Biomart -> Ensembl Genes 116 (database choice from dropdown) -> Human genes/Zebrafish genes (from dataset dropdown).
For each species
1. Unselected all attributes (go into the attribute menu by clicking it on the left hand side of the screen)
2. Checked the following attributes in order: Protein Stable ID, Gene Stable ID, Gene Name
3. Downloaded as a .tsv named and moved these two table files into Talapas (into my PS7 directory, ```/projects/bgmp/aylee/bioinfo/Bi621/PS/adrianylee-Bi621-PS7```):
     - human_proteinID_geneID_geneName.txt
     - zebrafish_proteinID_geneID_geneName.txt

Updated the bioinfo.py script with a new function: oneline_fasta. This takes an input fasta, makes each record two lines total and writes the resulting fasta file out. Worked with my group on this script (Zach, Pen, Rose). The script is as follows (takes a Fasta input and an output file name):

Note: initial attempt was wrong ```if dnaLine``` must be nested else first line will fail (first sequence will not append to a new line which ruins all downstream analyses)
```
def oneline_fasta(file, out):
    with open(file, "r") as fi:
        with open(out, "w") as fo:
            dnaLine = ""
            for line in fi:
                line = line.strip()
                if line.startswith(">"):
                    if dnaLine:
                        fo.write(f"{dnaLine}\n")
                    fo.write(f"{line}\n")
                    dnaLine = ""
                else:
                    dnaLine += line
            fo.write(f"{dnaLine}\n")
```

Created a new python script to retain the longest protein record per gene.
```
longestProtein.py
```

This script will takes in both the table downloaded from Biomart and the fasta file and gather all valid geneIDs with a protein sequence (not including gene variants) within the fasta file. It will then take the longest protein sequence for each gene ID and write all of this information out to a new, correctly formatted fasta file. 

This script is broken down to the following functions:
- ```fasta_one_liner``` - A function that runs the original fasta file through the one line fasta function created earlier to normalize all record lengths to two lines
- ```get_args``` - An argparse function to gather user input and allow it to be flexible with different datasets
- ```valid_gene``` - takes in the table and filters valid sequences (must have a protein ID) by checking the first column of each line. Stores each valid Protein ID as a key in a new dictionary genes with it's value being a list of [Gene ID, Gene Name, and an empty string]. If Gene name doesn't exist, stores Gene name as an empty string. Outputs the __uniqueProtein__ dictionary. The keys MUST be proteins because all proteins are unique (while genes are not). Initial attempt used Genes as a key and it caused the values to be overwritten each time a duplicate gene appeared in the fasta file.
- ```get_longest_gene``` - takes the __uniqueProtein__ dictionary along with the updated fasta file (2 lines per record). Loops through the fasta file. For every sequence, stores two lines at once (header and sequence). Grabs the proteinID and geneID from the header. Using the __uniqueProtein__ dictionary, makes sure each protein is only appended once. Creates a new dictionary __longestProteinPerGene__ which stores the geneID as the key and the proteinID, geneName, and longest sequence as a list-value pair. Using two dictionaries, one to grab unique proteins, and the second to only grab the longest protein sequence, this logic works effectively without overwriting anything. 
- ```write_out``` - which takes the updated __longestProteinPerGene__ dictionary and writes out a new fasta file using the information within.

Confirmed script works by checking line counts (2x more than expected records)

Attempted the challenge problem via bash, but didn't quite get it. Will revisit another day.

### 7/10/26
#### Working alone

## Part 1 - Downloading protein fasta files and filtering to retain the longest protein per gene (continued)

Reworked the bash command. Must be more specific else it finds other values (and is incorrect)
```grep ">" Danio_rerio.GRCz11.pep.all.fa | sed -E "s/.+gene:(ENSDARG[0-9]+).+\..+/\1/" | sort | uniq | wc -l``` -> outputs 30313
```grep ">" Homo_sapiens.GRCh38.pep.all.fa | sed -E "s/.+gene:(ENSG[0-9]+).+\..+/\1/" | sort | uniq | wc -l``` -> outputs 23879
Which match expected values

## Part 2 - BLASTp

Got on a work node within Talapas.
srun -A bgmp -p bgmp --time=2-00:00:00 --pty bash

Downloaded and installed blast via Pixi. Verification:
```
pixi init
pixi add python=3.14 blast
pixi shell
blastp -version
blastp: 2.17.0+
```

created a new bash script: makeblastdb.sh
blastp command: 
```
#!/bin/bash
#SBATCH --account=bgmp                    # REQUIRED: which account to use
#SBATCH --partition=bgmp                  # REQUIRED: which partition to use
#SBATCH --cpus-per-task=8                 # optional: number of cpus, default is 1
#SBATCH --job-name=blastDB                # optional: job name

/usr/bin/time -v pixi run makeblastdb -in part1.human_longest_protein_per_gene.fa -dbtype prot -out Homo_sapiens -parse_seqids -title "Homo_sapiens"
/usr/bin/time -v pixi run makeblastdb -in part1.zebrafish_longest_protein_per_gene.fa -dbtype prot -out Danio_Rerio -parse_seqids -title "Danio_rerio"
```

ran with ```sbatch makeblastdb.sh```
```squeue -u aylee``` to check run

SLURM out: ```slurm-45416275.out```
Summary of statistics:
The first blast database (for humans) took 0.81 seconds, used 80% of CPU, and 0.3 GB RAM. The second blast database command took 0.89 seconds, used 87% of CPU, and 0.063 GB RAM. 

This creates two databases that I am now using to run blastp with the following two commands in a new bash script: ```runblastp.sh```:
```
#!/bin/bash
#SBATCH --account=bgmp                    # REQUIRED: which account to use
#SBATCH --partition=bgmp                  # REQUIRED: which partition to use
#SBATCH --cpus-per-task=8                 # optional: number of cpus, default is 1
#SBATCH --job-name=runblastDB             # optional: job name


/usr/bin/time -v pixi run blastp -query part1.human_longest_protein_per_gene.fa -db Danio_Rerio -evalue 1e-6 -use_sw_tback -out "human_to_zebrafish_blastp" -num_threads 8 -outfmt 6
/usr/bin/time -v pixi run blastp -query part1.zebrafish_longest_protein_per_gene.fa -db Homo_sapiens -evalue 1e-6 -use_sw_tback -out "zebrafish_to_human_blastp" -num_threads 8 -outfmt 6
```
Make sure to specify number of threads, evalue, use_sw_tback, and output format. These are relevant for all downstream analyses.

SLURM out file: ```slurm-45416328.out```
Summary of statistics:
The first BLAST run (human query to zebrafish database) took 43 minutes and 29.23 seconds. Used 644% of CPU (6.4 cores), and used 0.617 GB Ram. The second BLAST run (zebrafish query to human database) took 36 minutes and 26.96 seconds. It used 789% cores and 0.747 GB RAM.

### 7/15/26

## Part 2 Continued

Outputs from the blast: 
```
human_to_zebrafish_blastp
zebrafish_to_human_blastp
```

Number of hits in each of the blastp files (Command: ```wc -l *blastp```):
```
human_to_zebrafish: 1986147
zebrafish_to_human: 2556891
```

Top 10 hits with highest bit scores. 
Commands: 
-- ```sort -k12,12 -n -r human_to_zebrafish_blastp | head -10```
-- ```sort -k12,12 -n -r zebrafish_to_human_blastp | head -10```

human_to_zebrafish:
```
    ENSG00000155657 ENSDARG00000000563      61.421  23829   8544    111     12403   35991   5733    29152   0.0     30359
    ENSG00000155657 ENSDARG00000028213      67.302  20671   6694    36      13626   34253   5617    26265   0.0     29867
    ENSG00000131018 ENSDARG00000063068      57.149  8812    3659    25      1       8797    1       8710    0.0     9902
    ENSG00000127481 ENSDARG00000009549      86.119  5194    667     16      20      5183    3       5172    0.0     8997
    ENSG00000127481 ENSDARG00000115260      86.102  5195    667     17      20      5183    3       5173    0.0     8992
    ENSG00000005810 ENSDARG00000113355      84.418  4749    658     24      32      4772    31      4705    0.0     8062
    ENSG00000128731 ENSDARG00000073841      82.457  4851    813     16      1       4834    1       4830    0.0     8047
    ENSG00000143341 ENSDARG00000016936      66.347  5634    1858    14      11      5635    12      5616    0.0     7940
    ENSG00000131018 ENSDARG00000009499      48.910  8857    4266    54      14      8797    27      8697    0.0     7929
    ENSG00000005810 ENSDARG00000001220      83.038  4746    598     19      32      4772    31      4574    0.0     7890
```

zebrafish_to_human: 
```
    ENSDARG00000000563      ENSG00000155657 63.604  22461   7785    50      6997    29152   13616   35991   0.0     30322
    ENSDARG00000028213      ENSG00000155657 67.315  20673   6688    37      5617    26265   13626   34253   0.0     29819
    ENSDARG00000063068      ENSG00000131018 57.149  8812    3659    25      1       8710    1       8797    0.0     9893
    ENSDARG00000009549      ENSG00000127481 85.943  5193    670     17      7       5172    24      5183    0.0     9017
    ENSDARG00000115260      ENSG00000127481 85.926  5194    670     18      7       5173    24      5183    0.0     9010
    ENSDARG00000073841      ENSG00000128731 82.543  4852    807     17      1       4830    1       4834    0.0     8118
    ENSDARG00000113355      ENSG00000005810 84.033  4785    670     26      1       4705    2       4772    0.0     8035
    ENSDARG00000009499      ENSG00000131018 48.967  8857    4261    53      27      8697    14      8797    0.0     7990
    ENSDARG00000016936      ENSG00000143341 66.347  5634    1858    14      12      5616    11      5635    0.0     7935
    ENSDARG00000001220      ENSG00000005810 82.643  4782    611     21      1       4574    2       4772    0.0     7854
```

All hits with the lowest evalue (sorted by bitscore) (Command: ``````) --> HLE.txt and ZLE.txt
HLE.txt (human query, zebrafish database)
```sort -k11,11g -k12,12rn human_to_zebrafish_blastp > sorted_human_to_zebrafish_blastp```
```awk '$11==0.0 {print $0}' sorted_human_to_zebrafish_blastp > HLE.txt```
total lines: 28383

ZLE.txt (zebrafish query, human database)
```sort -k11,11g -k12,12rn zebrafish_to_human_blastp > sorted_zebrafish_to_human_blastp```
```awk '$11==0.0 {print $0}' sorted_zebrafish_to_human_blastp > ZLE.txt```
total lines: 27995

Some notes on sort: -g is a generic number sort, need to use it for values with "e" in them. -n is a numeric sort use it on pure numbers. -r allows you to sort in reverse order (very helpful to get in the order you want). You can also use multiple -k flags to sort by multiple different categories, sequentially.

This lab notebook was mostly polished on 7/15/26



