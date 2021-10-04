---
title: How to use Adelaide Universities HPC
layout: post
post-image: "https://upload.wikimedia.org/wikipedia/commons/4/43/Phoenix-Fabelwesen.jpg"
description: A simple guide to working on the Phoenix HPC
tags:
  - Phoenix
  - Guide
  - HPC
  - Conda
  - Tutorial
---

At Adelaide University we have access to the Phoenix HPC. This system is perfect for large
compute jobs that exceed the capabilities of our own personal computers. I've put together
a little something to help first time users onto the system.

---

# The Phoenix High Performance Computer (HPC)

Phoenix is a large supercomputer at the University of Adelaide. The actual size of the system
changes over time as new compute resources are added, but you can visit [here][phoenix_allocations]
to see the detailed specifications if you're so interested (an have an Adelaide University account).

Phoenix is comprised of many smaller compute nodes that together make up the super-computer. What
we interface with when we log in is the _head node_, a separate node that is for user access and
**not** computation. From this head node, we can submit jobs to the cluster, which enter a queue
and eventually get executed. The Phoenix system uses the `SLURM` job scheduler, which handles job
submission and execution in a "fair" manner. Each job you submit will enter the queue with a
priority ranking, and it's SLURM's job to figure out when your job should be run.

All Phoenix users also have a resource allocation. Each user has _1TB_ of fast storage available
to them at `/hpcfs/users/a1234567` This storage is best suited for data analysis as it has
fast I/O, however it **is not backed up**. In addition to this, users also have
10GB of backed up storage in their `HOME` directory, however this has extremely slow I/O and should
not be used for storing data.

In essence, Phoenix is just a big server, which we interact with from a head node that we log in to
and submit jobs from. Below, I'll go through how to do all this.

## Official Phoenix Wiki + Notes

My notes here are a needs-know summary of the official [Phoenix-Wiki][phoenix_wiki]. Their
documentation should always be taken as gospel, with this just being a convenient helper.

Also, in the code blocks below, you will see some starting with `$`. **Do not copy the dollar-sign**.
It is there to represent the _bash-prompt_.

## Logging in to Phoenix

My preferred approach to access Phoenix is at the command-line using `SSH`. Once you've been given
access to Phoenix, you simply need run the following to log in (using your university credentials).

```{bash}
$ ssh a1234567@phoenix-login1.adelaide.edu.au
```

After pressign `Enter`, you'll be greeted with a page of text and the requirement to enter your
university password. Do that and you should see the prompt return. It should looks similar to the
following:

```{bash}
(base) a1645424@l01 ~ $
```

You're now on the head node of Phoenix in **your** `HOME` directory. The official documentation
for logging in can be found [here][phoenix_login], which may be useful for non Mac/Linux users.

## Changing your shell

The default shell on Phoenix is `Tcsh` and we want `Bash`. Run the following **once**.

```{bash}
$ rcshell -s /bin/bash
```

## Useful Phoenix Commands

Phoenix comes set-up with a few useful commands, which I'll detail below. Don't worry if you
don't understand what they do just yet, we'll cover most of them below.

- **rcdu**: This command tells you how your storage allocation is faring
- **rcquota**: This command gives you a summary of how many credits you've used from your allocation
- **rcstat**: This command takes a job-id and provides detailed information about how it ran/is
  running.

With these commands, it's easy to manage your workspace and jobs on the cluster.

## Phoenix Directory Structure

As outlined in the introduction, Phoenix has a couple of storage options for users.

The first is the `HOME` directory which has 10GB of space. This is suitable for scripts or valuable
documents that you want backed up. It is **not** suitable for large files or for data that is to
be analysed. If you exceed your 10GB quota, you will be restricted from submitting jobs.

The other main storage location is your `HPCFS` directory which should either be symlinked to your
`HOME` directory or accessible from `/hpcfs/users/a1234567`. This is your 1TB of storage for data
analysis. Similar to `HOME`, if you exceed your 1TB limit, you will not be able to submit any more
jobs until your get under it.

