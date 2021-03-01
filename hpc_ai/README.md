* hpc_ai: This directory contains labs related to usage of AI/ML/DL for Science/HPC Simulations

# Alvis instructions

## Getting access to Alvis

1. You need a SUPR account to access Alvis. If you don't already have one, visit https://supr.snic.se/ 
   and follow the instructions. Your SUPR account will be added to SNIC project for the bootcamp event. 
   After being added, you can request a user account on Alvis via https://supr.snic.se/account/.
   C3SE will create your user account and send the username and one-time temporary password in 
   separate emails.

2. Login to Alvis requires that you are using a Swedish university network. If you are working from a 
   university building and are using Eduroam or an ethernet connection, you should be able to log in 
   right away. If you are working from home you will need to set up a Virtual Private Network (VPN) 
   service, either the 
   [Chalmers VPN service](https://it.portal.chalmers.se/itportal/NonCDAWindows/NonCDAWindows#remote) 
   or another VPN service from your university. 
   Contact your university's IT support to get help to set it up. 
   Further information about accessing Alvis is available on the 
   [C3SE support pages](https://www.c3se.chalmers.se/documentation/connecting/).

3. Login to Alvis using your personal account:
   ``` 
   ssh -l <username> alvis1.c3se.chalmers.se
   ```

## Configuring Jupyter

You need to configure Jupyter to be able to launch a Jupyter server on Alvis 
and connect to it via your local browser.

Create `~/.jupyter/jupyter_notebook_config.py` with the following content:

```python
import errno, socket, random
def get_available_port(port_retries=10, ip='127.0.0.1'):
    for port in (random.randrange(8888, 8988) for i in range(port_retries)):
        try: socket.socket(socket.AF_INET, socket.SOCK_STREAM).bind((ip, port))
        except: print('Port %i not available, trying another port.' % port)
        else: return port
    print('ERROR: No available port could be found.'); exit(1)

port, hostname = get_available_port(), socket.gethostname()

c.NotebookApp.ip = hostname
c.NotebookApp.port = port
c.NotebookApp.base_url = '/{0}/'.format(hostname)
c.NotebookApp.custom_display_url = 'https://proxy.c3se.chalmers.se:{0}/{1}/'.format(port, hostname)
c.NotebookApp.allow_origin = '*'
c.NotebookApp.port_retries = 0
c.NotebookApp.open_browser = False
c.NotebookApp.allow_remote_access = True
```

You should now be able to run Jupyter and connect to it. First load module 
and run `jupyter-notebook`:

```bash
module load IPython/7.9.0-Python-3.7.4
jupyter-notebook
```

The output will show an URL like 
`https://proxy.c3se.chalmers.se:8932/alvis1/?token=667b5bb97e01032ed12347898f66c20c79234e34ffd0bdb9f7`
which you should copy-paste into your Firefox or Chrome browser. 

## Running notebooks on Alvis

After downloading the Jupyter notebooks and datasets to Alvis, 
you should run Jupyter on the Alvis login node to follow the training. You can run code cells 
to create and inspect models and plot data, but **all compute-intensive work should be run 
non-interactively on a compute node**. See detailed instructions below.

### Fetch notebooks and datasets

After logging in to Alvis, clone this repository to get the notebooks to 
your Alvis home directory (**it must be the home directory!**):
```bash
cd
git clone https://github.com/ENCCS/gpubootcamp.git
cd gpubootcamp
git checkout alvis
```

You will find all material under the `hpc_ai` subdirectory. There are 
two science cases which will be covered on day 2, 
`ai_science_cfd` and `ai_science_climate`. Under both folders,  
Jupyter notebooks can be found under `English/python/jupyter_notebook/`.

For the `ai_science_climate` case, you will need to copy a dataset 
(unless you're running inside Singularity, see below):
```
cd ~/gpubootcamp/hpc_ai/ai_science_climate/English/python/jupyter_notebook/Tropical_Cyclone_Intensity_Estimation
unzip unzip /cephyr/NOBACKUP/Datasets/Practical_DL/dataset.zip
```
This will unpack the dataset of tropical cyclones into the correct directory.

### Adjust notebooks

You may need to add a code cell with `%matplotlib inline` to certain notebooks
to make plots show properly.

### (Standard) Running without Singularity

Load modules:
```
module load GCC/8.3.0  CUDA/10.1.243  OpenMPI/3.1.4  IPython/7.9.0-Python-3.7.4
module load TensorFlow/2.3.1-Python-3.7.4
module load scikit-learn/0.21.3-Python-3.7.4
```

Install missing package into your home directory (under `$HOME/.local`)
```
pip install --user scikit-fmm==0.0.7
pip install --user opencv-python
```

### (Optional) Using Singularity

On the Alvis cluster we have a reservation for one compute node with 4 V100 GPUs and
one with 8 T4 GPUs. The T4 GPUs are for quick testing while production results
should be run on a V100. Each run will use a single GPU, and since multiple users can
share a compute node there can be up to 4 and 8 simultaneous runs on the V100 and T4 nodes,
respectively.

Start by copying all notebooks from the Singularity image to your home directory.
For the CFD case:
```
singularity run /cephyr/NOBACKUP/Datasets/Practical_DL/ai_science_cfd.simg cp -rT /workspace ~/workspace
```
For the climate case:
```
singularity run /cephyr/NOBACKUP/Datasets/Practical_DL/ai_science_climate.simg cp -rT /workspace ~/workspace
```

Module combo for running jupyter on login node:

```
module load GCC/8.3.0  CUDA/10.1.243  OpenMPI/3.1.4  IPython/7.9.0-Python-3.7.4
module load TensorFlow/2.3.1-Python-3.7.4
```

Open terminal inside Jupyter, and submit job inside Singularity container using
`jupyter-nbconvert --execute` to execute the notebook (Part_2.ipynb used here as an example):

```
module purge   # needed to not interfere with Jupyter running inside Singularity
srun -A SNIC2020-5-235 --gpus-per-node=V100:1 -t 0:10:00 -N 1 singularity run --nv /cephyr/NOBACKUP/Datasets/Practic\
al_DL/ai_science_cfd.simg jupyter nbconvert --ExecutePreprocessor.timeout=-1 --to notebook --execute ~/workspace/pyt\
hon/jupyter_notebook/Intro_to_DL/Part_2.ipynb
```
You will see an error message `AttributeError: 'NoneType' object has no attribute 'thread'` at the end of
the run but it is harmless.

After the run a new notebook is created with the name `Part_2.nbconvert.ipynb`. Open it
from the Jupyter dashboard and inspect the results. If you want to rerun `Part_2.ipynb`,
**make sure to close the `Part_2.nbconvert.ipynb` notebook by clicking "File" and "Close and halt".**
Otherwise the running notebook might autosave and overwrite the results of your new run.

