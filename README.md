# NYU HPC Tutorial

## Contents
-   [IMPORTANT](#important)
-   [Log in](#log-in-to-nyu-hpc-greene)
-   [Filesystem](#log-in-to-nyu-hpc-greene)
-   [Upload/Transfer Files](#uploadtransfer-files)
-   [Environment Setup](#environment-setup)
-   [Submit a Job](#submit-a-job)

## IMPORTANT
NYU HPC resources are only available to NYU members, including students, staffs, and faculty. You need an NYU NetID to access NYU HPC resources. For more information, please visit [NYU HPC Accounts and Eligibility](https://www.nyu.edu/life/information-technology/research-computing-services/high-performance-computing/high-performance-computing-nyu-it/hpc-accounts-and-eligibility.html).

Also, NYU HPC requires NYU internet. If you try to access NYU HPC from outside of NYU internet, you need to install and run NYU VPN. I also highly recommend you to use NYU VPN to access NYU HPC resources for stable connection eventhough you have connected NYU internet. For information of how to install and use NYU VPN, please visit [NYU Virtual Private Network (VPN)](https://www.nyu.edu/life/information-technology/infrastructure/network-services/vpn.html)

***Do not run compute-heavy jobs on log-in nodes! HPC admins (and other HPC users) will be very upset and you might get into trouble!***


## Log in to NYU HPC Greene
### Use VS-Code
For VS-Code user, please follow the steps:
-   Enable `remote ssh` extension
-   Use `remote ssh` extension to connect to `<net-id>@greene.hpc.nyu.edu`
-   Go to your folder by using `cd` command

### Use Terminal
-   ssh to gateway with the command `ssh <net-id>@gw.hpc.nyu.edu`
-   then ssh to greene with the command `ssh <net-id>@greene.hpc.nyu.edu`

After you successfully log in to the NYU HPC Greene, your terminal should have such symbol:
```
| \ | \ \ / / | | | | | | |  _ \ / ___|
|  \| |\ V /| | | | | |_| | |_) | |
| |\  | | | | |_| | |  _  |  __/| |___
|_| \_| |_|  \___/  |_| |_|_|    \____|

  ____
 / ___|_ __ ___  ___ _ __   ___
| |  _| '__/ _ \/ _ \ '_ \ / _ \
| |_| | | |  __/  __/ | | |  __/
 \____|_|  \___|\___|_| |_|\___|
```
If you keep getting error, please keep re-try to connect with it. This error may caused by NYU VPN.


## Looking Around the Filesystem

|  |	env  var |	what for |	flushed |	quota|
| --- | --- | --- | --- | --- | 
| /archive	| \$ARCHIVE | long term storage	| NO	| 2TB/20K inodes |
| /home	| \$HOME	| probably nothing	| NO	| 50GB/30K inodes |
| /scratch |	\$SCRATCH | experiments/stuff	| YES (60 days)	| 5TB/1M inodes |

Check quota by `myquota`. 

For most situations:
-   You probably won't be using `/archive`.  
-   You will store very very few things (maybe just a few lines of environment-related code) on `/home`.


Where to store your data
-   `/scratch/[netid]`(Highly recommend)

-   How to get on Greene login node? `ssh [netid]@greene.hpc.nyu.edu` (see above).

## Upload/Transfer Files
From A to B (you must be on NYU network; VPN also okay)
-   On A, do `scp [optional flags] [file-path] [netid]@greene.hpc.nyu.edu:[greene-destination-path]`

From B to A (you must be on NYU network)
-   On A, do `scp [optional flags] [netid]@greene.hpc.nyu.edu:[file-path] [local-destination-path]`

From B to C
-   On C, do `scp [optional flags] greene-dtn:[file-path] [gcp-destination-path]`

From C to B
-   On C, do `scp [optional flags] [file-path] greene-dtn:[greene-destination-path]`

From A to C
-   A -> B -> C

From C to A
-   C -> B -> A

## Environment Setup
Please use singularity with overlay file to setup the Miniconda environment. 

### What is Singularity?
Singularity is a free, cross-platform and open-source program that creates and executes containers on the HPC clusters. Containers are streamlined, virtualized environments for specific programs or packages. Singularity is an industry standard tool to utilize containers in HPC environments. Containers allow for the support of highly specific environments and further increase scientific reproducibility and portability. Using Singularity containers, researchers can work in the reproducible containerized environments of their choice can easily tailor them to their needs.

### Pre-install Warning
If you have initialized Conda in your base environment (your prompt on Greene may show something like `(base) [NETID@log-1 ~]$)` then you must first comment out or remove this portion of your `~/.bashrc file`:
```sh
# >>> conda initialize >>>
# !! Contents within this block are managed by 'conda init' !!
__conda_setup="$('/share/apps/anaconda3/2020.07/bin/conda' 'shell.bash' 'hook' 2> /dev/null)"
if [ $? -eq 0 ]; then
    eval "$__conda_setup"
else
    if [ -f "/share/apps/anaconda3/2020.07/etc/profile.d/conda.sh" ]; then
        . "/share/apps/anaconda3/2020.07/etc/profile.d/conda.sh"
    else
        export PATH="/share/apps/anaconda3/2020.07/bin:$PATH"
    fi
fi
unset __conda_setup
# <<< conda initialize <<<
```

### Miniconda Environment PyTorch Example
#### You only need to do the following steps when you first time to use NYU HPC
Create a directory for the environment
```
$ mkdir /scratch/<NetID>/pytorch-example
$ cd /scratch/<NetID>/pytorch-example
```

Copy an appropriate gzipped overlay images from the overlay directory. You can browse available images to see available options
```
$ ls /scratch/work/public/overlay-fs-ext3
```

In this example we use overlay-15GB-500K.ext3.gz as it has enough available storage for most conda environments. It has 15GB free space inside and is able to hold 500K files
```
$ cp -rp /scratch/work/public/overlay-fs-ext3/overlay-15GB-500K.ext3.gz .
$ gunzip overlay-15GB-500K.ext3.gz
```

Launch the appropriate Singularity container in read/write mode (with the :rw flag)
```
$ singularity exec --overlay overlay-15GB-500K.ext3:rw /scratch/work/public/singularity/cuda11.6.124-cudnn8.4.0.27-devel-ubuntu20.04.4.sif /bin/bash
```

Now, inside the container, download and install miniconda to /ext3/miniconda3
```
wget https://repo.continuum.io/miniconda/Miniconda3-latest-Linux-x86_64.sh
bash Miniconda3-latest-Linux-x86_64.sh -b -p /ext3/miniconda3
# rm Miniconda3-latest-Linux-x86_64.sh # if you don't need this file any longer
```

Next, create a wrapper script /ext3/env.sh

The wrapper script will activate your conda environment, to which you will be installing your packages and dependencies. The script should contain the following:
```
#!/bin/bash

source /ext3/miniconda3/etc/profile.d/conda.sh
export PATH=/ext3/miniconda3/bin:$PATH
export PYTHONPATH=/ext3/miniconda3/bin:$PATH
```

Activate your conda environment with the following:
```
source /ext3/env.sh
```

Now that your environment is activated, you can update and install packages
```
source /ext3/env.sh
```

#### You need to do the following steps every time when you use NYU HPC
First, start an interactive job with adequate compute and memory resources to install packages. The login nodes restrict memory to 2GB per user, which may cause some large packages to crash.
```
$ srun --cpus-per-task=2 --mem=10GB --time=04:00:00 --pty /bin/bash
```
If you wish to use GPU for your interactive job, please use
```
$ srun --mem=8GB --time=2:00:00 --gres=gpu:rtx8000:1 --pty /bin/bash
```

Then, active your environment:
```
$ singularity exec --overlay overlay-15GB-500K.ext3:rw /scratch/work/public/singularity/cuda11.6.124-cudnn8.4.0.27-devel-ubuntu20.04.4.sif /bin/bash
$ source /ext3/env.sh
```
For more information, please visit [NYU HPC: Singularity with Miniconda](https://sites.google.com/nyu.edu/nyu-hpc/hpc-systems/greene/software/singularity-with-miniconda).


## Submit a Job