# Google Cloud Platform (GCP) Instances

This guide is meant to provide a concise resource for those doing machine learning (using PyTorch) or simulation work (in MATLAB) on GCP Compute Engine Instances. It also includes random Linux commands and tips/tricks that may come in handy. I must credit the [Stanford CS231n Google Cloud Tutorial Repository](https://github.com/cs231n/gcloud), which has helped me countless times with my instances. 

(Last Updated on April 8, 2023)

# Table of Contents

1. [Setting Up a New Instance](#setting-up-a-new-instance)
2. [Using GCP for MATLAB](#using-gcp-for-matlab)
3. [Tips and Tricks](#tips-and-tricks)

# Setting Up a New Instance

This guide assumes you have a [Google Cloud account](https://cloud.google.com/) with a sufficient GPU quota (once you have [upgraded your account](https://cloud.google.com/free/docs/free-cloud-features#how-to-upgrade), go to IAM Admin -> Quotas) 

**Image**: keep the default Debian image. I found the 'GPU-optimized Debian 10 with CUDA 11.0' to be more tricky than helpful, unfortunately.

First, install your NVIDIA drivers and CUDA using this [handy script](https://cloud.google.com/compute/docs/gpus/install-drivers-gpu) from GCP.

```bash
curl https://raw.githubusercontent.com/GoogleCloudPlatform/compute-gpu-installation/main/linux/install_gpu_driver.py --output install_gpu_driver.py
sudo python3 install_gpu_driver.py
```

Next, install `pip`...

```bash
sudo apt install python3-pip
pip3 install jupyter jupyterlab mat73 scipy numpy torch matplotlib tqdm pillow natsort
```

And add it to your path:

```bash
nano ~/.bashrc
export PATH=$PATH:~/.local/bin
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

Pro Tip: if you're running a cluster of instances for MATLAB like a madman, there's a very easy way to `scp` between instances [here](#scp-between-instances).


# Tips and Tricks

### SCP Between Instances

Simply run `gcloud auth login` and you'll be able to `scp` between instances on the same account. If others use the instance, you may want to run `gcloud auth revoke` afterwards to protect your credentials.
