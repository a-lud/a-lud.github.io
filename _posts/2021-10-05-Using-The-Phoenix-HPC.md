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

```bash
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

```bash
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

```bash
$ man rsync        # Manual page
$ rsync --help     # Help page
```

### Copying Data to Phoenix

Below is an example command of copy data from our local machine to our `HPCFS` directory on Phoenix.

```bash
$ rsync -avP file.txt a1234567@phoenix-login1.adelaide.edu.au:/hpcfs/users/a123456/directory
```

This would copy the file `file.txt` to the directory `directory` in our `HPCFS` directory on Phoenix.

### Copying Multiple Files to Phoenix

To copy more than one file we can either list them all in the command, or use `globbing` if they
share a common pattern. Below I've written an example of copying all files in a directory that
have the file extension `.txt`.

```bash
$ rsync -avP *.txt a1234567@phoenix-login1.adelaide.edu.au:/hpcfs/users/a123456/directory
```

### Copying Files From Phoenix

The order of the `rsync` arguments is always

```bash
$ rsync -arguments item-to-copy destination-to-copy
```

As such, for getting data off Phoenix, we just need to reverse the order if we're on our local
system.

```bash
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

```bash
$ module list
```

This list could be quite long, but should detail every pre-installed bit of software that is
available.

If you see a bit of software that you want to use in a script, you simply need to include the
following in your code.

```bash
$ module load FastQC/0.11.7
```

Having that code in your `SLURM` submission script would load the `FastQC/0.11.7` module in your
jobs environment, giving your script access to the `FastQC` software.

You can load as many modules as you want, with the added benefit of the `module` system resolving
any software conflicts if it can. However, there can be incompatibilities meaning some software
modules will not be compatible with others. In these instances, you will receive an error.

Therefore, it can be beneficial to try loading the software modules at the terminal on the head node,
to check if they all play nice together.

### Conda Environments

Conda is an open source package and environment management system. If we set up Conda on Phoenix,
we can pretty much install whatever software we want on to Phoenix without requiring root permissions.
Further, it gets us away from having fixed versions of software, like with _modules_, as we can
manage our own software installations however we please. It also handles the installation for us,
as it installs from a recipe. This means it'll handle all the installation steps for us and should
_hopefully_ error early and informatively if something is incompatible. Further, if you're up for it,
Conda environments play really nicely with many workflow languages, which makes developing analysis
pipelines considerably easier.

#### Check Conda is working

The University of Adelaide HPC wiki has a really nice [guide][conda] relating to getting started
with Conda. Here, I'll show the main commands that are most important when it comes to getting going.

First, we need to check that the `conda` executable is in our path. At the command line run the
following:

```bash
$ conda -h
```

If `conda` is not installed, you'll likely get an error saying that the system can't find any
software called `conda`. In that case, run the following command:

```bash
$ module load Anaconda3/2020.07
```

I'm pretty certain that `conda` is now in everyone's path by default, so you **shouldn't** need to
run the module command above.

#### Configure Conda

Now that we know `conda` is working, we need to configure it a little bit. What we're first going to
do is specify **two** directories - a _packages_ directory and a _environments_ directory. At the
command line, run the following commands to configure each:

```bash
$ conda config --prepend pkgs_dirs /hpcfs/users/$USER/myconda/pkgs
$ conda config --prepend envs_dirs /hpcfs/users/$USER/myconda/envs
```

Those two commands have just set the respective paths to where packages and environments will be
installed - i.e. these are the destinations where software will be installed and where environments
will be housed.

Next, we will likely want to add a few `channels` to our configuration. Channels are essentially where
`conda` checks when it tries to install some software. Much like we might look in a few standard places
when we lose something, `conda` will check some 'standard' channels for a package when trying to
install something. We can add to the places `conda` will look by adding extra channels that we know
are useful. To do this, use the following command:

```bash
$ conda config --add channels <channelName>
```

Where `<channelName>` is your desired channel. Common channels to add include:

- conda-forge
- bioconda
- anaconda

With those three channels, you can pretty much guarantee you'll be able to find most common software.

#### Installing Software and Creating Environments

Now that we've configure `conda`, we can install software and create environments. Before we do that,
I'll first break down what an environment is.

If you tried to install a piece of software from scratch on Phoenix, you'd probably be met with a lot
of permission errors. This is because as a **user** you do not have permission to read and write from
certain locations on the computer. As such, this makes installing software difficult.

This is where `conda` environments come in. A `conda` envrionment is essentially a self-contained
bubble where we can essentially have all the permissions we could ever need to install anything we
want. First we need to create an environment:

```bash
$ conda create -n EXAMPLE_ENV
```

The above command would create a simple environment with the default packages `conda` installs.
Once we've created an environment, we need to activate it:

```bash
$ conda activate EXAMPLE_ENV
(EXAMPLE_ENV) $
```

What you'll notice once you've activated an environment is the `prompt` changing. It'll now show
which environment is active (see braces in code chunk above). Whilst active, we can install watever
software we want using conda or other package managers (e.g. `pip` but I'm not going into that here).
For example, we might install a text reader called `bat`:

```bash
$ conda install -c conda-forge bat
```

In the above command I've called the `install` argument of `conda` and specified the channel (`-c`)
I want to use (conda-forge). I then tell `conda` the software I want to install. That will install
the software `bat` into the environment `EXAMPLE_ENV`, meaning whenever I activate `EXAMPLE_ENV`
I will be able ot use `bat`.

We can deactivate conda environments like so:

```bash
(EXAMPLE_ENV) $ conda deactivate
$ ...
```

After deactivating an environment, you'll notice that the prompt goes back to normal. This means that
the environment is no longer active and any software that is installed in that environment is no longer
available.

#### How are Conda Environments Useful?

So how might these `conda` environments be useful? Well, in all sorts of ways.

Consider, for example, you have some software that needs `python=2.7` and some other software for the
same analysis that needs `python >= 3.X`. Further, imagine that we don't have many access permissions
to our computer, as is the situation on Phoenix. Without environments, we'd have to install everything
from source _AND_ we'd have to juggle python versions whilst doing this.

With `conda` environments, we can simply create two environments and install the respective software
in each. We don't have to worry about permissions or python versions, beyond specifying which `python`
version each environment should have.

This might looks like the following in a mock `SLURM` script:

```bash
#!/usr/bin/env bash
#SBATCH --job-name=random-job
#SBATCH -p batch
#SBATCH -N 1
#SBATCH -n 1
#SBATCH -c 12
#SBATCH --ntasks-per-core=1
#SBATCH --time=24:00:00
#SBATCH --mem 30GB
#SBATCH -o /home/a1234567/slurm/%x_%j.log
#SBATCH --mail-type=FAIL
#SBATCH --mail-user=first.last@adelaide.edu.au

