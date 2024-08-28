# BME 2024 Bash Introduction 
#### slides https://docs.google.com/presentation/d/1hjIfozfQkjL4gj1eUtvBqgzpWERAkF8Uw43ToAQSxa8/edit#slide=id.g97a7885a08_0_0

## Getting started on hummingbird 
Log into hummingbird
```
ssh yourNetID@hb.ucsc.edu
```
input your password and you should be in your home directory. 

```
pwd 
/hb/home/yourNetID
```

## Bash basics 

### 1. Set up your working directory 

First let's make a directory and move into that directory 
```
mkdir BashIntro
cd BashIntro
```



### Creating and removing files 
Create an executable file 
```
touch myScript.sh
```
Let's edit the file with nano

```
nano myScript.sh 
```

Now you can write bash code that will print something when you execute the script, it could be as simple as  
```echo "Hello World"```

Now we can execute our script
```
bash myScript.sh
```

### Input/Output direction 

Directing input and outputs is an important ability in bash. Let's try directing the output of our script above into an output file:
```
bash myScript.sh > myOutput.txt
```

Now you can check what is in our output file 
```
cat myOutput.txt 
```

### Using History 

A very useful way of recalling commands you've used is by using the `history` command. By combining it with `grep`  you can easily find commmands you've used before 

```
history | grep myOutput.txt

   89  bash myScript.sh > myOutput.txt 
   90  cat myOutput.txt 
   91  history | grep myOutput.txt

```

Grep is an incredibly useful command that you can configure to do all types of searches. Definitely spend some time reading the documentation on the basic bash commands like grep if you aren't familiar with them already. 

### Searching 
Now let's practice searching for a file. 

change directories back to your home directory 
```
cd ..
```
now we are going to use the find command to search for the path of our file 
```
find . -name myOutput.txt
./bashintro/myOutput.txt
```
you can get the complete path by directing that output to the readlink function 
```
find . -name myOutput.txt | readlink -f stdin
/hb/home/ourNetID/stdin
```

### Using screens 

Often when you are running an interactive session you might want to be able to step away or switch over to another project you are working on. One way of doing this is by using screens

First let's create a screen and run some basic commands 
```
screen -S myScreen  

pwd

touch Screentest.txt

```

Now detach from your screen using Ctrl+A+D and then change directories by typing `cd`

Now you should be able to reattach using 
```
screen -r myScreen
```
And voila, you history should be there and you should still be in the same location you were when you left the screen. 

To exit and kill the screen you can simply type `exit`while in the screen. 

Screens are useful if you need to let something run, or sometimes if you are using the cluster with an inconsistent network connection. This way if your PC or internet die your session will still be intact (barring a server reboot).

Make sure to clear any screens you aren't using. To check what screens you have in place you can use `screen -ls`

Tmux is also a good option similar to screens with more functionality. You can find more information [here](https://man7.org/linux/man-pages/man1/tmux.1.html)


### BashRC file familiarity 

The file ~/.bashrc is automatically executed when the shell is launched.  It contains some aliases and environment variables, and you can customize it to add your own.


You can take a look at the contents of your bashrc file using `cat ~/.bashrc`

Running `source ~/.bashrc` will reload bash with any current changes.

Often when you are trying to install software you will want to add paths to your bashrc file so you can run it more easily. You don't need to run this but you will often see this command when following software installs.
```
export PATH=$PATH:<path/to/dir>
```
You can do a lot of customizing of your bashrc file. For instance if you have commands that you use a lot you can set up aliases for them in your bashrc file 
`alias ll="ls -l"`

Here is more [information on bashrc file configuration](https://phoenixnap.com/kb/bashrc)

## Slurm basics 

Before we run any scripts it is a good idea to check in on what the cluster's load looks like 
We can use the `htop` command for this. To exit the htop interface you can just hit "q". 

The hummingbird cluster uses a job scheduler called Slurm. Slurm allows us to specify how much space we are going to take on the cluster so that it can coordinate our jobs and prevent crashes/overloads. Being familiar with slurm and other schedulers is an important skillset if you're going to be working with any large clusters to make sure that your jobs run correctly and that you are able to scale appropriately (without any angry emails from your sysadmin). We'll practice using slurm for most of our tutorials so definitely read the documentation [here](https://slurm.schedmd.com/quickstart.html)

You can check on the status of the cluster with `sinfo` which will look something like this 
```
[hloucks@hb-login ~]$ sinfo
PARTITION AVAIL  TIMELIMIT  NODES  STATE NODELIST
build        up   infinite      1   idle hbnode-05
128x24*      up   infinite      8    mix hbnode-[06,08,12,17-21]
128x24*      up   infinite     10  alloc hbnode-[07,09-11,13-16,22-23]
256x44       up   infinite      1   idle hbnode-25
96x24gpu4    up   infinite      1   idle hbnode-24

```
This will give you an idea of how much of the server is in use and how long your jobs might need to wait in the queue. Later on we will practice working interactively for slurm but for now we will practice launching an job with sbatch. We are going to create a basic slurm script so you have a template for when you are running your jobs later in the tutorial. 

Here is an overview of the basic makeup of a slurm script from the [Hummingbird dox](https://hummingbird.ucsc.edu/documentation/creating-scripts-to-run-jobs/):

```
#!/bin/bash
#SBATCH --job-name=serial_job_test    # Job name
#SBATCH --mail-type=ALL               # Mail events (NONE, BEGIN, END, FAIL, ALL)
#SBATCH --mail-user=   # Where to send mail	
#SBATCH --ntasks=1                    # Run on a single CPU
#SBATCH --time=00:05:00               # Time limit hrs:min:sec (you may not want this)
#SBATCH --output=   # Standard output and error log
#SBATCH --mem=250M                    # Allocated 250 megabytes of memory for the job.

#Load necessary modules (if needed)
#module load module_name

#Your job commands go here
#For example:
#python my_script.py

#Optionally, you can include cleanup commands here (e.g., after the job finishes)
#For example:
#rm some_temp_file.txt

```

Make your own edits to the above template to run your script we created above and launch using 
```
sbach mySlurm.slurm
```

Here is a [slurm cheatsheet](https://arc.umich.edu/wp-content/uploads/sites/4/2020/05/Great-Lakes-Cheat-Sheet.pdf) that I reference pretty frequently.


Once you have complete a slurm script let one of the instructors know so we can double check that it looks good. Then you're done! 