You can check your storage quotas by running the `rcdu` command.

If you have access to an `R:` drive, it's possible to have ITDS either mount it for your if it is
not already, or, give you access if it's already on the system.

## Transferring data to Phoenix

To use Phoenix we'll need data on the system. My preferred method for this is to use `rsync`. It's
a command line tool that is fast, simple and has some nice features - like checkpointing your copy
so you don't have to re-upload all the data again if the connection gets interrupted.

In the examples I provide below, I use the following `rsync` arguments:

- **-a** = archive
- **-v** = verbose
- **-P** = Progress

There are many, many more which can be accessed by calling the man page or help page.

```{bash}
$ man rsync        # Manual page
$ rsync --help     # Help page
```

### Copying Data to Phoenix

Below is an example command of copy data from our local machine to our `HPCFS` directory on Phoenix.

```{bash}
$ rsync -avP file.txt a1234567@phoenix-login1.adelaide.edu.au:/hpcfs/users/a123456/directory
```

This would copy the file `file.txt` to the directory `directory` in our `HPCFS` directory on Phoenix.

### Copying Multiple Files to Phoenix

To copy more than one file we can either list them all in the command, or use `globbing` if they
share a common pattern. Below I've written an example of copying all files in a directory that
have the file extension `.txt`.

```{bash}
$ rsync -avP *.txt a1234567@phoenix-login1.adelaide.edu.au:/hpcfs/users/a123456/directory
```

### Copying Files From Phoenix

The order of the `rsync` arguments is always

```{bash}
$ rsync -arguments item-to-copy destination-to-copy
```

As such, for getting data off Phoenix, we just need to reverse the order if we're on our local
system.

```{bash}
$ rsync -avP a1234567@phoenix-login1.adelaide.edu.au:/hpcfs/users/a123456/directory/\*.txt /local
```

Above is a command to copy text files from Phoenix to a directory called `local` on our own machine. Note that I've had to _escape_ the `*` symbol so that it gets interpreted as a regular expression
and not as a a literal asterisk symbol.

## Loading software on Phoenix

There are a few ways to load or download software we might need on Phoenix. Below I've outlines
three methods that you might require.

### Modules

Phoenix has a module system. You can think of these as pre-installed packages that you can load into
your current shell environment to access particular software. Run the following command to get a
list of all available.

```{bash}
$ module list
```

This list could be quite long, but should detail every pre-installed bit of software that is
available.

If you see a bit of software that you want to use in a script, you simply need to include the
following in your code.

```{bash}
$ module load FastQC/0.11.7
```

Having that code in your `SLURM` submission script would load the `FastQC/0.11.7` module in your
jobs environment, giving your script access to the `FastQC` software.

You can load as many modules as you want, with the added benefit of the `module` system resolving
any software conflicts if it can. However, there can be incompatibilities meaning some software
modules will not be compatible with others. In these instances, you will recieve an error.

Therefore, it can be beneficial to try loading the software modules at the terminal on the head node,
to check if they all play nice together.

### Conda Environments

< finish this>

<!-- Conda is a package management system

A really handy method for getting software is to create a conda environment. The Phoenix Wiki has a great guide on how to set up your Phoenix account to be able to create these environments: https://wiki.adelaide.edu.au/hpc/Anaconda

Conda environments are kind of like containers. They enable you to install a range of software locally without the hastle of you having to go through the manual process. It handles all of the nitty gritty, like permissions and install locations, so all you're left with is an environment that has the software you want.

You can create any number of environments, meaning it's often a good idea to create an environment for a specific analysis or bit of software that isn't available on Phoenix. You activate the environment at the beginning of a submission script on Phoenix, which makes all the software in the environment available, and then deactivate it at the end.

See the Wiki link above for examples of setting up your Phoenix account for conda and creating an environment with software.

**Local installation**

It's also possible to build software locally from source if it is not available on Phoenix as a module or not available as a conda install. For this I'd recommend creating your own software directory on your `FAST` drive and download software there.

This can get involved and cumbersome so I'd only do this as a last resort if the above two methods are not an option.

