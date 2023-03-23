# nfcoreWorkshopTesting

Testing and materials development for AusBioCommons customising nf-core workshop 2023. 

* [Set up environment](#set-up-environment)
    + [Download rnaseq pipeline](#download-rnaseq-pipeline)
    + [Download materials](#download-materials)
* [Materials testing](#materials-testing)
* [Issues and troubleshooting](#issues-and-troubleshooting)

Materials developed by: 
* Cali Willet
* Chris Hakkaart
* Georgie Samaha 

----------------------------

## How this repo works 

This repository contains draft materials and details regarding their development for the Australian BioCommons customising nf-core workshop. Materials have been organised into separate directories for parts 1 and 2: 
* Create a subdirectory for each exercise or session to be run in each part
* See [Materials testing](#materials-testing) for instructions on what information to provide for each session/exercise
* Include any useful files and scripts for each exercise/session in the relevant subdirectory
* See [Download materials](#download-materials) for how to get workshop data 
* See [Issues and troubleshooting](#issues-and-troubleshooting) for nf-core GitHub issue template 

## Set up Nimbus environment 

Instances set up using Nimbus bioimage which has Singularity, Nextflow, Docker already installed. 

> **_NOTE:_**  See `NIMBUSSETUP.md` for instructions on setting up your test instance. 

### **Download rnaseq pipeline**

Download and install nf-core/rnaseq code with: 

```bash
git clone https://github.com/nf-core/rnaseq.git
```

### **Download materials**

Using the same dataset as rnaseq workshop, but needed to recreate STAR indexes for most recent stable release of nf-core/rnaseq workflow and opting to rename files and directory for sake of clarity. Will send updated materials to Alex to be uploaded to cvmfs once finalised. For now materials are on Cloudstor (and subject to change) and contain:

* `mm10_reference/mm10_chr18.fa` 
* `mm10_reference/mm10_chr18.gtf`
* `mm10_reference/STAR/` (star index files)
* `samplesheet.csv` (check path to fqs: `/home/ubuntu/nfcoreWorkshopTesting/materials/`)
* `fastqs/`
* `README.md`

Download subset/test data from Cloudstor
```bash
wget -O nfcore_materials.tar.gz https://cloudstor.aarnet.edu.au/plus/s/gIBdDhKEwfq2j58/download
```
Unpack the files: 
```bash
tar -zxvf nfcore_materials.tar.gz
```

## Materials testing  

Notes for each exercise draft have been included in the Part1 and Part2 directories. In each exercise/part README.md, include: 

* Objectives 
* Testing reflection
* Content draft 
* Troubleshooting
* Links/resources

A template README.md can be found at: nfcoreWorkshopTesting/exerciseREADME.md

## Issues and troubleshooting 

Any issues with nf-core/rnaseq workflow needs to be submitted as an issue on their GitHub page and shared with Chris Hakkaart. A template for submitting bugs is [here](https://github.com/Sydney-Informatics-Hub/nfcoreWorkshopTesting/blob/main/issueTemplate.md). Please link to all issues below: 

* Email flag broken: https://github.com/nf-core/rnaseq/issues/949 
* TrimGalore parameter warning: https://github.com/nf-core/rnaseq/issues/961  