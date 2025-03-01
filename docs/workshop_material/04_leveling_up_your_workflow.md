# 04 - Leveling up your workflow!

## Catching up

!!! file-code "From section 03, you should have the following Snakefile:"

    ```python
    # define samples from data directory using wildcards
    SAMPLES, = glob_wildcards("../../data/{sample}_1.fastq.gz")
    
    # target OUTPUT files for the whole workflow
    rule all:
        input:
            "../results/multiqc_report.html",
            expand(["../results/trimmed/{sample}_1_val_1.fq.gz", "../results/trimmed/{sample}_2_val_2.fq.gz"], sample = SAMPLES)
    
    # workflow
    rule fastqc:
        input:
            R1 = "../../data/{sample}_1.fastq.gz",
            R2 = "../../data/{sample}_2.fastq.gz"
        output:
            html = ["../results/fastqc/{sample}_1_fastqc.html", "../results/fastqc/{sample}_2_fastqc.html"],
            zip = ["../results/fastqc/{sample}_1_fastqc.zip", "../results/fastqc/{sample}_2_fastqc.zip"]
        log:
            "logs/fastqc/{sample}.log"
        threads: 2
        conda:
            "envs/fastqc.yaml"
        shell:
            "fastqc {input.R1} {input.R2} -o ../results/fastqc/ -t {threads} &> {log}"
      
    rule multiqc:
        input:
            expand(["../results/fastqc/{sample}_1_fastqc.zip", "../results/fastqc/{sample}_2_fastqc.zip"], sample = SAMPLES)
        output:
            "../results/multiqc_report.html"
        log:
            "logs/multiqc/multiqc.log"
        conda:
            "envs/multiqc.yaml"
        shell:
            "multiqc {input} -o ../results/ &> {log}"
    
    rule trim_galore:
        input:
            ["../../data/{sample}_1.fastq.gz", "../../data/{sample}_2.fastq.gz"]
        output:
            ["../results/trimmed/{sample}_1_val_1.fq.gz", "../results/trimmed/{sample}_2_val_2.fq.gz"]
        log:
            "logs/trim_galore/{sample}.log"
        conda:
            "envs/trimgalore.yaml"
        threads: 2
        shell:
            "trim_galore {input} -o ../results/trimmed/ --paired --cores {threads} &> {log}"
    ```

<br>

## 4.1 Use a profile for HPC

