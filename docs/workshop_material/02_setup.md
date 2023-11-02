# 02 - Setup



## Install Miniconda

For this workshop, we will analyse our data using various software. However, the only software we will need to manually install is [Miniconda](https://docs.conda.io/en/latest/miniconda.html).

### Check your OS

If you already use Linux or MacOS X, great! Ignore this paragraph!. If you use Windows, log in to your ICER HPCC account instead.

### Installing miniconda

Information on how to install Miniconda can be found [on their website](https://docs.conda.io/en/latest/miniconda.html). Snakemake also provides information on installing Miniconda in [their documentation](https://snakemake.readthedocs.io/en/stable/tutorial/setup.html#step-1-installing-miniconda-3)

Once miniconda is installed, [set up your channels](https://bioconda.github.io/user/install.html#set-up-channels) (channels are locations where packages/software are can be installed from)

```bash
conda config --add channels defaults
conda config --add channels bioconda
conda config --add channels conda-forge
```

## Create a conda environment

With Miniconda, we can create a conda environment which acts as a space contained from the rest of the machine in which our workflow will automatically install all the necessary software it uses, supporting the portability and reproducibility of your workflow.

Create a conda environment (called `snakemake_env`) that has Snakemake installed (and all it's dependant software) and git (which will be used to clone this repository later)

```bash
conda create -n snakemake_env snakemake mamba
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
