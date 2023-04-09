# Google Cloud Platform (GCP) Instances

This guide is meant to provide a concise resource for those doing machine learning (using PyTorch) or simulation work (in MATLAB) on GCP Compute Engine Instances. It also includes random Linux commands and tips/tricks that may come in handy. I must credit the [Stanford CS231n Google Cloud Tutorial Repository](https://github.com/cs231n/gcloud), which has helped me countless times with my instances. 

(Last Updated on April 8, 2023)

# Table of Contents

1. [Setting Up a New Instance](#setting-up-a-new-instance)
2. [Using GCP for MATLAB](#using-gcp-for-matlab)
3. [Tips and Tricks](#tips-and-tricks)
  1. [SCP Between Instances](#scp-between-instances)
  2. [Create a Shared Directory](#create-a-shared-directory)
  3. [Random Commands](#random-commands)

# Setting Up a New Instance

This guide assumes you have a [Google Cloud account](https://cloud.google.com/) with a sufficient GPU quota (once you have [upgraded your account](https://cloud.google.com/free/docs/free-cloud-features#how-to-upgrade), go to IAM Admin -> Quotas) 

**Image**: keep the default Debian image. I found the 'GPU-optimized Debian 10 with CUDA 11.0' to be more tricky than helpful, unfortunately.

First, install your NVIDIA drivers and CUDA using this [handy script](https://cloud.google.com/compute/docs/gpus/install-drivers-gpu) from GCP:

```bash
curl https://raw.githubusercontent.com/GoogleCloudPlatform/compute-gpu-installation/main/linux/install_gpu_driver.py --output install_gpu_driver.py
sudo python3 install_gpu_driver.py
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

Add the following to the top of the file:

```bash
c = get_config()  #noqa

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

First, download the installer for your operating system and sign in. Select "Advanced Options" on the top-right and select "I want to download without installing."

Then, copy the installation to the instance:

```bash
gcloud compute scp --recurse ~/Downloads/MathWorks/ [username]@[instance]:~/
```

Pro Tip: if you're running many instances for say, parallelized GPU-accelerated MATLAB, there's a very easy way to `scp` between instances [here](#scp-between-instances).

Generate a license for your MATLAB install. Go to your MATLAB Account -> Click on Your License -> Install and Activate -> Activate to Retrieve License File -> Activate a Computer. You will need your username (`whoami`) and Host ID (`ip addr | grep ether`).

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

sudo adduser user1 mygroup
...
sudo adduser userN mygroup

# restart your SSH session
```

### Random Commands

Print the number of files in the current directory using `ls | wc -l`, or `ls ~/directory/ | wc -l` for a different directory.

Print the file name of the last file in a directory (natural sorted) using `ls -v ~/directory/ | tail -n 1`