In section 3.16, we have seen that a snakemake workflow can be run on an HPC cluster.
To reduce the boilerplate, we can use a [configuration profile](https://snakemake.readthedocs.io/en/stable/executing/cli.html#profiles) to configure default options.
In this case, we use it to set the `--executor` and the `--jobs` options.
Default resources for all rules should be set using the `default-resources` option.

!!! terminal-2 "Make a `slurm` profile folder"

    ```bash
    # create the profile folder inside demo_workflow/workflow
    cd demo_workflow/workflow
    mkdir slurm
    touch slurm/config.yaml
    ```
!!! file-code "write the following to config.yaml"
    ```bash
    jobs: 20
    executor: slurm
    default-resources:
        runtime: 10
        mem_mb: 512
        cpus_per_task: 8
    ```

!!! terminal-2 "Then run the snakemake workflow using the `slurm` profile"

    ```bash
    # remove output of last run
    rm -r ../results/*
    ```
    ```bash
    # run dryrun/run again
    snakemake --dryrun --workflow-profile slurm --software-deployment-method conda
    ```
    ```bash
    snakemake --workflow-profile slurm --software-deployment-method conda
    ```

!!! failure "Possible error"

    You may get errors like this:
    ```
    Error executing rule trim_galore on cluster (jobid: 6, external: 25266818, jobscript: /mnt/ufs18/home-081/fullarda/CMSE_890-602_snakemake/demo_workflow/workflow/.snakemake/tmp.yebsz1sm/snakejob.trim_galore.6.sh). For error details see the cluster log and the log files of the involved rule(s).
    ```
    The "cluster log" is the SLURM output log of the job. If you read the SLURM logs, it may show errors like this:
    ```
    MissingOutputException in rule trim_galore in file /mnt/ufs18/home-081/fullarda/CMSE_890-602_snakemake/demo_workflow/workflow/Snakefile, line 49:
    Job 0 completed successfully, but some output files are missing. Missing files after 5 seconds. This might be due to filesystem latency. If that is the case, consider to increase the wait time with --latency-wait.
    ```

    To resolve this, add `--latency-wait 30` to increase the wait time to 30s. This value can be increased further if needed.

If you interrupt the execution of a snakemake workflow using <KBD>CTRL+C</KBD>, the SLURM executor plugin will automatically cancel the jobs.

You can specify different resources (memory, cpus, gpus, etc.) for each rule in the workflow.

Here we give more CPU resources to `trim_galore` to make it run faster.

??? code-compare "Edit snakefile"

    ```diff
    # define samples from data directory using wildcards
    SAMPLES, = glob_wildcards("../../data/{sample}_1.fastq.gz")
    
    # target OUTPUT files for the whole workflow
    rule all:
        input:
            "../results/multiqc_report.html",
            expand(["../results/trimmed/{sample}_1_val_1.fq.gz", "../results/trimmed/{sample}_2_val_2.fq.gz"], sample = SAMPLES)
    
    # workflow
    rule fastqc:
        input:
            R1 = "../../data/{sample}_1.fastq.gz",
            R2 = "../../data/{sample}_2.fastq.gz"
        output:
            html = ["../results/fastqc/{sample}_1_fastqc.html", "../results/fastqc/{sample}_2_fastqc.html"],
            zip = ["../results/fastqc/{sample}_1_fastqc.zip", "../results/fastqc/{sample}_2_fastqc.zip"]
        log:
            "logs/fastqc/{sample}.log"
        threads: 2
        conda:
            "envs/fastqc.yaml"
        shell:
            "fastqc {input.R1} {input.R2} -o ../results/fastqc/ -t {threads} &> {log}"
      
    rule multiqc:
        input:
            expand(["../results/fastqc/{sample}_1_fastqc.zip", "../results/fastqc/{sample}_2_fastqc.zip"], sample = SAMPLES)
        output:
            "../results/multiqc_report.html"
        log:
            "logs/multiqc/multiqc.log"
        conda:
            "envs/multiqc.yaml"
        shell:
            "multiqc {input} -o ../results/ &> {log}"
    
    rule trim_galore:
        input:
            ["../../data/{sample}_1.fastq.gz", "../../data/{sample}_2.fastq.gz"]
        output:
            ["../results/trimmed/{sample}_1_val_1.fq.gz", "../results/trimmed/{sample}_2_val_2.fq.gz"]
        log:
            "logs/trim_galore/{sample}.log"
        conda:
            "envs/trimgalore.yaml"
        threads: 2
    +   resources:
    +       cpus_per_task=8
        shell:
            "trim_galore {input} -o ../results/trimmed/ --paired --cores {threads} &> {log}"
    ```


??? file-code "Current snakefile:"


    ```python
    # define samples from data directory using wildcards
    SAMPLES, = glob_wildcards("../../data/{sample}_1.fastq.gz")
    
    # target OUTPUT files for the whole workflow
    rule all:
        input:
            "../results/multiqc_report.html",
            expand(["../results/trimmed/{sample}_1_val_1.fq.gz", "../results/trimmed/{sample}_2_val_2.fq.gz"], sample = SAMPLES)
    
    # workflow
    rule fastqc:
        input:
            R1 = "../../data/{sample}_1.fastq.gz",
            R2 = "../../data/{sample}_2.fastq.gz"
        output:
            html = ["../results/fastqc/{sample}_1_fastqc.html", "../results/fastqc/{sample}_2_fastqc.html"],
            zip = ["../results/fastqc/{sample}_1_fastqc.zip", "../results/fastqc/{sample}_2_fastqc.zip"]
        log:
            "logs/fastqc/{sample}.log"
        threads: 2
        conda:
            "envs/fastqc.yaml"
        shell:
            "fastqc {input.R1} {input.R2} -o ../results/fastqc/ -t {threads} &> {log}"
      
    rule multiqc:
        input:
            expand(["../results/fastqc/{sample}_1_fastqc.zip", "../results/fastqc/{sample}_2_fastqc.zip"], sample = SAMPLES)
        output:
            "../results/multiqc_report.html"
        log:
            "logs/multiqc/multiqc.log"
        conda:
            "envs/multiqc.yaml"
        shell:
            "multiqc {input} -o ../results/ &> {log}"
    
    rule trim_galore:
        input:
            ["../../data/{sample}_1.fastq.gz", "../../data/{sample}_2.fastq.gz"]
        output:
            ["../results/trimmed/{sample}_1_val_1.fq.gz", "../results/trimmed/{sample}_2_val_2.fq.gz"]
        log:
            "logs/trim_galore/{sample}.log"
        conda:
            "envs/trimgalore.yaml"
        threads: 2
        resources:
            cpus_per_task=8
        shell:
            "trim_galore {input} -o ../results/trimmed/ --paired --cores {threads} &> {log}"
    ```
    
<br>

!!! terminal-2 "Run the workflow again"

    ```bash
    # remove output of last run
    rm -r ../results/*
    ```
    ```
    # run dryrun/run again
    snakemake --dryrun --workflow-profile slurm --software-deployment-method conda
    ```
    ```bash
    snakemake --workflow-profile slurm --software-deployment-method conda
    ```

    - If you monitor the progress of your jobs using `squeue --me -o "%.7i %9P %35j %.8u %.2t %.12M %.12L %.5C %.7m %.4D %R"`, you will notice that some jobs now request 2 or 8 CPUs.

    ??? success "output"

        ```
        JOBID         USER     ACCOUNT   NAME        CPUS MIN_MEM PARTITI START_TIME     TIME_LEFT STATE    NODELIST(REASON)
        26763281      lkemp    nesi99991 spawner-jupy   4      4G interac 2022-05-11T1     7:21:18 RUNNING  wbn003
        26763492      lkemp    nesi99991 snakejob.fas   2    512M large   2022-05-11T1        9:59 RUNNING  wbn144
        26763493      lkemp    nesi99991 snakejob.tri   8    512M large   2022-05-11T1        9:59 RUNNING  wbn212
        26763494      lkemp    nesi99991 snakejob.fas   2    512M large   2022-05-11T1        9:59 RUNNING  wbn145
        26763495      lkemp    nesi99991 snakejob.fas   2    512M large   2022-05-11T1        9:59 RUNNING  wbn146
        26763496      lkemp    nesi99991 snakejob.tri   8    512M large   2022-05-11T1        9:59 RUNNING  wbn217
        26763497      lkemp    nesi99991 snakejob.tri   8    512M large   2022-05-11T1        9:59 RUNNING  wbn229
        ```
    
<br>

### Logging

SLURM log files for each rule can be found in the workflow's `.snakemake/slurm_logs` directory.
The SLURM logs are stored separately for each rule within a directory named after the rule.

### Snakemake-SLURM communication

The SLURM executor plugin tracks job completion, failure and cancellations.
You can request specific numbers of retries in the case of job failures with the
option `--retries=N` where `N` is the number of retries per rule.

Alternatively, you can use SLURM itself to handle the requeuing of jobs by adding
`--slurm-requeue` to the `snakemake` command or in the workflow profile.

Once all of this is in place, we can:

!!! quote ""

    - submit Slurm jobs with the right resources per Snakemake rule,
    - cancel the workflow and Slurms jobs using CTRL-C,
    - keep all slurm jobs log files in a dedicated folder.

!!! question "Exercise"

    Run the snakemake workflow with Slurm jobs then use `scancel JOBID` to cancel some Slurm. See how Snakemake reacts.


## 4.2 Pull out parameters

We can set parameters for commands using the `params` rule option so that they can be more human-readable:

??? code-compare "Edit snakefile"
    
    ```diff
    # define samples from data directory using wildcards
    SAMPLES, = glob_wildcards("../../data/{sample}_1.fastq.gz")
    
    # target OUTPUT files for the whole workflow
    rule all:
        input:
            "../results/multiqc_report.html",
            expand(["../results/trimmed/{sample}_1_val_1.fq.gz", "../results/trimmed/{sample}_2_val_2.fq.gz"], sample = SAMPLES)
    
    # workflow
    rule fastqc:
        input:
            R1 = "../../data/{sample}_1.fastq.gz",
            R2 = "../../data/{sample}_2.fastq.gz"
        output:
            html = ["../results/fastqc/{sample}_1_fastqc.html", "../results/fastqc/{sample}_2_fastqc.html"],
            zip = ["../results/fastqc/{sample}_1_fastqc.zip", "../results/fastqc/{sample}_2_fastqc.zip"]
        log:
            "logs/fastqc/{sample}.log"
        threads: 2
        conda:
            "envs/fastqc.yaml"
        shell:
            "fastqc {input.R1} {input.R2} -o ../results/fastqc/ -t {threads} &> {log}"
      
    rule multiqc:
        input:
            expand(["../results/fastqc/{sample}_1_fastqc.zip", "../results/fastqc/{sample}_2_fastqc.zip"], sample = SAMPLES)
        output:
            "../results/multiqc_report.html"
        log:
            "logs/multiqc/multiqc.log"
        conda:
            "envs/multiqc.yaml"
        shell:
            "multiqc {input} -o ../results/ &> {log}"
    
    rule trim_galore:
        input:
            ["../../data/{sample}_1.fastq.gz", "../../data/{sample}_2.fastq.gz"]
        output:
            ["../results/trimmed/{sample}_1_val_1.fq.gz", "../results/trimmed/{sample}_2_val_2.fq.gz"]
    +   params:
    +       "--paired"
        log:
            "logs/trim_galore/{sample}.log"
        conda:
            "envs/trimgalore.yaml"
        threads: 2
        resources:
            cpus_per_task=8
        shell:
    -       "trim_galore {input} -o ../results/trimmed/ --paired --cores {threads} &> {log}"
    +       "trim_galore {input} -o ../results/trimmed/ {params} --cores {threads} &> {log}"
    ```
    
??? file-code "Current snakefile:"

    ```python
    # define samples from data directory using wildcards
    SAMPLES, = glob_wildcards("../../data/{sample}_1.fastq.gz")
    
    # target OUTPUT files for the whole workflow
    rule all:
        input:
            "../results/multiqc_report.html",
            expand(["../results/trimmed/{sample}_1_val_1.fq.gz", "../results/trimmed/{sample}_2_val_2.fq.gz"], sample = SAMPLES)
    
    # workflow
    rule fastqc:
        input:
            R1 = "../../data/{sample}_1.fastq.gz",
            R2 = "../../data/{sample}_2.fastq.gz"
        output:
            html = ["../results/fastqc/{sample}_1_fastqc.html", "../results/fastqc/{sample}_2_fastqc.html"],
            zip = ["../results/fastqc/{sample}_1_fastqc.zip", "../results/fastqc/{sample}_2_fastqc.zip"]
        log:
            "logs/fastqc/{sample}.log"
        threads: 2
        conda:
            "envs/fastqc.yaml"
        shell:
            "fastqc {input.R1} {input.R2} -o ../results/fastqc/ -t {threads} &> {log}"
      
    rule multiqc:
        input:
            expand(["../results/fastqc/{sample}_1_fastqc.zip", "../results/fastqc/{sample}_2_fastqc.zip"], sample = SAMPLES)
        output:
            "../results/multiqc_report.html"
        log:
            "logs/multiqc/multiqc.log"
        conda:
            "envs/multiqc.yaml"
        shell:
            "multiqc {input} -o ../results/ &> {log}"
    
    rule trim_galore:
        input:
            ["../../data/{sample}_1.fastq.gz", "../../data/{sample}_2.fastq.gz"]
        output:
            ["../results/trimmed/{sample}_1_val_1.fq.gz", "../results/trimmed/{sample}_2_val_2.fq.gz"]
        params:
            "--paired"
        log:
            "logs/trim_galore/{sample}.log"
        conda:
            "envs/trimgalore.yaml"
        threads: 2
        resources:
            cpus_per_task=8
        shell:
            "trim_galore {input} -o ../results/trimmed/ {params} --cores {threads} &> {log}"
    ```

<br>

!!! terminal-2 "Run a dryrun to check it works"

    ```bash
    # remove output of last run
    rm -r ../results/*
    ```
    ```
    # run dryrun again
    snakemake --dryrun --workflow-profile slurm --software-deployment-method conda
    ```

## 4.3 Pull out user configurable options

We can separate the user configurable options away from the workflow. This supports reproducibility by minimising the chance the user makes changes to the core workflow.

Create a configuration file in a new directory `config/`

File structure:

```bash
demo_workflow/
      |_______results/
      |_______workflow/
      |          |_______logs/
      |          |_______slurm/
      |          |_______Snakefile
      |          |_______status.py
      |_______config
                 |_______config.yaml
```
!!! terminal "code"

    ```bash
    # create config directory
    mkdir ../config
    ```
    ```bash
    # create configuration file
    touch ../config/config.yaml
    ```

Now we need to pull out the parameters the user would likely need to configure. Let's give the user the option to pass any parameters they like to fastqc. In our `../config/config.yaml` file, add the configuration options and add a couple flags to be passed to fastqc and multiqc:

```yaml
# set software parameters for...
PARAMS:
  # ... fastqc
  FASTQC: "--kmers 5"
  # ... multiqc
  MULTIQC: "--flat"
```

??? code-compare "Edit snakefile : In the Snakefile, tell Snakemake to grab the variables `PARAMS` from `../config/config.yaml`"

    ```diff
    # define samples from data directory using wildcards
    SAMPLES, = glob_wildcards("../../data/{sample}_1.fastq.gz")
    
    # target OUTPUT files for the whole workflow
    rule all:
        input:
            "../results/multiqc_report.html",
            expand(["../results/trimmed/{sample}_1_val_1.fq.gz", "../results/trimmed/{sample}_2_val_2.fq.gz"], sample = SAMPLES)
    
    # workflow
    rule fastqc:
        input:
            R1 = "../../data/{sample}_1.fastq.gz",
            R2 = "../../data/{sample}_2.fastq.gz"
        output:
            html = ["../results/fastqc/{sample}_1_fastqc.html", "../results/fastqc/{sample}_2_fastqc.html"],
            zip = ["../results/fastqc/{sample}_1_fastqc.zip", "../results/fastqc/{sample}_2_fastqc.zip"]
    +   params:
    +       fastqc_params = config['PARAMS']['FASTQC']
        log:
            "logs/fastqc/{sample}.log"
        threads: 2
        conda:
            "envs/fastqc.yaml"
        shell:
    -       "fastqc {input.R1} {input.R2} -o ../results/fastqc/ -t {threads} &> {log}"
    +       "fastqc {input.R1} {input.R2} -o ../results/fastqc/ -t {threads} {params.fastqc_params} &> {log}"
      
    rule multiqc:
        input:
            expand(["../results/fastqc/{sample}_1_fastqc.zip", "../results/fastqc/{sample}_2_fastqc.zip"], sample = SAMPLES)
        output:
            "../results/multiqc_report.html"
    +   params:
    +       multiqc_params = config['PARAMS']['MULTIQC']
        log:
            "logs/multiqc/multiqc.log"
        conda:
            "envs/multiqc.yaml"
        shell:
    -       "multiqc {input} -o ../results/ &> {log}"
    +       "multiqc {input} -o ../results/ {params.multiqc_params} &> {log}"
    
    rule trim_galore:
        input:
            ["../../data/{sample}_1.fastq.gz", "../../data/{sample}_2.fastq.gz"]
        output:
            ["../results/trimmed/{sample}_1_val_1.fq.gz", "../results/trimmed/{sample}_2_val_2.fq.gz"]
        params:
            "--paired"
        log:
            "logs/trim_galore/{sample}.log"
        conda:
            "envs/trimgalore.yaml"
        threads: 2
        resources:
            cpus_per_task=8
        shell:
            "trim_galore {input} -o ../results/trimmed/ {params} --cores {threads} &> {log}"
    ```
    
??? file-code "Current snakefile"

    ```python
    # define samples from data directory using wildcards
    SAMPLES, = glob_wildcards("../../data/{sample}_1.fastq.gz")
    
    # target OUTPUT files for the whole workflow
    rule all:
        input:
            "../results/multiqc_report.html",
            expand(["../results/trimmed/{sample}_1_val_1.fq.gz", "../results/trimmed/{sample}_2_val_2.fq.gz"], sample = SAMPLES)
    
    # workflow
    rule fastqc:
        input:
            R1 = "../../data/{sample}_1.fastq.gz",
            R2 = "../../data/{sample}_2.fastq.gz"
        output:
            html = ["../results/fastqc/{sample}_1_fastqc.html", "../results/fastqc/{sample}_2_fastqc.html"],
            zip = ["../results/fastqc/{sample}_1_fastqc.zip", "../results/fastqc/{sample}_2_fastqc.zip"]
        params:
            fastqc_params = config['PARAMS']['FASTQC']
        log:
            "logs/fastqc/{sample}.log"
        threads: 2
        conda:
            "envs/fastqc.yaml"
        shell:
            "fastqc {input.R1} {input.R2} -o ../results/fastqc/ -t {threads} {params.fastqc_params} &> {log}"
      
    rule multiqc:
        input:
            expand(["../results/fastqc/{sample}_1_fastqc.zip", "../results/fastqc/{sample}_2_fastqc.zip"], sample = SAMPLES)
        output:
            "../results/multiqc_report.html"
        params:
            multiqc_params = config['PARAMS']['MULTIQC']
        log:
            "logs/multiqc/multiqc.log"
        conda:
            "envs/multiqc.yaml"
        shell:
            "multiqc {input} -o ../results/ {params.multiqc_params} &> {log}"
    
    rule trim_galore:
        input:
            ["../../data/{sample}_1.fastq.gz", "../../data/{sample}_2.fastq.gz"]
        output:
            ["../results/trimmed/{sample}_1_val_1.fq.gz", "../results/trimmed/{sample}_2_val_2.fq.gz"]
        params:
            "--paired"
        log:
            "logs/trim_galore/{sample}.log"
        conda:
            "envs/trimgalore.yaml"
        threads: 2
        resources:
            cpus_per_task=8
        shell:
            "trim_galore {input} -o ../results/trimmed/ {params} --cores {threads} &> {log}"
    ```

<br>

!!! terminal-2 "Let's use our configuration file! Run workflow again:"

    ```bash
    # remove output of last run
    rm -r ../results/*
    ```
    ```bash
    # run dryrun/run again
    snakemake --dryrun --workflow-profile slurm --software-deployment-method conda
    ```
    ```bash
    snakemake --workflow-profile slurm --software-deployment-method conda
    ```

    ??? failure "Didn't work? My error:"
    
        ```bash
        KeyError in line 19 of /scale_wlg_persistent/filesets/project/nesi99991/snakemake20220512/lkemp/snakemake_workshop/demo_workflow/workflow/Snakefile:
        'PARAMS'
          File "/scale_wlg_persistent/filesets/project/nesi99991/snakemake20220512/lkemp/snakemake_workshop/demo_workflow/workflow/Snakefile", line 19, in <module>
        ```

<br>

Snakemake can't find our 'Key' - we haven't told Snakemake where our config file is so it can't find our config variables. We can do this by passing the location of our config file to the `--configfile` flag

!!! terminal "code"

    ```bash
    # remove output of last run
    rm -r ../results/
    ```
    ```diff
    # run dryrun/run again
    - snakemake --dryrun --workflow-profile slurm --software-deployment-method conda
    - snakemake --workflow-profile slurm --software-deployment-method conda
    + snakemake --dryrun --workflow-profile slurm --software-deployment-method conda --configfile ../config/config.yaml
    + snakemake --workflow-profile slurm --software-deployment-method conda --configfile ../config/config.yaml
    ```

Alternatively, we can define our config file in our Snakefile in a situation where the configuration file is likely to always be named the same and be in the exact same location `../config/config.yaml` and you don't need the flexibility for the user to specify their own configuration files:

??? code-compare "Edit snakefile"

    ```diff
    # define our configuration file
    + configfile: "../config/config.yaml"
    
    # define samples from data directory using wildcards
    SAMPLES, = glob_wildcards("../../data/{sample}_1.fastq.gz")
    
    # target OUTPUT files for the whole workflow
    rule all:
        input:
            "../results/multiqc_report.html",
            expand(["../results/trimmed/{sample}_1_val_1.fq.gz", "../results/trimmed/{sample}_2_val_2.fq.gz"], sample = SAMPLES)
    
    # workflow
    rule fastqc:
        input:
            R1 = "../../data/{sample}_1.fastq.gz",
            R2 = "../../data/{sample}_2.fastq.gz"
        output:
            html = ["../results/fastqc/{sample}_1_fastqc.html", "../results/fastqc/{sample}_2_fastqc.html"],
            zip = ["../results/fastqc/{sample}_1_fastqc.zip", "../results/fastqc/{sample}_2_fastqc.zip"]
        params:
            fastqc_params = config['PARAMS']['FASTQC']
        log:
            "logs/fastqc/{sample}.log"
        threads: 2
        conda:
            "envs/fastqc.yaml"
        shell:
            "fastqc {input.R1} {input.R2} -o ../results/fastqc/ -t {threads} {params.fastqc_params} &> {log}"
      
    rule multiqc:
        input:
            expand(["../results/fastqc/{sample}_1_fastqc.zip", "../results/fastqc/{sample}_2_fastqc.zip"], sample = SAMPLES)
        output:
            "../results/multiqc_report.html"
        params:
            multiqc_params = config['PARAMS']['MULTIQC']
        log:
            "logs/multiqc/multiqc.log"
        conda:
            "envs/multiqc.yaml"
        shell:
            "multiqc {input} -o ../results/ {params.multiqc_params} &> {log}"
    
    rule trim_galore:
        input:
            ["../../data/{sample}_1.fastq.gz", "../../data/{sample}_2.fastq.gz"]
        output:
            ["../results/trimmed/{sample}_1_val_1.fq.gz", "../results/trimmed/{sample}_2_val_2.fq.gz"]
        params:
            "--paired"
        log:
            "logs/trim_galore/{sample}.log"
        conda:
            "envs/trimgalore.yaml"
        threads: 2
        resources:
            cpus_per_task=8
        shell:
            "trim_galore {input} -o ../results/trimmed/ {params} --cores {threads} &> {log}"
    ```
    
??? file-code "Current snakefile:"

    ```python
    # define our configuration file
    configfile: "../config/config.yaml"
    
    # define samples from data directory using wildcards
    SAMPLES, = glob_wildcards("../../data/{sample}_1.fastq.gz")
    
    # target OUTPUT files for the whole workflow
    rule all:
        input:
            "../results/multiqc_report.html",
            expand(["../results/trimmed/{sample}_1_val_1.fq.gz", "../results/trimmed/{sample}_2_val_2.fq.gz"], sample = SAMPLES)
    
    # workflow
    rule fastqc:
        input:
            R1 = "../../data/{sample}_1.fastq.gz",
            R2 = "../../data/{sample}_2.fastq.gz"
        output:
            html = ["../results/fastqc/{sample}_1_fastqc.html", "../results/fastqc/{sample}_2_fastqc.html"],
            zip = ["../results/fastqc/{sample}_1_fastqc.zip", "../results/fastqc/{sample}_2_fastqc.zip"]
        params:
            fastqc_params = config['PARAMS']['FASTQC']
        log:
            "logs/fastqc/{sample}.log"
        threads: 2
        conda:
            "envs/fastqc.yaml"
        shell:
            "fastqc {input.R1} {input.R2} -o ../results/fastqc/ -t {threads} {params.fastqc_params} &> {log}"
      
    rule multiqc:
        input:
            expand(["../results/fastqc/{sample}_1_fastqc.zip", "../results/fastqc/{sample}_2_fastqc.zip"], sample = SAMPLES)
        output:
            "../results/multiqc_report.html"
        params:
            multiqc_params = config['PARAMS']['MULTIQC']
        log:
            "logs/multiqc/multiqc.log"
        conda:
            "envs/multiqc.yaml"
        shell:
            "multiqc {input} -o ../results/ {params.multiqc_params} &> {log}"
    
    rule trim_galore:
        input:
            ["../../data/{sample}_1.fastq.gz", "../../data/{sample}_2.fastq.gz"]
        output:
            ["../results/trimmed/{sample}_1_val_1.fq.gz", "../results/trimmed/{sample}_2_val_2.fq.gz"]
        params:
            "--paired"
        log:
            "logs/trim_galore/{sample}.log"
        conda:
            "envs/trimgalore.yaml"
        threads: 2
        resources:
            cpus_per_task=8
        shell:
            "trim_galore {input} -o ../results/trimmed/ {params} --cores {threads} &> {log}"
    ```

<br>

Then we don't need to specify where the configuration file is on the command line

!!! terminal "code"

    ```bash
    # remove output of last run
    rm -r ../results/*
    ```
    ```diff 
    # run dryrun/run again
    - snakemake --dryrun --workflow-profile slurm --software-deployment-method conda --configfile ../config/config.yaml
    - snakemake --workflow-profile slurm --software-deployment-method conda --configfile ../config/config.yaml
    + snakemake --dryrun --workflow-profile slurm --software-deployment-method conda
    + snakemake --workflow-profile slurm --software-deployment-method conda
    ```

## 4.4 Leave messages for the user

We can provide the user of our workflow more information on what is happening at each stage/rule of our workflow via the `message:` directive. We are able to call many variables such as:

- Input and output files `{input}` and `{output}`
- Specific input and output files such as `{input.R1}`
- Our `{params}`, `{log}` and `{threads}` directives

??? code-compare "Edit snakefile"

    ```diff
    # define our configuration file
    configfile: "../config/config.yaml"
    
    # define samples from data directory using wildcards
    SAMPLES, = glob_wildcards("../../data/{sample}_1.fastq.gz")
    
    # target OUTPUT files for the whole workflow
    rule all:
        input:
            "../results/multiqc_report.html",
            expand(["../results/trimmed/{sample}_1_val_1.fq.gz", "../results/trimmed/{sample}_2_val_2.fq.gz"], sample = SAMPLES)
    
    # workflow
    rule fastqc:
        input:
            R1 = "../../data/{sample}_1.fastq.gz",
            R2 = "../../data/{sample}_2.fastq.gz"
        output:
            html = ["../results/fastqc/{sample}_1_fastqc.html", "../results/fastqc/{sample}_2_fastqc.html"],
            zip = ["../results/fastqc/{sample}_1_fastqc.zip", "../results/fastqc/{sample}_2_fastqc.zip"]
        params:
            fastqc_params = config['PARAMS']['FASTQC']
        log:
            "logs/fastqc/{sample}.log"
        threads: 2
        conda:
            "envs/fastqc.yaml"
    +   message:
    +       "Undertaking quality control checks {input}"
        shell:
            "fastqc {input.R1} {input.R2} -o ../results/fastqc/ -t {threads} {params.fastqc_params} &> {log}"
      
    rule multiqc:
        input:
            expand(["../results/fastqc/{sample}_1_fastqc.zip", "../results/fastqc/{sample}_2_fastqc.zip"], sample = SAMPLES)
        output:
            "../results/multiqc_report.html"
        params:
            multiqc_params = config['PARAMS']['MULTIQC']
        log:
            "logs/multiqc/multiqc.log"
        conda:
            "envs/multiqc.yaml"
    +   message:
    +       "Compiling a HTML report for quality control checks. Writing to {output}."
        shell:
            "multiqc {input} -o ../results/ {params.multiqc_params} &> {log}"
    
    rule trim_galore:
        input:
            ["../../data/{sample}_1.fastq.gz", "../../data/{sample}_2.fastq.gz"]
        output:
            ["../results/trimmed/{sample}_1_val_1.fq.gz", "../results/trimmed/{sample}_2_val_2.fq.gz"]
        params:
            "--paired"
        log:
            "logs/trim_galore/{sample}.log"
        conda:
            "envs/trimgalore.yaml"
        threads: 2
        resources:
            cpus_per_task=8
    +   message:
    +       "Trimming using these parameter: {params}. Writing logs to {log}. Using {threads} threads."
        shell:
            "trim_galore {input} -o ../results/trimmed/ {params} --cores {threads} &> {log}"
    ```
    
??? file-code "Current snakefile:"

    ```python
    # define our configuration file
    configfile: "../config/config.yaml"
    
    # define samples from data directory using wildcards
    SAMPLES, = glob_wildcards("../../data/{sample}_1.fastq.gz")
    
    # target OUTPUT files for the whole workflow
    rule all:
        input:
            "../results/multiqc_report.html",
            expand(["../results/trimmed/{sample}_1_val_1.fq.gz", "../results/trimmed/{sample}_2_val_2.fq.gz"], sample = SAMPLES)
    
    # workflow
    rule fastqc:
        input:
            R1 = "../../data/{sample}_1.fastq.gz",
            R2 = "../../data/{sample}_2.fastq.gz"
        output:
            html = ["../results/fastqc/{sample}_1_fastqc.html", "../results/fastqc/{sample}_2_fastqc.html"],
            zip = ["../results/fastqc/{sample}_1_fastqc.zip", "../results/fastqc/{sample}_2_fastqc.zip"]
        params:
            fastqc_params = config['PARAMS']['FASTQC']
        log:
            "logs/fastqc/{sample}.log"
        threads: 2
        conda:
            "envs/fastqc.yaml"
        message:
            "Undertaking quality control checks {input}"
        shell:
            "fastqc {input.R1} {input.R2} -o ../results/fastqc/ -t {threads} {params.fastqc_params} &> {log}"
      
    rule multiqc:
        input:
            expand(["../results/fastqc/{sample}_1_fastqc.zip", "../results/fastqc/{sample}_2_fastqc.zip"], sample = SAMPLES)
        output:
            "../results/multiqc_report.html"
        params:
            multiqc_params = config['PARAMS']['MULTIQC']
        log:
            "logs/multiqc/multiqc.log"
        conda:
            "envs/multiqc.yaml"
        message:
            "Compiling a HTML report for quality control checks. Writing to {output}."
        shell:
            "multiqc {input} -o ../results/ {params.multiqc_params} &> {log}"
    
    rule trim_galore:
        input:
            ["../../data/{sample}_1.fastq.gz", "../../data/{sample}_2.fastq.gz"]
        output:
            ["../results/trimmed/{sample}_1_val_1.fq.gz", "../results/trimmed/{sample}_2_val_2.fq.gz"]
        params:
            "--paired"
        log:
            "logs/trim_galore/{sample}.log"
        conda:
            "envs/trimgalore.yaml"
        threads: 2
        resources:
            cpus_per_task=8
        message:
            "Trimming using these parameter: {params}. Writing logs to {log}. Using {threads} threads."
        shell:
            "trim_galore {input} -o ../results/trimmed/ {params} --cores {threads} &> {log}"
    ```
    
<br>

!!! terminal "code"

    ```diff
    # remove output of last run
    rm -r ../results/*
    ```

    ```bash
    # run dryrun/run again
    snakemake --dryrun --workflow-profile slurm --software-deployment-method conda
    snakemake --workflow-profile slurm --software-deployment-method conda
    ```
    
    - Now our messages are printed to the screen as our workflow runs

    ??? success "output"

        ```bash
        Building DAG of jobs...
        Using shell: /usr/bin/bash
        Provided cluster nodes: 20
        Job stats:
        job            count    min threads    max threads
        -----------  -------  -------------  -------------
        all                1              1              1
        fastqc             3              2              2
        multiqc            1              1              1
        trim_galore        3              2              2
        total              8              1              2
        
        Select jobs to execute...
        
        [Wed May 11 13:20:52 2022]
        Job 4: Undertaking quality control checks ../../data/NA24694_1.fastq.gz ../../data/NA24694_2.fastq.gz
        
        Submitted job 4 with external jobid '26763840'.
        
        [Wed May 11 13:20:52 2022]
        Job 6: Trimming using these parameter: --paired. Writing logs to logs/trim_galore/NA24695.log. Using 2 threads.
        
        Submitted job 6 with external jobid '26763841'.
        
        [Wed May 11 13:20:52 2022]
        Job 2: Undertaking quality control checks ../../data/NA24631_1.fastq.gz ../../data/NA24631_2.fastq.gz
        
        Submitted job 2 with external jobid '26763842'.
        
        [Wed May 11 13:20:52 2022]
        Job 3: Undertaking quality control checks ../../data/NA24695_1.fastq.gz ../../data/NA24695_2.fastq.gz
        
        Submitted job 3 with external jobid '26763843'.
        
        [Wed May 11 13:20:52 2022]
        Job 5: Trimming using these parameter: --paired. Writing logs to logs/trim_galore/NA24631.log. Using 2 threads.
        
        Submitted job 5 with external jobid '26763844'.
        
        [Wed May 11 13:20:52 2022]
        Job 7: Trimming using these parameter: --paired. Writing logs to logs/trim_galore/NA24694.log. Using 2 threads.
        
        Submitted job 7 with external jobid '26763845'.
        [Wed May 11 13:22:08 2022]
        Finished job 4.
        1 of 8 steps (12%) done
        [Wed May 11 13:22:08 2022]
        Finished job 6.
        2 of 8 steps (25%) done
        [Wed May 11 13:22:08 2022]
        Finished job 2.
        3 of 8 steps (38%) done
        [Wed May 11 13:22:08 2022]
        Finished job 3.
        4 of 8 steps (50%) done
        Select jobs to execute...
        
        [Wed May 11 13:22:09 2022]
        Job 1: Compiling a HTML report for quality control checks. Writing to ../results/multiqc_report.html.
        
        Submitted job 1 with external jobid '26763848'.
        [Wed May 11 13:22:09 2022]
        Finished job 5.
        5 of 8 steps (62%) done
        [Wed May 11 13:22:09 2022]
        Finished job 7.
        6 of 8 steps (75%) done
        [Wed May 11 13:24:21 2022]
        Finished job 1.
        7 of 8 steps (88%) done
        Select jobs to execute...
        
        [Wed May 11 13:24:21 2022]
        localrule all:
            input: ../results/multiqc_report.html, ../results/trimmed/NA24631_1_val_1.fq.gz, ../results/trimmed/NA24695_1_val_1.fq.gz, ../results/trimmed/NA24694_1_val_1.fq.gz, ../results/trimmed/NA24631_2_val_2.fq.gz, ../results/trimmed/NA24695_2_val_2.fq.gz, ../results/trimmed/NA24694_2_val_2.fq.gz
            jobid: 0
            resources: mem_mb=512, disk_mb=1000, tmpdir=/dev/shm/jobs/26763281, cpus=2, time_min=10
        
        [Wed May 11 13:24:21 2022]
        Finished job 0.
        8 of 8 steps (100%) done
        Complete log: .snakemake/log/2022-05-11T132052.454902.snakemake.log
        ```


<br>

## 4.5 Create temporary files

In our workflow, we are likely to be creating files that we don't want, but are used or produced by our workflow (intermediate files). We can mark such files as temporary so Snakemake will remove the file once it doesn't need to use it anymore.

For example, we might not want to keep our fastqc output files since our multiqc report merges all of our fastqc reports for each sample into one report. Let's have a look at the files currently produced by our workflow with:

!!! terminal "code"

    ```bash
    ls -lh ../results/fastqc/
    ```

    ??? success "output"

        ```bash
        total 4.5M
        -rw-rw----+ 1 lkemp nesi99991 250K May 11 13:22 NA24631_1_fastqc.html
        -rw-rw----+ 1 lkemp nesi99991 327K May 11 13:22 NA24631_1_fastqc.zip
        -rw-rw----+ 1 lkemp nesi99991 249K May 11 13:22 NA24631_2_fastqc.html
        -rw-rw----+ 1 lkemp nesi99991 327K May 11 13:22 NA24631_2_fastqc.zip
        -rw-rw----+ 1 lkemp nesi99991 254K May 11 13:22 NA24694_1_fastqc.html
        -rw-rw----+ 1 lkemp nesi99991 334K May 11 13:22 NA24694_1_fastqc.zip
        -rw-rw----+ 1 lkemp nesi99991 250K May 11 13:22 NA24694_2_fastqc.html
        -rw-rw----+ 1 lkemp nesi99991 328K May 11 13:22 NA24694_2_fastqc.zip
        -rw-rw----+ 1 lkemp nesi99991 252K May 11 13:22 NA24695_1_fastqc.html
        -rw-rw----+ 1 lkemp nesi99991 328K May 11 13:22 NA24695_1_fastqc.zip
        -rw-rw----+ 1 lkemp nesi99991 253K May 11 13:22 NA24695_2_fastqc.html
        -rw-rw----+ 1 lkemp nesi99991 330K May 11 13:22 NA24695_2_fastqc.zip
        ```

<br>

Let's mark all the trimmed fastq files as temporary in our Snakefile by wrapping it up in the `temp()` function

??? code-compare "Edit snakefile"
    
    ```diff
    # define our configuration file
    configfile: "../config/config.yaml"
    
    # define samples from data directory using wildcards
    SAMPLES, = glob_wildcards("../../data/{sample}_1.fastq.gz")
    
    # target OUTPUT files for the whole workflow
    rule all:
        input:
            "../results/multiqc_report.html",
            expand(["../results/trimmed/{sample}_1_val_1.fq.gz", "../results/trimmed/{sample}_2_val_2.fq.gz"], sample = SAMPLES)
    
    # workflow
    rule fastqc:
        input:
            R1 = "../../data/{sample}_1.fastq.gz",
            R2 = "../../data/{sample}_2.fastq.gz"
        output:
    -       html = ["../results/fastqc/{sample}_1_fastqc.html", "../results/fastqc/{sample}_2_fastqc.html"],
    +       html = temp(["../results/fastqc/{sample}_1_fastqc.html", "../results/fastqc/{sample}_2_fastqc.html"]),
            zip = ["../results/fastqc/{sample}_1_fastqc.zip", "../results/fastqc/{sample}_2_fastqc.zip"]
        params:
            fastqc_params = config['PARAMS']['FASTQC']
        log:
            "logs/fastqc/{sample}.log"
        threads: 2
        conda:
            "envs/fastqc.yaml"
        message:
            "Undertaking quality control checks {input}"
        shell:
            "fastqc {input.R1} {input.R2} -o ../results/fastqc/ -t {threads} {params.fastqc_params} &> {log}"
      
    rule multiqc:
        input:
            expand(["../results/fastqc/{sample}_1_fastqc.zip", "../results/fastqc/{sample}_2_fastqc.zip"], sample = SAMPLES)
        output:
            "../results/multiqc_report.html"
        params:
            multiqc_params = config['PARAMS']['MULTIQC']
        log:
            "logs/multiqc/multiqc.log"
        conda:
            "envs/multiqc.yaml"
        message:
            "Compiling a HTML report for quality control checks. Writing to {output}."
        shell:
            "multiqc {input} -o ../results/ {params.multiqc_params} &> {log}"
    
    rule trim_galore:
        input:
            ["../../data/{sample}_1.fastq.gz", "../../data/{sample}_2.fastq.gz"]
        output:
            ["../results/trimmed/{sample}_1_val_1.fq.gz", "../results/trimmed/{sample}_2_val_2.fq.gz"]
        params:
            "--paired"
        log:
            "logs/trim_galore/{sample}.log"
        conda:
            "envs/trimgalore.yaml"
        threads: 2
        resources:
            cpus_per_task=8
        message:
            "Trimming using these parameter: {params}. Writing logs to {log}. Using {threads} threads."
        shell:
            "trim_galore {input} -o ../results/trimmed/ {params} --cores {threads} &> {log}"
    ```

??? file-code "Current snakefile:"

    ```python
    # define our configuration file
    configfile: "../config/config.yaml"
    
    # define samples from data directory using wildcards
    SAMPLES, = glob_wildcards("../../data/{sample}_1.fastq.gz")
    
    # target OUTPUT files for the whole workflow
    rule all:
        input:
            "../results/multiqc_report.html",
            expand(["../results/trimmed/{sample}_1_val_1.fq.gz", "../results/trimmed/{sample}_2_val_2.fq.gz"], sample = SAMPLES)
    
    # workflow
    rule fastqc:
        input:
            R1 = "../../data/{sample}_1.fastq.gz",
            R2 = "../../data/{sample}_2.fastq.gz"
        output:
            html = temp(["../results/fastqc/{sample}_1_fastqc.html", "../results/fastqc/{sample}_2_fastqc.html"]),
            zip = ["../results/fastqc/{sample}_1_fastqc.zip", "../results/fastqc/{sample}_2_fastqc.zip"]
        params:
            fastqc_params = config['PARAMS']['FASTQC']
        log:
            "logs/fastqc/{sample}.log"
        threads: 2
        conda:
            "envs/fastqc.yaml"
        message:
            "Undertaking quality control checks {input}"
        shell:
            "fastqc {input.R1} {input.R2} -o ../results/fastqc/ -t {threads} {params.fastqc_params} &> {log}"
      
    rule multiqc:
        input:
            expand(["../results/fastqc/{sample}_1_fastqc.zip", "../results/fastqc/{sample}_2_fastqc.zip"], sample = SAMPLES)
        output:
            "../results/multiqc_report.html"
        params:
            multiqc_params = config['PARAMS']['MULTIQC']
        log:
            "logs/multiqc/multiqc.log"
        conda:
            "envs/multiqc.yaml"
        message:
            "Compiling a HTML report for quality control checks. Writing to {output}."
        shell:
            "multiqc {input} -o ../results/ {params.multiqc_params} &> {log}"
    
    rule trim_galore:
        input:
            ["../../data/{sample}_1.fastq.gz", "../../data/{sample}_2.fastq.gz"]
        output:
            ["../results/trimmed/{sample}_1_val_1.fq.gz", "../results/trimmed/{sample}_2_val_2.fq.gz"]
        params:
            "--paired"
        log:
            "logs/trim_galore/{sample}.log"
        conda:
            "envs/trimgalore.yaml"
        threads: 2
        resources:
            cpus_per_task=8
        message:
            "Trimming using these parameter: {params}. Writing logs to {log}. Using {threads} threads."
        shell:
            "trim_galore {input} -o ../results/trimmed/ {params} --cores {threads} &> {log}"
    ```
    
    
<br>

!!! terminal "code"

    ```diff
    # remove output of last run
    rm -r ../results/*
    ```
    ```bash    
    # run dryrun/run again
    snakemake --dryrun --workflow-profile slurm --software-deployment-method conda
    snakemake --workflow-profile slurm --software-deployment-method conda
    ```

!!! terminal-2 "Now when we have a look at the `../results/fastqc/` directory with:"

    ```bash
    ls -lh ../results/fastqc/
    ```

    - These html files have been removed once Snakemake no longer needs the files for another rule/operation, and we've saved some space on our computer (from 4.5 megabytes to 3 megabytes in this directory).

    ??? success "output"

        ```bash
        total 3.0M
        -rw-rw----+ 1 lkemp nesi99991 327K May 11 13:26 NA24631_1_fastqc.zip
        -rw-rw----+ 1 lkemp nesi99991 327K May 11 13:26 NA24631_2_fastqc.zip
        -rw-rw----+ 1 lkemp nesi99991 334K May 11 13:26 NA24694_1_fastqc.zip
        -rw-rw----+ 1 lkemp nesi99991 328K May 11 13:26 NA24694_2_fastqc.zip
        -rw-rw----+ 1 lkemp nesi99991 328K May 11 13:26 NA24695_1_fastqc.zip
        -rw-rw----+ 1 lkemp nesi99991 330K May 11 13:26 NA24695_2_fastqc.zip
        ```

<br>

!!! note ""

    This becomes particularly important when our data become big data, since we don't want to keep any massive intermediate output files that we don't need. Otherwise this can start to clog up the memory on our computer. It ensures our workflow is scalable when our data becomes big data.

## 4.6 Generating a snakemake report

With Snakemake, we can automatically generate detailed self-contained HTML reports after we run our workflow with the following command:

!!! terminal "code"

    ```bash
    snakemake --report ../results/snakemake_report.html
    ```

!!! note "Note"
    You won't be able to view a rendered version of this html while it is on the remote server, however after you transfer it to your local computer you should be able to view it in your web browser.

In our report:

- We get an interactive version of our directed acyclic graph (DAG).
- When you click on a node in the DAG, the input and output files are fully outlined, the exact software used and the exact shell command that was run.
- You are also provided with runtime information under the `Statistics` tab outlining how long each rule/sample ran for, and the date/time each file was created.

!!! html5 "My report"

    ![snakemake_report](./images/snakemake_report.gif)


<br>

These reports are highly configurable, have a look at an example of what can be done with a report [here](https://koesterlab.github.io/resources/report.html)

*See more information on creating Snakemake reports [in the Snakemake documentation](https://snakemake.readthedocs.io/en/stable/snakefiles/reporting.html)*

## 4.7 Linting your workflow

Snakemake has a built in linter to support you building best practice workflows, let's try it out:

!!! terminal "code"

    ```bash
    snakemake --lint
    ```

    ??? success "output"

        ```bash
        Congratulations, your workflow is in a good condition!
        ```


<br>

Writing a best practice workflow is more important than having Marie Kondo level tidiness, it increases the chance your workflow will continue to be used and maintained by others (and ourselves), making the code we write useful (it's exciting seeing someone else using your code!). If your workflow was used in scientific research, it makes your workflow accessible for people to reproduce your research findings; it isn't going to be a nightmare for them to run and they are more likely to try and have success doing so.

Read more about the [best practices for Snakemake](https://snakemake.readthedocs.io/en/stable/snakefiles/best_practices.html)

# Takeaways

---
!!! quote ""

    - Pull out your parameters and put them in `params:` directive
    - Pulling the user configurable options away from the core workflow will support reproducibility by reducing the chance of changes to the core workflow
    - Leaving messages for the user of your workflow will help them understand what is happening at each stage and follow the workflows progress
    - Mark files you won't need once the workflow completes to reduce the memory usage - *particularly* when dealing with big data
    - Generate a snakemake report to get a summary of the workflow run - these are highly configurable
    - Lint your workflow and check it complies with best practices - this supports reproducibility and portability
    - There is so much more to explore, such as creating [modular workflows](https://snakemake.readthedocs.io/en/stable/snakefiles/modularization.html), automatically grabbing [remote files](https://snakemake.readthedocs.io/en/stable/snakefiles/remote_files.html) from places like Google Cloud Storage and Dropbox, [run various types of scripts](https://snakemake.readthedocs.io/en/stable/snakefiles/rules.html#external-scripts) such as [python scripts](https://snakemake.readthedocs.io/en/stable/snakefiles/rules.html#python), [R and RMarkdown scripts](https://snakemake.readthedocs.io/en/stable/snakefiles/rules.html#r-and-r-markdown) and [Jupyter Notebooks](https://snakemake.readthedocs.io/en/stable/snakefiles/rules.html#jupyter-notebook-integration)
    
---

# Summary commands

!!! terminal ""

    - Use the parameter directive (`params`) to keep the parameters and flags of your programs separate from your shell command, for example:
    
    ```bash
    params:
        "--paired"
    ```
    
    - Run your snakemake workflow (using environment modules to load your software AND with a configuration file) with:
    
    ```bash
    snakemake --cores 2 --software-deployment-method conda --configfile ../config/config.yaml
    ```
    
    - Alternatively, define your config file in the Snakefile:
    
    ```bash
    configfile: "../config/config.yaml"
    ```
    
    - Use the `message` directive to provide information to the user on what is happening real time, for example:
    
    ```bash
    message:
        "Undertaking quality control checks {input}"
    ```
    
    - Mark temporary files to remove (once they are no longer needed by the workflow) with `temp()`, for example:
    
    ```bash
    temp(["../results/trimmed/{sample}_1_val_1.fq.gz", "../results/trimmed/{sample}_2_val_2.fq.gz"])
    ```
    
    - Create a basic interactive Snakemake report after running your workflow with:
    
    ```bash
    snakemake --report ../results/snakemake_report.html
    ```

# Our final snakemake workflow!

See [leveled_up_demo_workflow](https://github.com/msu-cmse-courses/CMSE_890-602_snakemake/tree/main/leveled_up_demo_workflow) for the final Snakemake workflow we've created up to this point

- - - 


<p align="center"><b><a class="btn" href="https://msu-cmse-courses.github.io/CMSE_890-602_snakemake/" style="background: var(--bs-dark);font-weight:bold">Back to homepage</a></b></p>
