# De-novo assembly of a _Wolbachia_ genome

This document contains instructions for generating a de-novo assembly of a _Wolbachia_ genome for BMEB bootcamp 2024. This is part 1 of the computational portion of the bootcamp.

I recommend that you clone this repository to your local computer, and open up this document in a text editor. That way you can save any changes you make to the code in this tutorial (like file paths).

> The output files for all steps in this tutorial can be found in our group folder on hummingbird: `/hb/groups/bmebootcamp-2024/wWil_results`. If you get stuck and fall behind, feel free to use these files to move ahead.  

## 0. Log onto the hummingbird server and create a project directory

You can access the hummingbird server with your UCSC username and Gold password. Please see the humming bird wiki on ["Getting Started"](https://hummingbird.ucsc.edu/getting-started/) for more details. (Feel free to run this analysis on your local computer if you prefer, but note that we may not be familiar with your operating system if you need help.)

Open your terminal and type:
```
ssh <your_cruzid>@hb.ucsc.edu
```

You should be in your home directory. You can check the directory you are in by typing `pwd`. This stands for "print working directory" Mine looks like this:
```
[aanakamo@hb ~]$ pwd
/hb/home/aanakamo
```

Create a new folder in your home directory for this analysis. Its good practice to keep your directories well organized.
```
mkdir bootcamp2024
```

Change into this new directory for the rest of the analysis
```
cd bootcamp2024
```

> Note: If you are new to linux commands, please refer to the [provided reference slides](https://docs.google.com/presentation/d/1hjIfozfQkjL4gj1eUtvBqgzpWERAkF8Uw43ToAQSxa8/edit#slide=id.p)  

## 0.5. Start an interactive slurm job

Before running more computationally intensive commands, you want to reserve space on a node. This is good cluster ettiquite and ensures that we don't back up the hummingbird login node, which could otherwise become slow for other users if everyone runs stuff there. We can do this by submitting a job to slurm, or by starting an interactive job. Here we will start an interactive job, so that you can see what's happening during the assembly process and test things out.

Start an interactive job (that will last for 3 hours) by running:
```
## request resources
salloc --partition=instruction --time=03:00:00 --mem=4G --tasks=1 --cpus-per-task=1
```
If you want, you can see what all the options mean by running `salloc -h`, or visiting [this humminbird tutorial](https://hummingbird.ucsc.edu/documentation/getting-an-interactive-allocation-for-instructional-use/).

Once granted resources on a node, ssh to that node
```
ssh ${SLURM_NODELIST}
```

Once you ssh to the node where you've been granted resources, you should see the host in the terminal prompt change from the login node (ie. `aanakamo@hb-login`) to a different node (ie. `aanakamo@hbnode-03`). It will also return you to your home directory, so change back into your bootcamp directory.
```
cd bootcamp2024
```

Please remember to start interactive jobs (or submit a job to slurm) whenever you're downloading files, installing/running tools, etc!

Once you are done running things, you can end the interactive job by running `exit`, which will end the job and return you to the login node. Or, the job will end once it reaches the time limit, but try to remember to exit when you're done. For now though, leave it running as you move onto the next step.

## 1. Access the fastq files produced by the Guppy basecaller

The fastq files from our preliminary nanopore experiments are located in our shared group directory at `/hb/groups/bmebootcamp-2024/Wwil_fastq`. We will use last year's data (from _Wolbachia willistoni_) for the purposes of this tutorial, while waiting for the data from the libraries you all generated for wRi. The fastq files that will be generated from the nanopore library you created for wRi will be here: `/hb/groups/bmebootcamp-2024/${coming_soon}`. 

The fastq file we will be working with in that directory is called `wWil.merged.fastq.gz`. Make sure you're still in your bootcamp directory (the the `~` indicates your home directory, where your bootcamp directory is located). Then copy the fastq file into your bootcamp folder (the `.` indicates the current directory, which is bootcamp2024):
```
cd ~/bootcamp2024
cp /hb/groups/bmebootcamp-2024/Wwil_fastq/wWil.merged.fastq.gz .
```

Now `ls` and see that `wWil.merged.fastq.gz` has been copied into your bootcamp directory.

## 2. Preprocessing data

First we need to make sure to remove any duplicate reads from the fastq files.

We can use `seqkit` for this, which is available as a module on hummingbird. Load the module by running:
```
module load seqkit
```

You can check that it loaded properly by running the following command, and you should see `seqkit/2.5.1` in the list of currently loaded modules.
```
module list
```

Now run the following to remove duplicates
```
time seqkit rmdup wWil.merged.fastq.gz -o wWil.merged.rmdup.fastq.gz
```
> Note: This took about 1 minute to run  

When seqkit finishes, you'll see the following message indicating that there were duplicate reads removed.
```
[INFO] 203581 duplicated records removed
```
This step is important, since assembly with Flye will error if there are duplicates.

All of the tools we will be using for the assembly portion of this tutorial are available as modules on hummingbird. Each module contains the exact dependencies needed for running the particular tool, and these can sometimes conflict if you have more than one module loaded (doesn't always happen, but it's a good habit to keep only modules you need loaded). So, lets unload `seqkit` before moving on.
```
module unload seqkit
```

## 3. Running Flye assembler

Next we will use Flye to perform genome assembly. Load the module:
```
module load flye
```

The [Flye manual](https://github.com/fenderglass/Flye/blob/flye/docs/USAGE.md) gives a whole list of all the possible parameters we can give Flye. You can also check these by running `flye -h`. Please read through the section in the manual giving descriptions of these parameters [(here)](https://github.com/fenderglass/Flye/blob/flye/docs/USAGE.md#-parameter-descriptions) and make sure you understand why this is the command we need to run:
```
# create output directory for flye
mkdir flye

# run flye assembler
time flye --nano-hq wWil.merged.rmdup.fastq.gz -t 1 --out-dir flye
```
> Note: Flye took me about ___ to run on one thread.

In the interest of time, it would be a good idea to grab the flye output from our shared folder and continue to the next step. You can always try running flye all the way through on your own time.
```
cp -r /hb/groups/bmebootcamp-2024/wWil_results/flye .
```

Take a look at the output of Flye
```
cd flye; ls
```

You should see the following files in your directory
```
[aanakamo@hb flye]$ ls
00-assembly   30-contigger    assembly_graph.gfa  flye.log
10-consensus  40-polishing    assembly_graph.gv   params.json
20-repeat     assembly.fasta  assembly_info.txt
```

Consult the Flye manual about what these files represent. Which one contains the assembly? Discuss these files as a group.
> Hint: take a look in `assembly_info.txt`  

Conce you're done, unload Flye before continuing to the next step.
```
module unload flye
```

## 4. Assembly quality control

We will use the tool [Quast](https://quast.sourceforge.net/docs/manual.html#sec2.1) to assess the quality of our genome assembly.

First, lets load the module:
```
module load quast
```

Running Quast:
```
# go back to your bootcamp directory
cd ~/bootcamp2024
mkdir quast

time quast flye/assembly.fasta --nanopore wWil.merged.rmdup.fastq.gz -t 1 -o quast --circos --k-mer-stats --glimmer --conserved-genes-finding --rna-finding --est-ref-size 1200000
```
> Quast took me about 20 minutes to run on 1 thread.  

Take some time to research the metrics and figures that QUAST produces, and discuss as a group. Which ones are informative about the quality of our assembly?

- [Quast Github](https://github.com/ablab/quast)
- [Quast Manual](https://quast.sourceforge.net/docs/manual.html#sec2.1)

I would reccommend downloading the quast output to your personal computer, so you can open all the figures it produces. To do this, open a new terminal window (on your personal computer, NOT on hummingbird) and run the following command (replace `{username}` with yours)

```
scp -r {username}@hb.ucsc.edu:/hb/home/{username}/bootcamp2024/quast/ .
```

What do the metrics and plots output by Quast tell us about the quality and completeness of our assembly? Do we have enough information to say whether our assembly is "good"?

## 5. Independent project and presentation

For the rest of bootcamp, your task is to find an interesting analysis to do with our _Wolbachia_ data. You may use the assembly, the sequencing reads, or both. This is **purposefully open-ended**, to give you practice with developing your own question or hypothesis, figuring out the research steps necessary to answer it, executing those steps, and presenting your work to others.  

We DO NOT expect everyone to come up with incredible groundbreaking results. The **worst thing you could do** would be to give up and not present anything, just because you couldn't get an analysis to work. Share your project idea, what you tried, what worked and what didn't, and what you learned from the project if you aren't able to get results for this independent portion.

To get you started, we've come up with some project ideas you may use for the independent portion, but coming up with your own idea is highly encouraged! Follow your interests.

#### Project ideas:

- Find additional assembly tools and run them on our data. Compare their quality against our Flye assembly. Which assembly tool produces the best quality assembly?
- Implement an algorithm to walk along the repeat graph produced by Flye `assembly_graph.gfa` and produce an assembly sequence. Compare your assembly to the one Flye produces.
- Characterize the repetitive elements in our assembly (Hint: RepeatMasker)
- Build a phylogeny with our _Wolbachia_ assembly and other species (Hint: USHER)
- Comparative genomics: ([Mauve](https://darlinglab.org/mauve/mauve.html), [Mummer](https://mummer.sourceforge.net)) Are there interesting variations (rearrangements, changes in functional elements) between our assembly and other relevant datasets? (Hint: compare to the wRi assembly available on NCBI)
- Take the repeat graph produced by Flye and visualize it in [Bandage](https://github.com/rrwick/Bandage). What does this visualization show you about the repeat structure and quality of the assembly?
- Present an in depth dive into QUAST performance metrics. Generate informative plots about the quality of our assemblies. Can you find any other tools to evaluate the quality of our assembly?

Finally, there are many bioinformatic tools already installed on hummingbird, which you can view by running `module avail`. We expect most tools you might consider using for your independent project to already be available. However, if there's a tool you're interested in that is not on hummingbird, let us know and we can help you with installing it.

Looking forward to seeing the projects you all come up with!

