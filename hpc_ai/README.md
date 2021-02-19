* hpc_ai: This directory contains labs related to usage of AI/ML/DL for Science/HPC Simulations

# Alvis instructions

## Adjustments of notebooks

In notebooks using the data_utils.py script under `workspace/python/source_code/utils`,
you need to make an adjustment to be able to run it on the login node or on compute nodes 
without Singularity.
Thus, if you see `sys.path.append('/workspace/python/source_code')` in a code cell, replace
it with `sys.path.append('/path/to/home/workspace/python/source_code')` where
you replace `/path/to/home` with the actual path of your home directory (you can find it by typing
`echo $HOME` in terminal). With this change, the notebook will still run fine inside Singularity on
a compute node.

## Running without Singularity

Load modules:
```
module load GCC/8.3.0  CUDA/10.1.243  OpenMPI/3.1.4  IPython/7.9.0-Python-3.7.4
module load TensorFlow/2.3.1-Python-3.7.4
#module load scikit-learn/0.21.3-Python-3.7.4
```

Install a missing package into your home directory (under `$HOME/.local`)
```
pip install --user scikit-fmm==0.0.7
```

Copy all notebooks from the Singularity image to your home directory.
For the CFD case:
```
singularity run /cephyr/NOBACKUP/Datasets/Practical_DL/ai_science_cfd.simg cp -rT /workspace ~/workspace
```
For the climate case:
```
singularity run /cephyr/NOBACKUP/Datasets/Practical_DL/ai_science_climate.simg cp -rT /workspace ~/workspace
```

## Using Singularity

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

