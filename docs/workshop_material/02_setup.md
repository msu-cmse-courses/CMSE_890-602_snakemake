# 02 - Setup



## Install Miniforge

For this workshop, we will analyse our data using various software. However, the only software we will need to manually install is [Miniforge](https://github.com/conda-forge/miniforge).

### Check your OS

If you already use Linux or MacOS X, great! Ignore this paragraph!. If you use Windows, log in to your ICER HPCC account instead. The total required storage for this 
workshop is **~2 GB**.

### Installing Miniforge

Information on how to install Miniforge can be found [on their repository](https://github.com/conda-forge/miniforge?tab=readme-ov-file).

### What if I already have a conda installation, or I am on ICER?

Great! You can use your existing conda installation, whether it be Anaconda, Miniconda, Miniforge etc. On ICER, you can also load our Miniforge3 module with the command `module purge; module load Miniforge3`.

## Create a conda environment

With Miniforge, we can create a conda environment which acts as a space contained from the rest of the machine in which our workflow will automatically install all the necessary software it uses, supporting the portability and reproducibility of your workflow.

Create a conda environment (called `snakemake_env`) that has Snakemake installed (and all it's dependant software) and git (which will be used to clone this repository later)

```bash
conda create -n snakemake_env bioconda::snakemake
```

Respond yes to the following prompt to install the necessary software in the new conda environment:

```bash
Proceed ([y]/n)?
```

**Note. this installed Snakemake version 6.7.0 for me, you can use the same version this workshop was created with `conda create -n snakemake_env snakemake=6.7.0`**

Activate the conda environment we just created

```bash
conda activate snakemake_env
```

Now we can see which conda environment we are in on the command line, `(base)` has been replaced with `(snakemake_env)`

```bash
(snakemake_env) bash-4.2$ 
```

*Snakemake has been installed within your `snakemake_env` environment, so you won't be able to see or use your Snakemake install unless you are within this environment*


## Clone this repo

Clone this repo with the following:

!!! terminal "code"

    ```bash
    git clone https://github.com/msu-cmse-courses/CMSE_890-602_snakemake.git
    ```
    ```bash
    cd CMSE_890-602_snakemake
    ```
    !!! info ""

        See the [Git Guides](https://github.com/git-guides/git-clone) for information on cloning a repo

- - - 

<p align="center"><b><a class="btn" href="https://msu-cmse-courses.github.io/CMSE_890-602_snakemake/" style="background: var(--bs-dark);font-weight:bold">Back to homepage</a></b></p>
