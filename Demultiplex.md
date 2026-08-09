# Demultiplexing

## Summary

Files written (input/output):

### Assignment the First

- [qualityScoreDistribution.py](https://github.com/adrianylee/Demultiplex/blob/master/Assignment-the-first/qualityScoreDistribution.py) - reads the sequencing FASTQ files and calculates the mean Phred quality score at each nucleotide position
- [qualityScoreDistribution.sh](https://github.com/adrianylee/Demultiplex/blob/master/Assignment-the-first/qualityScoreDistribution.sh) - SLURM script used to run the quality-score distribution program
- [bioinfo.py](https://github.com/adrianylee/Demultiplex/blob/master/Assignment-the-first/bioinfo.py) - contains functions used for Phred-score calculations and sequence manipulation
- [pseudocode.md](https://github.com/adrianylee/Demultiplex/blob/master/Assignment-the-first/pseudocode.md) - pseudocode outlining the initial demultiplexing strategy
- [Answers.md](https://github.com/adrianylee/Demultiplex/blob/master/Assignment-the-first/Answers.md) - written answers for Assignment the First
- [R1.png](https://github.com/adrianylee/Demultiplex/blob/master/Assignment-the-first/R1.png) - mean quality distribution for read 1
- [R2.png](https://github.com/adrianylee/Demultiplex/blob/master/Assignment-the-first/R2.png) - mean quality distribution for read 2
- [I1.png](https://github.com/adrianylee/Demultiplex/blob/master/Assignment-the-first/I1.png) - mean quality distribution for index 1
- [I2.png](https://github.com/adrianylee/Demultiplex/blob/master/Assignment-the-first/I2.png) - mean quality distribution for index 2

### Test Data

- [TEST-input_FASTQ](https://github.com/adrianylee/Demultiplex/tree/master/TEST-input_FASTQ) - test Fastq
- [TEST-output_FASTQ](https://github.com/adrianylee/Demultiplex/tree/master/TEST-output_FASTQ) - expected fast1 output

### Assignment the Third

- [bioinfo.py](https://github.com/adrianylee/Demultiplex/blob/master/Assignment-the-third/bioinfo.py) - contains previously written functions used by the final demultiplexing program, with updated functions
- [demultiplex.py](https://github.com/adrianylee/Demultiplex/blob/master/Assignment-the-third/demultiplex.py) - demultiplexes. takes in reads, outputs read statistics.
- [demultiplex.sh](https://github.com/adrianylee/Demultiplex/blob/master/Assignment-the-third/demultiplex.sh) - SLURM script used to run `demultiplex.py` 
- [Q30 results.md](https://github.com/adrianylee/Demultiplex/blob/master/Assignment-the-third/Q30%20results.md) - user friendly summary of demultiplexing statistics
- [Q30IndexPairHeatmap.png](https://github.com/adrianylee/Demultiplex/blob/master/Assignment-the-third/Q30IndexPairHeatmap.png) - log10 heatmaps showing counts for all possible known index pairs for threshold = 30
- [Q30BarcodePercentages.png](https://github.com/adrianylee/Demultiplex/blob/master/Assignment-the-third/Q30BarcodePercentages.png) - percentage of successfully matched reads for each barcode
- [Q30PiChart.png](https://github.com/adrianylee/Demultiplex/blob/master/Assignment-the-third/Q30PiChart.png) -pi chart for matched, hopped, unknown
- `summary.tsv` - generated within each `/scratch` output directory and contains counts for all 576 possible known index pairs as well as total matched, hopped, and unknown counts
- Full demultiplexed FASTQ files are written to `/scratch/bgmp/aylee/demux/` and are not uploaded to GitHub

Software version info:

- Python = `>=3.14.6,<3.15`
- numpy = `>=2.5.1,<3`
- matplotlib = `>=3.11.1,<4`

---

### 7/22/26

#### Working alone

## Assignment the First: Part 1 Quality Score Distribution per-nucleotide

I used a number of commands to examine the data and how it was formatted.

```
zcat /projects/bgmp/shared/2017_sequencing/1294_S1_L008_R3_001.fastq.gz | head -2
zcat /projects/bgmp/shared/2017_sequencing/1294_S1_L008_R4_001.fastq.gz | head -6
zcat /projects/bgmp/shared/2017_sequencing/1294_S1_L008_R1_001.fastq.gz | head -6
zcat /projects/bgmp/shared/2017_sequencing/1294_S1_L008_R3_001.fastq.gz | head -6
zcat /projects/bgmp/shared/2017_sequencing/1294_S1_L008_R2_001.fastq.gz | head -6
zcat /projects/bgmp/shared/2017_sequencing/1294_S1_L008_R1_001.fastq.gz | head -2 | tail -1 | wc
zcat /projects/bgmp/shared/2017_sequencing/1294_S1_L008_R2_001.fastq.gz | head -2 | tail -1 | wc
```

This initial data exploration showed which files were FASTQ biological sequence files (R1 and R4) and which ones were FASTQ index files (R2 and R3). I also checked to verify that the files were FASTQ formatted, determined the sequence lengths, and examined the quality-score encoding.

This information was helpful in filling out the following table:

| File name | label | Read length | Phred encoding |
| ---------- | ----- | ----------- | -------------- |
| 1294_S1_L008_R1_001.fastq.gz | read1 | 101 | phred33 |
| 1294_S1_L008_R2_001.fastq.gz | index1 | 8 | phred33 |
| 1294_S1_L008_R3_001.fastq.gz | index2 | 8 | phred33 |
| 1294_S1_L008_R4_001.fastq.gz | read2 | 101 | phred33 |

I then created [qualityScoreDistribution.py](https://github.com/adrianylee/Demultiplex/blob/master/Assignment-the-first/qualityScoreDistribution.py) to calculate the per-base quality distributions.

The script reads through each FASTQ file and keeps a running total of the Phred quality score at each nucleotide position rather than storing the entire dataset in memory. The total score at each position is divided by the total number of records to calculate the mean quality score per nucleotide.

I ran the program using [qualityScoreDistribution.sh](https://github.com/adrianylee/Demultiplex/blob/master/Assignment-the-first/qualityScoreDistribution.sh).

The four resulting plots are shown below.

### Read 1

![R1 quality score distribution](https://raw.githubusercontent.com/adrianylee/Demultiplex/master/Assignment-the-first/R1.png)

### Read 2

![R2 quality score distribution](https://raw.githubusercontent.com/adrianylee/Demultiplex/master/Assignment-the-first/R2.png)

### Index 1

![Index 1 quality score distribution](https://raw.githubusercontent.com/adrianylee/Demultiplex/master/Assignment-the-first/I1.png)

### Index 2

![Index 2 quality score distribution](https://raw.githubusercontent.com/adrianylee/Demultiplex/master/Assignment-the-first/I2.png)

I also counted the number of index reads containing undetermined (`N`) base calls.

Index 1 with N base calls: 3976613  
Index 2 with N base calls: 3328051

Commands used:

```
zcat /projects/bgmp/shared/2017_sequencing/1294_S1_L008_R2_001.fastq.gz | grep -A 1 "^@" | grep -v "^@" | grep -c "N"
zcat /projects/bgmp/shared/2017_sequencing/1294_S1_L008_R3_001.fastq.gz | grep -A 1 "^@" | grep -v "^@" | grep -c "N"
```

These plots and initial analyses were useful for deciding on appropriate quality-score cutoffs to test during demultiplexing.

Written answers for this portion are stored in [Answers.md](https://github.com/adrianylee/Demultiplex/blob/master/Assignment-the-first/Answers.md).

---

### 7/29/26

#### Working alone

## Assignment the First: Part 2 Demultiplexing Strategy

Started developing the pseudocode and test FASTQ files for the actual demultiplexing algorithm.

The basic problem is to take the following FASTQ files:

```
R1 = read 1
R2 = index 1
R3 = index 2
R4 = read 2
```

and sort the R1/R4 biological read pair based on its corresponding indexes. R3 must be reverse complemented before comparing it to R2

There are three possible output categories:

```
matched = both indexes are known and identical after reverse complementing R3
hopped = both indexes are known but are different (in indexes.txt)
unknown = one or both indexes are unknown, contain N, or fail the quality cutoff
```
There are 24 known indexes. I need to keep track of both correctly matched combinations and every possible known index-hopping combination.
There should therefore be possible index pairs (permutations):

```
24 * 24 = 576
```
Strategy outlined in [pseudocode.md](https://github.com/adrianylee/Demultiplex/blob/master/Assignment-the-first/pseudocode.md). 
Thanks to Hannah B, Hannah K, and Pen for feedback.

The initial high-level functions included:

- `read_indexes` - reads all of the known indexes from the index file
- `initialize_indexes` - creates a dictionary containing every possible known index pair
- `reverse_complement` and `average_phred` - already exists inside `bioinfo.py`
- `write_record` - writes a biological FASTQ record to the appropriate output and adds the corrected index pair to the header
- `demultiplex` - demultiplexes

I also created four test FASTQ files in [TEST-input_FASTQ](https://github.com/adrianylee/Demultiplex/tree/master/TEST-input_FASTQ):

- [R1Test.fastq](https://github.com/adrianylee/Demultiplex/blob/master/TEST-input_FASTQ/R1Test.fastq)
- [R2Test.fastq](https://github.com/adrianylee/Demultiplex/blob/master/TEST-input_FASTQ/R2Test.fastq)
- [R3Test.fastq](https://github.com/adrianylee/Demultiplex/blob/master/TEST-input_FASTQ/R3Test.fastq)
- [R4Test.fastq](https://github.com/adrianylee/Demultiplex/blob/master/TEST-input_FASTQ/R4Test.fastq)

The test data contain records covering all three required categories.

Expected result:

```
Matched: 2
Hopped: 1
Unknown: 1
```

Expected output FASTQs are stored in [TEST-output_FASTQ](https://github.com/adrianylee/Demultiplex/tree/master/TEST-output_FASTQ).

The main pseudocode strategy was:

### 8/4/26

#### Working alone

## Assignment the Third: Initial Demultiplexing Script

Started translating the pseudocode into the actual script
The input files are:

```
/projects/bgmp/shared/2017_sequencing/1294_S1_L008_R1_001.fastq.gz
/projects/bgmp/shared/2017_sequencing/1294_S1_L008_R2_001.fastq.gz
/projects/bgmp/shared/2017_sequencing/1294_S1_L008_R3_001.fastq.gz
/projects/bgmp/shared/2017_sequencing/1294_S1_L008_R4_001.fastq.gz
```

The known index file is:

```
/projects/bgmp/shared/2017_sequencing/indexes.txt
```

`read_indexes()` reads the final column of the index file and stores the 24 valid 8 bp index sequences.

`initialize_indexes()` calls `read_indexes()` and then uses:

```
itertools.product(indexList, repeat=2)
```

to create a dictionary containing all 576 possible ordered known index pairs. All counts initially start at zero.

The sequencing FASTQ files are opened using:

```
gzip.open(file, "rt")
```

**All input and output files are opened before entering the main demultiplexing loop.**

The main loop reads one complete FASTQ record (4 lines) from each of the four synchronized files:

```
read1 = (r1.readline(), r1.readline(), r1.readline(), r1.readline())
index1Read = (r2.readline(), r2.readline(), r2.readline(), r2.readline())
index2Read = (r3.readline(), r3.readline(), r3.readline(), r3.readline())
read2 = (r4.readline(), r4.readline(), r4.readline(), r4.readline())
```

R2 is used directly as index1. The R3 sequence is reverse complemented using:

```
bioinfo.reverse_complement()
```

The corrected index pair is stored in the following format:

```
index1-index2
```

This pair is added to the header of both biological reads using `write_record()`.

Total output files: 

```
48 FASTQ files for properly matched indexes
2 FASTQ files for index-hopped reads
2 FASTQ files for unknown/low-quality reads
```

q30 resutls:

```
Matched: 226715602
Hopped: 330975
Unknown: 136200158
```

Total:

```
226715602 + 330975 + 136200158 = 363246735
```

The total number of records was correct (matches data found at the beginning)
---

### 8/6/26

#### Working alone

## Assignment the Third: Debugging quality filtering

Continued debugging the quality filtering.

Confirmed that the quality cutoff should be based on the average Phred score of each complete 8 bp barcode, rather than requiring every individual nucleotide to hit quality cutoff

Changed the quality calculation to:

```
quality1 = bioinfo.qual_score(index1Read[3].strip())
quality2 = bioinfo.qual_score(index2Read[3].strip())
```

`bioinfo.qual_score()` calculates the average Phred+33 score across the quality string for each index.

I initially tested the filtering condition using:

```
quality1 < qualityScoreCutoff
```

With `< 30` I obtained:

```
total num records: 363246735
known barcode records: 304980270
hopped barcode records: 517612
unknown barcode records: 57748853
```
which did not match with my classmates. 
I then confirmed that the intended filtering used `<=` rather than `<`. This means an index with an average quality score exactly equal to the cutoff is also placed into the unknown/low-quality category.
updated results:
```
Matched: 303645222
Hopped: 513040
Unknown: 59088473
```

### 8/7/26

#### Working alone

## Assignment the Third continued

Continued writing/debugging and organizing demultiplex.py. Added documnetation.

Added argparse arguments for:

```
-r1 / --read1
-r2 / --index1
-r3 / --index2
-r4 / --read2
-i  / --indexes
-t  / --threshold
-o  / --outputDir
-u  / --uncompressed
```

The normal sequencing inputs are compressed, so the default input method uses:
```
gzip.open()
```
The `-u/--uncompressed` option allows the program to run on the small uncompressed test FASTQ files using normal `open()`.

I kept `write_record()` as a separate function because the same four FASTQ-writing operations are required multiple times for matched, hopped, and unknown reads.

Created [demultiplex.sh](https://github.com/adrianylee/Demultiplex/blob/master/Assignment-the-third/demultiplex.sh) to submit python script

General SLURM format (same as all previous)

```
#!/bin/bash
#SBATCH --account=bgmp
#SBATCH --partition=bgmp
#SBATCH --cpus-per-task=8
#SBATCH --job-name=demultiplex
```

The large output FASTQ files are written to `/scratch/bgmp/aylee/demux/q{score cutoff}` rather than the GitHub repository.

---

### 8/8/26

#### Working alone

## Assignment the Third: Reports, figures, and final testing

Added `Q30 Results.md` as a detailed output for 130 run (or outputted for whatever threshold) with the following: 

- quality-score cutoff
- total read pairs
- matched count
- hopped count
- unknown count
- matched percentage
- index-hopping percentage
- unknown percentage
- matched count for each barcode
- percentage of successfully matched reads represented by each barcode

The Markdownis for the user and generates in the current directory= while `summary.tsv` can be used for more processing later, in scratch.

### Index Pair Heatmaps

The 576 known index-pair counts are converted into a 24 x 24 numpy matrix. The index sequences are alphabetically sorted along both axes before the matrix is generated. Generated using standard format for matplotlib heatmaps

![Q30IndexPairHeatmap.png](https://github.com/adrianylee/Demultiplex/blob/master/Assignment-the-third/Q30IndexPairHeatmap.png)

### Barcode Percentage Bar Graphs

For each known barcode, the program gets the corresponding properly matched count percentage. Sorted by biggest to smallest:
![Q30 Barcode Percentages](https://raw.githubusercontent.com/adrianylee/Demultiplex/master/Assignment-the-third/Q30BarcodePercentages.png)

### Pie Chart
![Q30 Read Categories](https://raw.githubusercontent.com/adrianylee/Demultiplex/master/Assignment-the-third/Q30PiChart.png)

Final results

The primary Q30 run produced:

```
Total read pairs: 363246735
Matched: 303645222
Hopped: 513040
Unknown: 59088473
Matched: 83.5920%
Index hopped: 0.1412%
Unknown: 16.2668%
```

The final Assignment-the-Third GitHub directory contains the code, SLURM script, Q0/Q20/Q30 Markdown result files, and all final figures. The large demultiplexed FASTQ files and `summary.tsv` outputs remain in the appropriate `/scratch/bgmp/aylee/demux/` directories rather than being pushed to GitHub.

Lab notebook finalized on 8/8/26.
