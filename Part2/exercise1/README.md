# Exercise 1: Command line run

## Objectives 
* View available parameters
* Add key inputs on the command line
* Explore workflow setup

---------------------
## Testing reflection
* FYI cvmfs was not auto-cached, will need to prepare VMs so users can access without needing to edit pub keys 
* FYI using the latest stable release and documentation (docs for earlier versions not available)
* TODO provide context to rnaseq workflow 
* TODO ask Audrey to update version of nextflow on bio image (for running 3.10.1, need nextflow 22.10.1). See issue below.
* TODO ask Audrey to install http://www.graphviz.org
* TODO change the names/paths to files (on cvmfs) for sake of tidiness 
* TODO confirm container path changes with Alex 
* TODO port all materials to cvmfs (cloudstor link as backup) 
* Q should max mem, max cpu flag parameters be an exercise with cpu and memory limits provided? 
* Q how to explain container tools available. Include: Singularity, Docker, Charliecloud, Podman, Shifter, gitpod?
* Q should we run -stub instead of whole command?
* DONE tested with most recent version of nf-core/rnaseq. Not using Nandan's nextflow.config 
* DONE re-created reference data and STAR indexes. New [Cloudstor link](https://cloudstor.aarnet.edu.au/plus/s/gIBdDhKEwfq2j58/download).

-----------------

## Content draft UPDATED

7/4/23 I am working through each exercise - not wanting to interfere with Georgie's flow, I will make additions/changes here for GS to incorporate:

### Updated exercsie 1 run command 

- Need to ensure that the samplesheet.csv file has the correct path to the fastqs based on final working directory structure
- Need to sed 'forward' with 'unstranded' within samplesheet.csv 
- Relative to GS command 
    - removed the dag/report/trace (rnaseq conf will put them in the params.outdir with timestamp so we dont need to worry about overwrite error/warn) 
    - replaced samples.tsv with samplesheet.csv
    - added mm10_reference to materials path 
    
```
nextflow run rnaseq/main.nf \
    --input $materials/samplesheet.csv \
    -profile singularity \
    --fasta $materials/mm10_reference/mm10_chr18.fa \
    --gtf $materials/mm10_reference/mm10_chr18.gtf \
    --star_index $materials/mm10_reference/STAR \
    --max_memory '6 GB' \
    --max_cpus 2 \
    --outdir Exercise_1 
```

### Singularity cache dir

- After the run has completed, show the location of the container image files in `./work/singularity`
    - This activity may be placed at the end of this exercise or the start of exercise 2
- Discuss how this is not an ideal location (we may need to periodically clean out the `work` directory to save disk space and inodes) 
- Run the following commands to move the images and set the cachedir:

```
mv ./work/singularity ~ 
ls ~/singularity
# Add to ~/.bashrc file then save:
export NXF_SINGULARITY_CACHEDIR=/home/ubuntu/singularity 
source ~/.bashrc 
echo $NXF_SINGULARITY_CACHEDIR
```

## Content draft 

You can find the parameters for each nf-core workflow on their respective documentation sites. The nf-core/rnaseq workflow parameters can be found [here](https://nf-co.re/rnaseq/3.10.1/parameters). In the case of the nf-core/rnaseq workflow, parameters are grouped based on various stages of the workflow: 

1. Input/output options for specifying which files to process and where to save results
2. UMI options for processing reads with unique molecular identifiers (UMI)
3. Read filtering options to be run prior to alignment 
4. Reference genome options related to pre-processing of the reference FASTA 
5. Read trimming options prior to alignment 
6. Alignment options for read mapping and filtering criteria 
7. Process skipping options for adjusting the processes to run with the workflow 

On the command line you can view these options by running: 
```
nextflow run ../rnaseq/main.nf --help 
```

Notice at the bottom of the print out, there is 
```
!! Hiding 24 params, use --show_hidden_params to show them !!
```

Three additional parameter sections are hidden from view. This is because they are less commonly used. They include: 

8. Institutional config options for various compute environments 
9. Max job request options for limiting memory and cpu usage based on what is available to you
10. Generic options focused on how the pipeline is run

To view all the workflow run options on the command line, run: 
```
nextflow run ../rnaseq/main.nf --help --show_hidden_params
```

We will be using a number of flags from each section today. In addition to the required parameters, we have chosen to use some additional flags that limit the compute resource usage of the pipeline, specify prepared fasta, gtf, and star index files, and run the workflow using Singularity. 

> **:fire: HOT TIP :fire:** 
Hyphens matter when it comes to parameter flags! Nextflow command-line parameters use one (-), whereas pipeline-specific parameters use two (--). For example: `-profile` is a Nextflow parameter, while `--input` is an nf-core parameter. 

**[INSERT A DIAGRAM OF THE COMMAND WHICH CATEGORISES/EXPLAINS EACH FLAG USED?]**

The instance we are working on today has fewer CPUs and memory available than the default run (16 CPUs, 128GB) settings. As such, we will be using the following flags:  
```
--max_memory '6.GB'
--max_cpus 2
```

From previous experience running STAR, we know it requires XX GB to generate the index files. For the sake of this workshop, we are working with subset files and have provided these along with previously generated STAR index files. These are stored on CVMFS at: 
```
cvmfs_path=/cvmfs/data.biocommons.aarnet.edu.au/Final_resources_250722/Mouse_chr18_reference
```

**[ACTIVITY TO CACHE CVMFS?]**

We will specify the use of this STAR index and other prepared reference files with the following flags:
```
--gtf $materials/mm10_chr18.gtf
--fasta $materials/Mouse_chr18_reference/chr18.fa
--star_index $materials/STAR
```


A nice feature of nf-core workflows is integrated software dependency management via container management software and/or conda environments. This saves us a lot of blood, sweat, and tears in installing all the tools run by this workflow and their dependencies. To run containers, nf-core requires we have either Docker or Singularity installed. These are container management tools and both are available on our Nimbus instances. By running the following command, we have instructed Nextflow to execute jobs on Nimbus using Singularity to fulfill software dependencies:
```
-profile singularity
```

Another nice feature of Nextflow is the ability to generate resource and runtime reports. These are produced by default (see `nextflow.config`), and are output to the defined tracedir with the file name and a timestamp: 

```
def trace_timestamp = new java.util.Date().format( 'yyyy-MM-dd_HH-mm-ss')
timeline {
    enabled = true
    file    = "${params.tracedir}/execution_timeline_${trace_timestamp}.html"
}
report {
    enabled = true
    file    = "${params.tracedir}/execution_report_${trace_timestamp}.html"
}
trace {
    enabled = true
    file    = "${params.tracedir}/execution_trace_${trace_timestamp}.txt"
}
dag {
    enabled = true
    file    = "${params.tracedir}/pipeline_dag_${trace_timestamp}.html"
}
```

So we can keep track of all the different exercises in today's workshop, we're going to opt to rename these reports at runtime, with the following flags:
```
-with-report report/execution_report_exercise1.html
-with-trace report/execution_trace_exercise1.txt
-with-timeline report/timeline_exercise1.html
-with-dag report/dag_exercise1.png
``` 

We've used a standard experimental design and do not need to adjust any additional parameters at this stage. Now that we have decided on the additional flags to use, run the workflow with the following:
```
materials=/home/ubuntu/nfcoreWorkshopTesting/materials/mm10_reference

nextflow run ../rnaseq/main.nf \
    --input ./materials/samplesheet.csv \
    --outdir /home/ubuntu/nfcoreWorkshopTesting/exercise1/results \
    --max_memory '6.GB' --max_cpus 2 \
    --gtf $materials/mm10_chr18.gtf \
    --fasta $materials/mm10_chr18.fa \
    --star_index $materials/STAR \
    -profile singularity \
    -with-report execution_report_exercise1.html \
    -with-trace execution_trace_exercise1.txt \
    -with-timeline timeline_exercise1.html \
    -with-dag dag_exercise1.png 
```

While the workflow runs (~20 mins), let's look at how it has been configured for a default run by looking at the `rnaseq/nextflow.config` file.

Also look at local nextflow.config [ Georgie please add this to download materials]
    - This config only has a set of parameters to overwrite some of the reporting files. This is useful if we re-run and want to overwrite the output files, eg if we have made an incorrect command. Without this, the nextflow command will fail (will fail only for dag and trace, but will warn for report and timeline) 
    - Discuss how anything in our local nextflow config will overwrite what is present in the `rnaseq/nextflow.config` file
    - Remind about order of priority for reading parameters 

**<INSERT SOME ACTIVITIES>**

------------------
## Troubleshooting

### **Nextflow version incompatible with nf-core/rnaseq**

The version of Nextflow currently installed on Nimbus BioImage is incompatible with the most recent stable release of the nf-core/rnaseq workflow. When running Nandan's command with cvmfs materials: 
```
nextflow run ../rnaseq/main.nf \
    --input samplesheet.csv \
    --outdir /home/ubuntu/nfcoreWorkshopTesting/exercise1/results \
    --max_memory '6.GB' --max_cpus 2 \
    --gtf $cvmfs_path/chr_18_startOfLine.gtf \
    --fasta $cvmfs_path/chr18.fa \
    --star_index $cvmfs_path/chr18_STAR_singularity_index \
    -profile singularity \
    -with-report execution_report_exercise1.html \
    -with-trace execution_trace_exercise1.txt \
    -with-timeline timeline_exercise1.html \
    -with-dag dag_exercise1.png 
```

Got the following error: 
```
Nextflow version 22.04.3 does not match workflow required version: >=22.10.1
```

Workaround was to update Nextflow with:
```
sudo nextflow self-update
```

Then rerun workflow script `exercise1_run.sh`, but got: 
```
exercise1_run.sh: line 3: /usr/local/bin/nextflow: Permission denied
```

Running the following worked:
```
sudo bash exercise1_run.sh
```

### **Graphviz installation**

Had to install graphviz for DAG:
```
sudo apt install graphviz
```

### **Generating STAR index**

STAR index for chr18 was created with outdated version of STAR. In this workshop we will be running the latest stable release of nf-core/rnaseq and will need to update the index. When running the pipeline with Nandan's materials, got the following:
```
EXITING because of FATAL ERROR: Genome version: 20201 is INCOMPATIBLE with running STAR version: 2.7.9a
SOLUTION: please re-generate genome from scratch with running version of STAR, or with version: 2.7.4a
```

Resolved. Prepared indexes on personal testing VM:

1. Downloaded mm10 chr18 fasta and mm10 gtf
```
wget https://ftp.ensembl.org/pub/release-109/fasta/mus_musculus/dna/Mus_musculus.GRCm39.dna.chromosome.18.fa.gz
wget https://ftp.ensembl.org/pub/release-109/gtf/mus_musculus/Mus_musculus.GRCm39.109.gtf.gz 
```

2. Unzip 
```
gunzip Mus_musculus.GRCm39.dna.chromosome.18.fa.gz 
gunzip Mus_musculus.GRCm39.109.gtf.gz 
```

3. Index with STAR 
```
singularity run -B /data docker://quay.io/biocontainers/star:2.7.9a--h9ee0642_0 \
    STAR \
    --runThreadN 10 \
    --runMode genomeGenerate \
    --genomeSAindexNbases 12 \
    --genomeDir /data/NFCORE_MATERIALS/mm10_reference \
    --genomeFastaFiles /data/NFCORE_MATERIALS/mm10_reference/mm10_chr18.fa \
    --sjdbGTFfile /data/NFCORE_MATERIALS/mm10_reference/Mus_musculus.GRCm39.109.gtf
```

4. Extract chr18 and header lines only from gtf:
```
awk '/^18/ || /^#!/'  Mus_musculus.GRCm39.109.gtf > mm10_chr18.gtf
```

5. Reran pipeline
```
materials=/home/ubuntu/nfcoreWorkshopTesting/materials/mm10_reference

nextflow run ../rnaseq/main.nf \
    --input samplesheet.csv \
    --outdir /home/ubuntu/nfcoreWorkshopTesting/exercise1/results \
    --max_memory '6.GB' --max_cpus 2 \
    --gtf $materials/mm10_chr18.gtf \
    --fasta $materials/mm10_chr18.fa \
    --star_index $materials/STAR \
    -profile singularity \
    -with-report execution_report_exercise1.html \
    -with-trace execution_trace_exercise1.txt \
    -with-timeline timeline_exercise1.html \
    -with-dag dag_exercise1.png 
```

```
[nf-core/rnaseq] Pipeline completed successfully-
Completed at: 14-Feb-2023 01:46:38
Duration    : 22m 23s
CPU hours   : 0.5
Succeeded   : 202
```

-------------------
## Links/resources 

* [CVMFS testing repository](https://github.com/Sydney-Informatics-Hub/cvmfsTesting)
* [STAR user guide](https://physiology.med.cornell.edu/faculty/skrabanek/lab/angsd/lecture_notes/STARmanual.pdf)