# Activate Python 2 environment
conda activate PY2.7

# <code to do stuff goes here>

conda deactivate


# Activate Python 3 environment
conda activate PY3.6

# <code to do stuff goes here>

conda deactivate
```

In the above example, we've been able to run code requiring two separate python versions super
simply by simply activating and deactivating pre-made `conda` environments.

## Submitting Jobs to Phoenix

Now that we know how to interact with Phoenix, the final step is to submit a job. As mentioned above,
Phoenix uses `SLURM` to manage job submissions, which means the workflow from writing a script to
it running on the HPC looks like the following:

1. You write a script with the `SLURM` header information in it
2. You submit your script to the scheduler
3. `SLURM` dictates where your job is in the priority list
4. Your job will run on the cluster when it's time comes
5. You job will finish up and you'll be notified if anything has gone wrong

Let's get into a few of the points above

### SLURM header

Every job submitted to `SLURM` needs a `SLURM` header. It looks like the following:

```bash
#!/usr/bin/env bash
#SBATCH --job-name=random-job                   # Job name
#SBATCH -p batch                                # Cluster to submit to
#SBATCH -N 1                                    # Number of nodes to request
#SBATCH -n 1                                    # Number of jobs per node
#SBATCH -c 12                                   # Number of cores per job
#SBATCH --ntasks-per-core=1                     # This is to prevent multithreading
#SBATCH --time=24:00:00                         # Time for the job
#SBATCH --mem 30GB                              # Memory for the job
#SBATCH -o /home/a1234567/slurm/%x_%j.log       # Path for log file
#SBATCH --mail-type=FAIL                        # Email if the job fails
#SBATCH --mail-user=first.last@adelaide.edu.au  # Email to send fail/completion email
```

Essentially, it houses the meta-data that `SLURM` will use when allocating resources for your job.
In the above example I've commented what each line does. See the `SLURM` documentation for extra
parameters that can be set. The arguments above are what I typically set for my own jobs.

After the header you can put whatever code you want. I generally will just put any bash code after
the header rather than in a separate file, but feel free to write a separate script that gets called
from the submission script if you so please.

```bash
#!/usr/bin/env bash
#SBATCH --job-name=random-job
#SBATCH -p batch
#SBATCH -N 1
#SBATCH -n 1
#SBATCH -c 12
#SBATCH --ntasks-per-core=1
#SBATCH --time=24:00:00
#SBATCH --mem 30GB
#SBATCH -o /home/a1234567/slurm/%x_%j.log
#SBATCH --mail-type=FAIL
#SBATCH --mail-user=first.last@adelaide.edu.au

samtools view \
  -b \
  -h \
  -q 20 \
  ${BAM}/input.bam

# etc...
```

### Submitting Jobs

To submit a script like the one above to the cluster, simply call the following command:

```bash
$ sbatch name-of-script.sh
```

### Monitoring Jobs and Cancelling Jobs

To monitor how your job is tracking, you can use the command `squeue`. Essentially you just need to
run the following:

```bash
$ squeue -u a1234567
```

This will provide you with some information about how long the job has been running and how
much wall time was requested.

To cancel a job, simply run the command:

```bash
$ scancel -u jobid
```

Where `jobid` is replaced with the actual job identifier obtained from `squeue`.

[phoenix_wiki]: https://wiki.adelaide.edu.au/hpc/index.php/Main_Page
[phoenix_allocations]: https://wiki.adelaide.edu.au/hpc/Phoenix/Hardware
[phoenix_login]: https://wiki.adelaide.edu.au/hpc/Connecting_to_Phoenix
[conda]: https://wiki.adelaide.edu.au/hpc/Anaconda
