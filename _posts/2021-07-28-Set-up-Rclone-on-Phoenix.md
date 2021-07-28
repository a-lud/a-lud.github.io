---
title: Using RClone on Phoenix for transferring data
layout: post
post-image: "https://forum.rclone.org/uploads/default/original/2X/0/0f430bb0f2f7a9ed020a9e93c89f4d332adcac4b.gif"
description: A simple guide to setting up RClone on Phoenix using Conda
tags:
  - Phoenix
  - Guide
  - HPC
  - Conda
---

Often we want to transfer data from a cluster to a cloud storage location so it can be shared
with someone. Here, I walk through how to set up RClone on Phoenix to enable simple data transfer
from Phoenix to Box.

---

# Introduction

A tricky aspect of working on a compute cluster is sharing the data that we generate. Often we have
to download the data manually, then upload it to a cloud storage location which is accessible to all
parties that are interested. This might be ok for small datasets (e.g. a few files), but becomes
increasingly tedious when files are many (thousands) or large. This document is going to introduce
[`Rclone`](https://rclone.org/), a command line program to manage files on cloud storage.

## RClone

As stated above, `Rclone` is a program that provides a command-line-interface (CLI) to managing
files in the cloud. It has support for [over 40 cloud storage products](https://rclone.org/#providers)
and is totally open source. It has equivalents of standard command-line commands such as `mv`,
`cp`, `mount` and a whole lot more, meaning it should be familiar to anyone who has some experience
using `UNIX` systems. It also provides some nice data-checks when copying/moving files, preserving
timestamps whilst also verifying checksums at all times (i.e. ensuring files are identical between
the systems).

The take-away should be that the software has many features, works across a range of could storage
products and should be relatively easy to use.

## Configuring Rclone for use on Phoenix with 'Box'

Below I'll outline the steps that are needed to configure `Rclone` with Adelaide University's
Phoenix HPC. I'll specifically demonstrate how to run the configuration with `Box`, however
the setup will be pretty similar regardless of the cloud product you use.

### Step 1: Autherisation password on Box website

The first step is to create an authorisation password on the `Box` website. The steps are:

1. Click on the circle with your initials at the top right of the Box home scree
2. Select `Account settings` from the drop-down menu
3. Select the `Account` tab (should be on this by default)
4. Scroll down to the section titled `Authentication`. It should say that you need to create a password
5. Create a password of your choosing, following the prompts

That's all that needs to be done for this step.

### Step 2: Install Rclone on Phoenix AND locally

Next, we need to install `Rclone` to Phoenix. I'm assuming you know how to log in to Phoenix, however
if you don't, do the following:

First, open your terminal (on Mac this is terminal, on windows I have no idea).

Next, log in to Phoenix using the following command:

```{bash}
$ ssh phoenix-login1.adelaide.edu.au
$ ...
$ Password: <type password here>
```

When you run the `ssh` command above, you'll be met with a whole lot of text. At the bottom of it
all it should ask for your password. Type your university password here. **NOTE: No characters will**
**show up as you type, but they will be there!**

You now should be on Phoenix. Now we'll install the software using `Conda`. At the command-line, run
the following:

```{bash}
$ conda create -n RCLONE_env -c conda-forge rclone
```

This will create a `conda` environment called `RCLONE_env` which will house the software `Rclone`.
Any time we want to use `Rclone`, we'll have to activate the environment. Once the environment has
been created, activate it using the follwoing command:

```{bash}
$ conda activate RCLONE_env
```

Your command-line prompt should look something like the following:

```{bash}
(RCLONE_env) a1234567@l01 ~ $
```

#### Install Rclone locally

You'll also need to install `Rclone` on your own system, as one of the set-up steps needs to be run
on a computer with internet access. Do this either via `conda` or through a local install.

### Step 3: Configuring Rclone to work with Box

Now that we have `Rclone` installed, we need to configure it so it has access to our `Box` account.
This will seem daunting, but is relatively straight-forward.

The first step is to run the command (from the active environment!) below:

```{bash}
$ rclone config
```

This will bring up some text with some options. It should look similar to below:

```{bash}
No remotes found - make a new one
n) New remote
s) Set configuration password
q) Quit config
n/s/q> n
```

We want to create a new remote, so type `n` as I have done in the example above, and press enter.

Next, we have to choose the type of cloud storage that we want to configure, in this case `Box`. In
my session, `Box` was number _6_. It may be different in yours. At any rate, you should be met with
a list of options with a corresponding number. Choose whichever number corresponds to `Box` and
press enter.

```{bash}
Type of storage to configure.
Enter a string value. Press Enter for the default ("").
Choose a number from below, or type in your own value
 1 / 1Fichier
   \ "fichier"
 2 / Alias for an existing remote
   \ "alias"
 3 / Amazon Drive
   \ "amazon cloud drive"
 4 / Amazon S3 Compliant Storage Providers including AWS, Alibaba, Ceph, Digital Ocean, Dreamhost, IBM COS, Minio, and Tencent COS
   \ "s3"
 5 / Backblaze B2
   \ "b2"
 6 / Box
   \ "box"
...
38 / Yandex Disk
   \ "yandex"
39 / Zoho
   \ "zoho"
40 / http Connection
   \ "http"
41 / premiumize.me
   \ "premiumizeme"
42 / seafile
   \ "seafile"
Storage> 6
```

The next few prompts can all be set to the default value. For the options shown below, just press
enter:

```{bash}
** See help for box backend at: https://rclone.org/box/ **

OAuth Client Id
Leave blank normally.
Enter a string value. Press Enter for the default ("").
client_id>
OAuth Client Secret
Leave blank normally.
Enter a string value. Press Enter for the default ("").
client_secret>
Box App config.json location
Leave blank normally.

Leading `~` will be expanded in the file name as will environment variables such as `${RCLONE_CONFIG_DIR}`.

Enter a string value. Press Enter for the default ("").
box_config_file>
Box App Primary Access Token
Leave blank normally.
Enter a string value. Press Enter for the default ("").
access_token>

Enter a string value. Press Enter for the default ("user").
Choose a number from below, or type in your own value
 1 / Rclone should act on behalf of a user
   \ "user"
 2 / Rclone should act on behalf of a service account
   \ "enterprise"
box_sub_type>
```

Stop when you reach the prompt below:

```{bash}
Edit advanced config? (y/n)
y) Yes
n) No (default)
y/n>
```

Here, we do want to edit the `advanced config`. We're not really going to change any of the
defaults, but there is one key thing we do need to change.

Continue by selection '`y`' to edit the config.

```{bash}
Edit advanced config? (y/n)
y) Yes
n) No (default)
y/n> y
```

Some more prompts will come up; set them all to default (see below):

```{bash}
OAuth Access Token as a JSON blob.
Enter a string value. Press Enter for the default ("").
token>
Auth server URL.
Leave blank to use the provider defaults.
Enter a string value. Press Enter for the default ("").
auth_url>
Token server url.
Leave blank to use the provider defaults.
Enter a string value. Press Enter for the default ("").
token_url>
Fill in for rclone to use a non root folder as its starting point.
Enter a string value. Press Enter for the default ("0").
root_folder_id>
Cutoff for switching to multipart upload (>= 50MB).
Enter a size with suffix k,M,G,T. Press Enter for the default ("50M").
upload_cutoff>
Max number of times to try committing a multipart file.
Enter a signed integer. Press Enter for the default ("100").
commit_retries>
This sets the encoding for the backend.

See: the [encoding section in the overview](/overview/#encoding) for more info.
Enter a encoder.MultiEncoder value. Press Enter for the default ("Slash,BackSlash,Del,Ctl,RightSpace,InvalidUtf8,Dot").
encoding>
```

Once you reach a prompt about the `Remote config`, stop:

```{bash}
Remote config
Use auto config?
 * Say Y if not sure
 * Say N if you are working on a remote or headless machine
y) Yes (default)
n) No
y/n>
```

Here, we do **NOT** want to use the auto-config. We want to select `NO` as Phoenix does not have
internet access (in the traditional sense). To authorise remote cloud storage, `Rclone` will open
a webpage with an authorisation button. Phoenix will not be able to do this if you select `y`.

```{bash}
Remote config
Use auto config?
 * Say Y if not sure
 * Say N if you are working on a remote or headless machine
y) Yes (default)
n) No
y/n> n
```

You'll now be greeted by the following:

```{bash}
For this to work, you will need rclone available on a machine that has
a web browser available.

For more help and alternate methods see: https://rclone.org/remote_setup/

Execute the following on the machine with the web browser (same rclone
version recommended):

	rclone authorize "box"

Then paste the result below:
result>
```

Copy and paste `rclone authorize "box"` into the terminal on **YOUR PERSONAL SYSTEM**.

```{bash}
~ via 🅒 base <-- NOTICE THIS IS DIFFERENT (MY LOCAL COMPUTER)
➜ rclone authorize "box"
```

This should open up a web browser wich will ask you to `Grant access to Box`. Once you authorise
this, an authorisation key will be generated in your terminal. Copy this key and paste it
into the terminal **ON PHOENIX**

```{bash}
Then paste the result below:
result> --> PASTE KEY HERE FROM LOCAL MACHINE <--
```

This should complete the set up. You might be asked a couple of extra things, just use the default
values. `Rclone` will then take you back to a page that looks like the following:

```{bash}
Name                 Type
====                 ====
Box                  box

e) Edit existing remote
n) New remote
d) Delete remote
r) Rename remote
c) Copy remote
s) Set configuration password
q) Quit config
e/n/d/r/c/s/q>
```

If `Box` is under the Name column and `box` is under the Type column, then you're all good and can
exit (select '`q`'):

```{bash}
Name                 Type
====                 ====
Box                  box

e) Edit existing remote
n) New remote
d) Delete remote
r) Rename remote
c) Copy remote
s) Set configuration password
q) Quit config
e/n/d/r/c/s/q> q
```

### Step 4: Test it all works

To test if the configuration has been successful, run the following command at the command-line:

```{bash}
$ rclone lsd Box:
```

This should return all directories at the top level of your `Box` repository. For example, for me
this returns:

```{bash}
(RCLONE_env) a1645424@l01 ~ $ rclone lsd Box:
          -1 2020-10-15 12:34:30        -1 00_PhD_papers_data
          -1 2019-02-12 11:13:37        -1 1901_Echidnaproject
          -1 2020-01-29 10:35:12        -1 1903_Ray_lungfish
          -1 2019-05-02 13:43:56        -1 1905_variants
          -1 2019-06-12 08:02:57        -1 1906_James_Alastair
          -1 2019-07-01 12:32:11        -1 1907_CODEML_checking
          -1 2019-11-13 16:09:21        -1 1907_James_Alastair
          -1 2019-07-22 14:31:47        -1 1907_codeml
          -1 2019-10-10 08:06:47        -1 1908_Manuscript
          -1 2021-05-12 07:43:58        -1 1910_Transcriptomes
          -1 2020-02-07 14:04:48        -1 2002_James_Alastair
          ...
```

### Step 5: Copying files

Copying files to and from `Box` follows the same principals as copying files to a server. The copy
command is:

`$ rclone copy source destination`

When copying files _from_ Phoenix _to_ `Box`, you'll want a command that looks something liek:

```{bash}
(RCLONE_env) a1234567@l01 $ rclone copy -p ${USER}/dir/*.txt Box:/location/in/Box/cloud
```

The above will copy all text files from `${USER}/dir` to a directory in your `Box` repository.
The `Box` destination directory will be created if it does not exist.

Note that we prefixed the `Box` path with:

```{bash}
Box:
```

This tells `Rclone` that we want to copy files to the `Box` remote that we set up. If during set-up
we named our remote `Box-2-eletricboogaloo`, we would change our command to the following:

```{bash}
(RCLONE_env) a1234567@l01 $ rclone copy -p ${USER}/dir/*.txt Box-2-eletricboogaloo:...
```

To copy files from `Box` to Phoenix, simply reverse the order:

```{bash}
(RCLONE_env) a1234567@l01 $ rclone copy -p Box:/some/dir/*.txt ${USER}/dir/output
```

That will copy all text files from `/some/dir/` on `Box` to `${USER/dir/output` on Phoenix.
