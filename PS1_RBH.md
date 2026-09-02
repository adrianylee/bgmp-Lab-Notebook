Lab Notebook for PS1 --> RBH

## Summary
- Isolating reciprocal best hits from two genome files

Main Files (input/output)
- [RBH.py](https://github.com/2026-BGMP/adrianylee-Bi621-PS7/blob/main/RBH.py)
- [Rbh.sh](https://github.com/2026-BGMP/adrianylee-Bi621-PS7/blob/main/RBH.sh)
- [RBH tsv files](https://github.com/2026-BGMP/adrianylee-Bi621-PS7/tree/main/RBHs)
- [pseudocode](https://github.com/2026-BGMP/adrianylee-Bi621-PS7/blob/main/pseudocode.md)
- [answers](https://github.com/2026-BGMP/adrianylee-Bi621-PS7/blob/main/PS1_answers.txt)

Test files are attached in part 1.5 below.

Software Versions
python = "3.14.*"

No other software packages was used. BLAST was already done or given
---

## 8/25/2026
### Working with Rose and Alex
### Part 1 + 2

Worked on pseudocode alone, completing RBH best_hits pseudocode (rough draft), before working extensively with Alex and Rose on the rest of the logic. Got a complete idea of code and worked past potential issues when implementing the code.

Using existing blastp files, used sort with specific flags (g for general due to evalues, and n for numerics, and cut out relevant columns. RBH will only depend on queryID, subjectID, and evalue. Using this as the input, designed a python script based on discussion and pseudocode. arg parse was copied from previous assignments and modified for usage. 

best_hits() takes the sorted and cut blast file as input to create a dictionary of best hits (for that blast) and a set that checks if a queryID has already been seen. If a queryID has a duplicate evalue, it is removed from the dictionary, otherwise the lowest evalue is taken. Dictionary stores queryID: [subjectID, e-value]. Duplicate queryID, subjectID are kept if the evalues differ. This is done on both blast runs.

Using the two resulting dictionaries, does the following: for each s1 protein finds the best S2 hit, checking if s2 has s1 as its best hit. If both are best hits of each other, it is saved as within an RBH dictionary.

readTable uses the exact same logic as PS7. Storing information in a dictionary: proteinID: [geneID, geneName] for fast lookup when writing output.

The output takes the RBH hits, looks up the corresponding ID from the table dictionary to generate an output file with the following format (tab separated): S1_GeneID S1_ProteinID S1_GeneName S2_GeneID S2_ProteinID S2_GeneName, as specified by the assignment outline. 

Completed test files. Copied the gene IDs from the top of the file but updated the evalues. Code works on test files. Test files can be found here:
[1](https://github.com/2026-BGMP/adrianylee-Bi621-PS7/blob/main/testblast1_blastp)
[2](https://github.com/2026-BGMP/adrianylee-Bi621-PS7/blob/main/testblast2_blastp)

Updated my code so that it will automatically set test files as default. Used it to test code.

## 8/29/2026
### Working with Wesam/Hannah
### Part 1.5? Exchanged test files

Was really out of it all week, ended up missing RBH test swap so did it with my roommates. Exchanged and ran test files. Had issues with copying pasting test files from slack. Used scp, to move a downloaded file into Talapas for testing.

## 9/1/2026
### Working alone (with Wesam discussion)
### Part 2-5

Wrote a SLURM script to finish the rest of the RBH comparisons. Had to add #SBATCH #SBATCH --time=3:00:00. Otherwise, it would crash due to maintenance. All other runs followed the logic of the initial run, and correct file locations were inputted. No issues on run. SLURM output: ```slurm-47030748.out```

Each run took approximately ~1 second, using 88% CPU usage and 146 MB Ram (used bc to calculate).

Answers were updated in the notebook. Results for run:
```
 wc -l *
  10186 DreEelRBH.tsv
   9342 DrePkaRBH.tsv
  10663 EelPkaRBH.tsv
   9023 HsaEelRBH.tsv
   9036 HsaPkaRBH.tsv
   7962 humanZebrafishRBH
```
subtract 1 from every file to account for the header. Thus:
```
  10185 DreEelRBH.tsv
   9341 DrePkaRBH.tsv
  10662 EelPkaRBH.tsv
   9022 HsaEelRBH.tsv
   9035 HsaPkaRBH.tsv
   7961 humanZebrafishRBH
```

Talked with Wesam about analysis questions. Checked the genespace ICA2, to compare pangene orthorologs to RBHs. Findings outlined in the PS1 answers, linked above. For the ortholog hits I semi-randomly sampled, most of them were also RBH hits. 

This notebook was last updated on: 9/1/26

