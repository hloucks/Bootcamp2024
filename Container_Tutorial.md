# BME 2024 Using and creating Containers 

Tutorial loosely based on https://docs.sylabs.io/guides/2.6/user-guide/quick_start.html

## Environments - Conda 


First let's allocate some resources for our session and load singularity 
```
salloc -N 1 -n 1 -p 128x24 -t 03:00:00 

module load singularity-ce/singularity-ce.4.1.4
```


## Pulling and building an existing image with Singularity 

```
singularity pull shub://vsoch/hello-world 

singularity build hello-world.simg shub://vsoch/hello-world

```

Now we can interact with our container either via an interactive session with`singularity shell` 

```
singularity shell hello-world.simg

# try running the following commands 
whoami
id

# to leave your shell you can just use 
exit

```

We can also run container without first entering into it with shell 
```
singularity run hello-world.simg
```

You can also use run with containers directly from shub and docker

```
singularity run shub://GodloveD/lolcow
```

Here are a few more useful commands you can use to check out your containers 

```
singularity inspect hello-world.simg
```

If we want to just run a single command within our container we can use `singularity exec`

```
singularity exec hello-world.simg ls
```


### tricky things with containers 

Ensuring that you mounted directories correctly 
One of the most common issues when working with containers is not mounting your files correctly. This often means that you enter into your container and can't find the datafiles you wanted to work with. Singularity automatically will mount your execution directory, home directory, and /tmp, but you may want to mount other locations like this 

```
# first let's create or select a file that we want to mount
mkdir ~/testData

nano ~/testData/data.txt
# write some text into the file, it doesn't matter what 

# now we can mount that directory when we execute a command in our container 

singularity exec --bind ~/testData:/mnt hello-world.simg cat /mnt/data.txt

```
That exec command should show you the contents of data.txt  

### finding containers 

There are a number of container repositories that are compatible across different container platforms. Often when you need a container there is an existing one you can use online. Here are some places you can find containers  
https://biocontainers.pro/registry
https://quay.io/
https://hub.docker.com/

Now go find a container that has some software you are familiar with, pull the image, and see if you can view the software documentation with a `singularity exec` command 

It should look something like this 

```
singularity exec image.simg softwareName --help  
```

### creating custom containers (???)

9/2/24 - need to talk to Rion about how we can do this - it appears to require sudo

docker run into base images 

make changes (keep notes)


turn notes into a docker file 

build the container locally 

push container 

