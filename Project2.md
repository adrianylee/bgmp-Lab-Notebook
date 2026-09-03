Lab Notebook for Project2
## Summary
- 

Main Files (input/output)
- []()

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