## Submitting jobs

Now that we've covered how to log on to Phoenix, copy data to Phoenix and get the software we need, we can look at running an analysis on Phoenix.

Let's say we have a script `task.sh` that we want to run. To submit this script to the resource management tool SLURM for job scheduling, we need to create a submission script. I've included a template of what I generally include in my submission scripts below. Let's say these go into a script `task_submission.sh`.

```
#!/bin/bash
#SBATCH --job-name=run_test_script
#SBATCH -p batch
#SBATCH -N 1
#SBATCH -n 1
#SBATCH -c 4
#SBATCH --time=02:00:00
#SBATCH --mem=8GB
#SBATCH -o /path/to/slurm/stdout_file/%x_%j.out
#SBATCH -e /path/to/slurm/stderr_file/%x_%j.err
#SBATCH --mail-type=END
#SBATCH --mail-type=FAIL
#SBATCH --mail-user=a1234567@adelaide.edu.au

## You can call a script
bash task.sh

## You can also write your code directly in the submission script
for i in {1..10}; do
    echo ${i}
done
```

These are some basic parameters which define the resources we're requesting of the Phoenix HPC. All are prefaced by the `#SBATCH` argument to indicate that they are parameters for the job scheduler. They are as follows:

- **#!/bin/bash** = shebang indicating this is a shell script
- **--job-name=run_test_script** = Name of the job we are running
- **-p batch** = The partition we are submitting the job to. Can be batch/cpu/skylake/skypool/highmem/etc...
- **-N 1** = Number of nodes we are requesting. Generally will be 1
- **-n 1** = Number of tasks per node we are requesting. Generally will be 1
- **-c 4** = Number of multi-threading cores to use for our task
- **--time=02:00:00** = Time in HH:MM:SS (max time 72hrs)
- **--mem=8GB** = Amount of memory for the job
- **-o /.../%x\_%j.out** = An output file that will have the job name (above) and job id. Records standard out
- **-e /.../%x\_%j.err** = An output file that will have the job name (above) and job id. Records standard error
- **--mail-type=END** = Email if the job ends
- **--mail-type=FAIL** = Email if the job fails
- **--mail-user=a1234567@adelaide.edu.au** = Email

Think about how you allocate resources. If you know you're doing a lightweight task then don't go requesting 3 days of wall-time and a tonne of memory. Getting your job to run on Phoenix is a balancing act between requesting parameters that are likely to get your job submitted quickly but also enough that they finish in a reasonable amount of time.

For example, if you asked for 1 day, 20 cores and 100Gb of memory, your job would sit in the queue for days and take a long time to being running. Once submitted it might run pretty quickly, but the excessive resources pushes the job down the queue meaning there is a huge submission delay.

On the other hand, if you rolled back to 8 cores and 20Gb, your job would likely fill in a resource gap and be submitted much faster. It might take a bit longer to run, but the shorter queue time and slightly longer run time would still have you coming out ahead compared to the overkill submission above.

Now we have a task script we want to run (`task.sh`) and have our submission script (`task_submission.sh`) that executes our task script. To submit the job to Phoenix we run the following command from the head node:

```
$ sbatch task_submission.sh
```

This will now queue our job and will run the code in `task.sh` with the resources we've requested in `task_submission.sh`

## Monitoring Jobs

It's possible to monitor the run time of jobs using the command `squeue -u a1234567`. This will provide you with a list of jobs that are currently submitted, what node they're running on and how long they've been running for. You can also check the state of a running job or a job that has finished running using the command `rcstat jobid` where _jobid_ is the numerical value assigned to a submitted job.

## Cancelling jobs

To cancel a job that is running, simply run the following code

```
$ scancel jobid
``` -->

[phoenix_wiki]: https://wiki.adelaide.edu.au/hpc/index.php/Main_Page
[phoenix_allocations]: https://wiki.adelaide.edu.au/hpc/Phoenix/Hardware
[phoenix_login]: https://wiki.adelaide.edu.au/hpc/Connecting_to_Phoenix
