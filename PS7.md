# Lab Notebook for PS7 --> Working with BLASTp

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

Note: initial logic was wrong if dnaLine must be nested else first line will fail
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
- ```valid_gene``` - takes in the table and filters valid sequences (must have a protein ID) by checking the first column of each line. Stores each valid Gene ID as a key in a new dictionary genes with it's value being a list of [Protein ID, Gene Name, and an empty string]. If Gene name doesn't exist, stores Gene name as an empty string. Outputs the _genes_ dictionary.
- ```get_longest_gene``` - takes the _genes_ dictionary along with the updated fasta file (2 lines per record). Loops through the fasta file. For every sequence, checks to see if it exists in the dictionary (exits if not) if it does checks to see if the current sequence is longer than the existing one (in list position 2 of the key:value dictionary _genes_). If it is longer, replaces it. Repeats for all sequences updating the genes dictionary directly
- ```write_out``` - which takes the updated _genes_ dictionary and writes out a new fasta file using the information within.

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

Creation of blastp.sh using the SLURM batch headers. Then got on a work node within Talapas.
srun -A bgmp -p bgmp --time=2-00:00:00 --pty bash

Downloaded and installed blast via Pixi.
```
blastp -version
blastp: 2.17.0+
```

created a new bash script: makeblastdb.sh
blastp command: 
```
/usr/bin/time -v pixi run makeblastdb -in part1.human_longest_protein_per_gene.fa -dbtype prot -out Homo_sapiens -parse_seqids -title "Homo_sapiens"
/usr/bin/time -v pixi run makeblastdb -in part1.zebrafish_longest_protein_per_gene.fa -dbtype prot -out Danio_Rerio -parse_seqids -title "Danio_rerio"
```

ran with ```sbatch makeblastdb.sh```
```squeue -u aylee``` to check run

SLURM out: ```slurm-45260143.out```
Summary of statistics:
Took 1.24 seconds, 55% CPU, 0.059 GB RAM

This creates two databases that I am now using to run blastp with the following two commands in a new bash script: ```runblastp.sh```
```
/usr/bin/time -v pixi run blastp -query Homo_sapiens.GRCh38.pep.all.fa -db Danio_Rerio -evalue 1e-6 -use_sw_tback -out "human_to_zebrafish_blastp" -num_threads 8
/usr/bin/time -v pixi run blastp -query Danio_rerio.GRCz11.pep.all.fa -db Homo_sapiens -evalue 1e-6 -use_sw_tback -out "zebrafish_to_human_blastp" -num_threads 8
```
SLURM out file: ``````
Summary of statistics:

### 7/13/26

The files have an absurdly large amount of lines (and I did not specify the correct format)
Rerunning with the following commands and actually using the longest protein fasta that I generated before:
```
/usr/bin/time -v pixi run blastp -query part1.human_longest_protein_per_gene.fa -db Danio_Rerio -evalue 1e-6 -use_sw_tback -out "human_to_zebrafish_blastp" -num_threads 8 -outfmt 6
/usr/bin/time -v pixi run blastp -query part1.zebrafish_longest_protein_per_gene.fa -db Homo_sapiens -evalue 1e-6 -use_sw_tback -out "zebrafish_to_human_blastp" -num_threads 8 -outfmt 6
```

### 7/14/26

## Part 2 Continued

Outputs from the blast: 
```

```

Number of hits in each of the blastp files (Command: ``````):
```
human_to_zebrafish: 
zebrafish_to_human: 
```

Top 10 hits with highest bit scores. (Command: ``````)
```
human_to_zebrafish: 
zebrafish_to_human: 
```

All hits with the lowest evalue (sorted by bitscore) (Command: ``````) --> HLE.txt and ZLE.txt
```
human_to_zebrafish: HLE.txt
zebrafish_to_human: ZLE.txt
```

Differences between best bitscores vs best evalues:



