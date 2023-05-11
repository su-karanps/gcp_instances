# Google Cloud Platform (GCP) Instances

This guide is meant to provide a concise resource for those doing machine learning (using PyTorch) or simulation work (in MATLAB) on GCP Compute Engine Instances. It also includes random Linux commands and tips/tricks that may come in handy. I must credit the [Stanford CS231n Tutorial Repository](https://github.com/cs231n/gcloud) for some of these links and instructions, which have helped me countless times with my own projects. 

(Last Updated on April 8, 2023)

# Table of Contents

1. [Setting Up a New Instance](#setting-up-a-new-instance)
2. [Using GCP for MATLAB](#using-gcp-for-matlab)
3. [Tips and Tricks](#tips-and-tricks)
   - [SCP Between Instances](#scp-between-instances)
   - [Create a Shared Directory](#create-a-shared-directory)
   - [Large Directories](#large-directories)
   - [Duplicate Instances](#duplicate-instances)
   - [Python or Pip Issues?](#python-or-pip-issues)

# Setting Up a New Instance

This guide assumes you have a [Google Cloud account](https://cloud.google.com/) with a sufficient GPU quota (once you have [upgraded your account](https://cloud.google.com/free/docs/free-cloud-features#how-to-upgrade), go to IAM Admin -> Quotas and request an increase) 

**Image**: keep the default Debian image. I found the 'GPU-optimized Debian 10 with CUDA 11.0' to be more tricky than helpful, unfortunately.

First, install your NVIDIA drivers and CUDA using this [handy script](https://cloud.google.com/compute/docs/gpus/install-drivers-gpu) from GCP:

```bash
curl https://raw.githubusercontent.com/GoogleCloudPlatform/compute-gpu-installation/main/linux/install_gpu_driver.py --output install_gpu_driver.py
sudo python3 install_gpu_driver.py
```

If at any point you get `unable to find the kernel source tree for the currently running kernel`, run the following command before trying `nvidia-smi` again:

```bash
sudo apt-get install linux-headers-`uname -r`
```

Next, install `pip` and all the libraries you need...

```bash
sudo apt install python3-pip
pip3 install jupyter jupyterlab mat73 scipy numpy torch matplotlib tqdm pillow natsort
```

And add them to your path:

```bash
nano ~/.bashrc

# add the following to the top of the file:
export PATH=$PATH:~/.local/bin

# restart your SSH session
```

Now we can configure Jupyter.

```bash
jupyter notebook --generate-config
nano ~/.jupyter/jupyter_notebook_config.py
```

Add the following to the top of the file, after `c = get_config()`:

```bash

c.NotebookApp.allow_origin = "*"

c.NotebookApp.ip = '0.0.0.0'

c.NotebookApp.password_required = True

c.NotebookApp.port = 8888

c.NotebookApp.open_browser = False
```

And set a password for Jupyter:

```bash
python3 -m jupyter_server.auth password
```

You can now run Jupyter using `jupyter notebook` or, for the bold, `jupyter-lab`!

# Using GCP for MATLAB

Python is better ;), but here's a guide to installing MATLAB on a GCP instance if your application necessitates it. First, download the installer for your operating system and sign in. Select "Advanced Options" on the top-right and select "I want to download without installing."

Then, copy the installation to the instance:

```bash
gcloud compute scp --recurse ~/Downloads/MathWorks/ [username]@[instance]:~/
```

Pro Tip: if you're running many instances for say, parallelized GPU-accelerated MATLAB, there's a very easy way to `scp` between instances [here](#scp-between-instances). 

Generate a license for your MATLAB install. Go to your MATLAB Account -> Click on Your License -> Install and Activate -> Activate to Retrieve License File -> Activate a Computer. You will need your username (`whoami`) and Host ID (`ip addr | grep ether`).

Another Tip: if you ever want to change the license on your existing install, all you have to do is move the new license to the `licenses` folder inside the install. For example, `sudo mv ~/license.lic /opt/MATLAB/licenses`.

Next, navigate into your MATLAB Installation and copy the `installer_input.txt` file. Fill it out with your destination folder (I use `/opt/MATLAB/R2023a/`), installation key that you just got from MATLAB, and license path (I `scp` this to `/tmp/license.dat`). 

Finally, you can install MATLAB using the following:

```bash
sudo mkdir /opt/MATLAB
sudo apt install libxt6 
cd MathWorks/R2023a/[date]/
sudo ./install -inputfile ~/installer_input.txt
```

To open MATLAB, simply run `/opt/MATLAB/bin/matlab`. As there is no GUI here, I recommend developing/testing your script locally and only running it on the instance when you're finished. 

# Tips and Tricks

### SCP Between Instances

Simply run `gcloud auth login` and you'll be able to `scp` between instances on the same account. If others use the instance, you may want to run `gcloud auth revoke` afterwards to protect your credentials.

If you're working between different zones, you can use `gcloud config set compute/zone us-west1-b` to change your current zone when using secure copy (scp).

### Create a Shared Directory

```bash
sudo mkdir /home/shared
sudo groupadd mygroup
sudo chgrp mygroup /home/shared
sudo chmod 775 -R /home/shared
sudo chmod +s /home/shared

sudo usermod -a -G mygroup user1
...
sudo usermod -a -G mygroup userN

# restart your SSH session
```

### Large Directories

Print the number of files in the current directory using `ls | wc -l`, or `ls ~/directory/ | wc -l` for a different directory.

Print the file name of the last file in a directory (natural sorted) using `ls -v ~/directory/ | tail -n 1`

### Duplicate Instances

If you're parallelizing a task and need to duplicate many Compute Platform instances, you can do by creating a snapshot of the instance and basing each new instance disk off of that snapshot. [This page](https://cloud.google.com/compute/docs/disks/create-snapshots) has more information about snapshots.

### Python or Pip Issues?

Try `which python` and `echo $PATH` to see where your install is located or being sourced. Also take a look at your profile configurations with `nano ~/.bashrc` and `nano ~/.profile` to see if any package (conda for instance) might be modifying the path.

<!-- 
## Fancy Terminal Profiles (Mac)

You can customize your terminal in many ways. Below is my (very simple) configuration, which I put in `.zshrc`. [Here](https://www.cyberciti.biz/faq/apple-mac-osx-terminal-color-ls-output-option/#:~:text=You%20can%20enable%20colorized%20output,colorized%20output%20on%20Unix%20box.) is a link that explains the terminal colors.

```bash
PROMPT='%(?.%F{green}√.%F{red}%?%f) %F{blue}%~ >%f '

# Terminal Colors
export CLICOLOR=1
export LSCOLORS=ExFxBxDxCxegedabagacad

# Aliases
alias ls="ls -a"
``` -->
