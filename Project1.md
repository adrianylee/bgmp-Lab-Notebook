# Lab Notebook for Project1
## Summary
- 

Main Files (input/output)

Part 1  
- [pseudocode](https://github.com/2026-BGMP/adrianylee-Bi623-Project-1/blob/main/part1pseudocode.md)
- [python script](https://github.com/2026-BGMP/adrianylee-Bi623-Project-1/blob/main/part1.py)
- [bash script](https://github.com/2026-BGMP/adrianylee-Bi623-Project-1/blob/main/part1.sh)
- [sorted Rocc output](https://github.com/2026-BGMP/adrianylee-Bi623-Project-1/blob/main/PhyloP_RoCC_output_sorted.txt)


Part 2
- 

**Software Versions**  
PART 1
Python 3.14.6

PART 2 


### 9/3/26
#### Working with Rose

Worked with Rose to sketch a framework of design for part 1. Also talked with Hope to figure out some immediate issues with storing specific types of information. 

Rough sketch of part 1 logic, updated pseudocode. Planning to store information in two dictionaries, although it should be possible with one. Also figured out how to implement filtering and merging logic. 

### 9/4/26
#### Working with Rose

Created test file. Updated pseudocode and started converting to to actual code. Created SLURM script. Same format. Here's what it looks like now:
```
#!/bin/bash
#SBATCH --account=bgmp                    # REQUIRED: which account to use
#SBATCH --partition=bgmp                  # REQUIRED: which partition to use
#SBATCH --cpus-per-task=8                 # optional: number of cpus, default is 1
#SBATCH --job-name=part1                  # optional: job name
#SBATCH --time=11:00:00

/usr/bin/time -v python part1.py -f /projects/bgmp/shared/Bi623/ZoonomiaWorkshop/241-mammalian-2020v2.bigWigToBedGraph.gz -m 20 -o rocc_output.txt
/usr/bin/time -v sort -k4,4nr -k1,1Vr -k2,2nr rocc_output.txt > sorted_rocc_output.txt
```

### 9/5/26 - 9/7/26
#### Working Alone

Iterative testing until scripts run without issues. Python script will use gzip and store information of previous lines to check for initial filtering/merging logic on the first pass. Makes sure phylo score is high enough. Makes sure continuous RoCCs are merged. Makes sure resulting output is 2 bp long. Takes this initially sorted dictionary and loops thorugh it, applying secondary merging logic. All RoCCs that are 1 bp away within the same chromosome and at least 2 bp long will be merged. New outputs are stored in a final dictionary. Uses the same storing logic as the first pass. Finally outputs to directed file. Ran tests on my own test file and Colin's.

Slurm output is ```slurm-47118180.out```. Took 28 minutes and 46 seconds and 99% CPU. Used 9.8977 GB RAM. Sorting logic took 2 seconds to run using 346% CPU and negligible memory.

Moving on to Part 2. 
---

Located the clinical data via Talapas: ```/projects/bgmp/shared/Bi623/ZoonomiaWorkshop/variant_summary.txt.gz```. Used scp to move to my computer and opened up in R markdown. Using tidyverse and dplyr packages. 

Full R code in markdown:
library(tidyverse)
library(dplyr)
```

## R Markdown

This is an R Markdown document. Markdown is a simple formatting syntax for authoring HTML, PDF, and MS Word documents. For more details on using R Markdown see <http://rmarkdown.rstudio.com>.

When you click the **Knit** button a document will be generated that includes both content as well as the output of any embedded R code chunks within the document. You can embed an R code chunk like this:

```{r cars}
read_lines("variant_summary.txt.gz", n_max = 1)

variants = read_tsv(
  "variant_summary.txt.gz", 
  col_select = c(
    Name, 
    GeneSymbol, 
    ClinicalSignificance, 
    PhenotypeList, 
    Assembly, 
    ChromosomeAccession, 
    Chromosome, 
    Start, 
    Stop))
```

```{r}
Cranio = variants %>%
  filter(
    str_detect(PhenotypeList, "Cranio"),
    ClinicalSignificance %in% c(
      "Pathogenic",
      "Pathogenic/Likely pathogenic"),
    Assembly == "GRCh38"
    ) %>%
  select(Chromosome, Start, Stop, PhenotypeList)

Cranio = Cranio[order(as.numeric(Cranio$Chromosome), Cranio$Start), ] %>%
  mutate(Chromosome = paste0("chr", Chromosome))


```
```{r}
write_tsv(Cranio, "Cranio_variants_sorted.tsv")
```



