## 7/9/2026
### Working with Zach

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
sbatch spadesTest.sh -- spadesTest.sh formatted based on github
```

To check the run:
- cd spades.k21/ - in the directory where I ran sbatch
- squeue -u aylee - to check if somethings running
