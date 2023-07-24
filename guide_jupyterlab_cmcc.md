# JupyterLab on the CMCC HPC system

### Purpose

The purpose of this guide is to provide instructions for running JupyterLab on the CMCC HPC systems (Zeus and Juno) from a compute node. JupyterLab, developed and maintained by Jupyter, is the latest web-based interactive development environment for notebooks, code, and data. Its flexible interface allows users to configure and arrange workflows in data science, scientific computing, computational journalism, and machine learning. A modular design invites extensions to expand and enrich functionality. More information on JupyterLab can be found [here](https://jupyter.org).

Admittedly, this process is quite convoluted and it would be great to have a [JupyterHub](https://jupyter.org/hub) installation at CMCC to bypass all this. We will inquiry with the HPC Systems Management if this is a possibility.

### How to get help

If these steps do not work for you, feel free to open an issue in this git repository and update this documentation to make the steps more clear.

### Additional resources 

A lot of this is inspired by this guide: [How to Connect to JupyterLab Remotely](https://towardsdatascience.com/how-to-connect-to-jupyterlab-remotely-9180b57c45bb) and by several posts in Stack Overflow. Also, running this requires to have some familiarity with [`conda`](https://conda.io/projects/conda/en/latest/user-guide/getting-started.html) or [`mamba`](https://mamba.readthedocs.io/en/latest/) to manage your kernels to deploy on JupyterLab. Also, I assume you have the login credentials for the CMCC HPC platforms, and that you have space for storing your data on the systems.


## STEP BY STEP GUIDE


### Step 0 – Familiarize yourself with `screen` (not mandatory)

[`screen`](https://linuxize.com/post/how-to-use-linux-screen/) is a tool that helps you manage your sessions on Zeus or Juno. Using `screen` is useful also by itself for many reasons. For instance, if your VPN connection drops because of internet issues or simply because you leave the office, your work environment on the server survives and can be recovered. `screen` is easy to use and it is available on the CMCC HPC systems. 

The first step is to create a `screen` session called `jupyterlab_screen` where we will run our JupyterLab. 

```
screen -S jupyterlab_screen
```

This session is supposed to survive so there is no need to create a new session every time. I guess that once a week the jobs running on the Zeus and Juno are cleaned, probably during the weekend, and this includes the `screen` sessions. So if you cannot find your old session you can always create a new one. To list your `screen` sessions run. 

```
screen -ls
```

You should see something like 

```
There is a screen on:
	169723.jupyterlab_screen	(Attached)
1 Socket in /var/run/screen/S-lz01723.
```

A session can be Attached or Detached. To reattach to an existing session run `>screen -R jupyterlab_screen`. This should be enough for our purpose, but more information can be found online. The following steps can be also done without `screen`. However, if your VPN connection drops, you very likely need to start from scratch. 


### Step 1 – Installing JupyterLab

**This step needs to be done just once!** Create a new conda environment named *generic* and install JupyterLab. Here, I use `mamba` because it is faster, but all the commands can be adapted by substituting `mamba` with `conda`.

```
conda create --name generic
conda activate generic
mamba install -c conda-foge jupyterlab
```

I suggested creating a clean conda environment to install JupyterLab. I think it is smart to dedicate one conda environment just to the JupyterLab installation to keep it slim and to avoid breaking it in the future, but you are free to install JupyterLab also on an existing conda environemt. 

Protecting your JupyterLab with a password is a good idea, so that your work remains protected. You can refer to the guide [How to Connect to JupyterLab Remotely](https://towardsdatascience.com/how-to-connect-to-jupyterlab-remotely-9180b57c45bb) for more instructions on this. 


### Step 2 – Run JupyterLab from a compute node

From a screen session (or not), start an interactive bash shell on a compute node.

```
bsub -q p_long -n 4 -P R000 -Is bash
```

Here I am requesting 4 processors on the `p_long` queue. You should adapt these settings based on your needs. At this point, you can activate the `generic` conda environment where JupyterLab is installed, and run JupyterLab

```
conda activate generic
jupyter lab --no-browser --ip=0.0.0.0
```

**You should note which compute node you are using. You will need it in the next step!** The node number is a string like n### (e.g., `n097`).


### Step 3 – Port forwarding Jupyterlab

This step is needed to make sure we can access our JupyterLab from a browser. In a new terminal, connect to Zeus (or Juno) via

```
ssh -l cmcc_user zeus01.cmcc.scc -L 8888:n###:8888
```

where `cmcc_user` is the your CMCC HPC username, `n###` is the node number from the previous step.

Now open your browser and open the page [http://localhost:8888/lab](http://localhost:8888/lab) and you should be able to access JupyterLab. The password to access JupyterLab might be required if you set it. 


### Step 3 – Adding more kernels to Jupyterlab

The Python kernel used by default by your lab is the one you launched `jupyter lab --no-browser --ip=0.0.0.0` from. In our case, it is `generic`. To create a new conda environment and install it as IPython kernel do the following.

```
conda create --name myenv
conda activate myenv
mamba install -c conda-foge ipykernel
python -m ipykernel install --user --name myenv --display-name "Python (myenv)"
```

You should now see a kernel `Python (myenv)` in JupyterLab.
