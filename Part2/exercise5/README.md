# Exercise 5: Specifying external arguments to a process

## Objectives 
- Discuss the importance of reading the tool documentation for key tools in the workflow to decide on customised parameters
- Identify which tool parameters are included by nf-core, either hard-coded or customisable as a workflow argument either by command line or parameters file
- Learn how to specify additional external arguments to a process ie those that are available to the tool but not covered by nf-core arguments
- Perform a little troubleshooting by exploring QC, workflow code and command file

---------------------
## Testing reflection

---------------------
## Content draft 

nf-core provides a number of flexible parameters for each process in the workflow. These are specified with the double dash syntax on the command line, or saved in a parameters file as we have covered in the previous exercises.

However, nf-core has not included every possible parameter that is available to every tool within the workflow; they have included only those most commonly changed from the default as chosen by the software developers. 

For example, look at the parameter options for the tool Trim Galore, under [Read trimming options](https://nf-co.re/rnaseq/3.10.1/parameters)

You will see 8 parameters, 6 of which are Trim Galore-specific parameters and the last 2 which are nf-core parameters. 

Now look at the full list of trim_galore parameters:
Option 1) View the [Trim Galore parameters online](https://github.com/FelixKrueger/TrimGalore/blob/master/Docs/Trim_Galore_User_Guide.md#general-options)
Option 2) Use the command line help:
```
singularity run ~/singularity/depot.galaxyproject.org-singularity-trim-galore-0.6.7--hdfd78af_0.img
trim_galore --help | more
exit
```
It would be very unwieldy for nf-core to cover all of the available options for this tool and every other in the workflow on their main workflow page!

You have the flexibility to use ANY tool parameter, whether a parameter is specified by the nf-core workflow parameters page or not, by providing these as **extra arguments**.
 
View the Trim Galore process `main.nf`: 
```bash
cat -n ../rnaseq/modules/nf-core/trimgalore/main.nf
```

Look at the top of the `script:` block (lines 24-25):
```bash
script:
def args = task.ext.args ?: ''
```

We can use the `ext.args` variable to parse any tool parameters to the tool command run by the process. Neat huh! 

Now look at the actual `trim_galore` run command (starting at line 42 or 58). 

We can see that nf-core has hardcoded the parameters `cores` and `gzip`, and also fed in the `paired` argument if the input is detected as paired-end reads. 

As a researcher, its up to you to decipher what customisations are required to make your analysis suitable for your data and research questions. 

In order to understand what parameters you may want to specify as extra arguments, its first important to: 
a) Read the tool documentation to understand all the available parameters for that tool
	- Tool doc
b) View the list of nf-core parameters for that tool
	- nf-core workflow params page
c) View which paramteres are hard-coded within the nextflow process 
	- process main.nf
	
Then, if parameters you want to use are not covered by the nf-core parameters or hard-coded in the run command, you can add them using the `ext.args` variable. 

For the sake of the exercise, let's assume we want to increase the minimum quality Phred score from the default of 20 to a much more stringent value of 40 (note for RNAseq researchers, this of course is way too high!). 

Following the Trim Galore user guide, we would do this with the syntax `--quality 40`. 

To parse that to the Trim Galore process, we need to specify this in a custom config, just like we have done for eg `max_cpus`, `max_memory` and `multiqc` in the previous exercises. 

The difference this time is that we want to restrict the usage of the parameter to the only place it makes sense: to the `TRIMGALORE` process. We can do this by using the `withName: <process>` syntax.  

Open a new file `extra_args.config` for editing, then add the following content:

```bash
#!/usr/bin/env nextflow

nextflow.enable.dsl=2

process {
    withName: ".*:TRIMGALORE"  {    
        ext.args    =   '--quality 40'
    }
}
``` 
NOTE TO US - we need to add a part in here about how to find the correct name for a process to specify in the above - my first attempt using the full process name (as per doc) DID NOT WORK. See Troubleshooting section below. Clarify with Chris H then add in ... 

We need to instruct nextflow to use the custom config, and we do this with the `-c` flag. 

```
nextflow run \
    ../rnaseq/main.nf \
    -profile singularity \
    -params-file params.yaml \
    -c extra_args.config \
    --outdir Exercise5 \
    -resume
```

Given we have applied the `-resume` flag, what tasks do you expect to be re-run, and what outputs do you expect to be taken from cache?
- Cached output for initial fastqc
- Rerun trim and all downstream

How long did your run take? Was the run time what you expected? Do you notice anything different on the STDOUT compared to the previous runs? 

<insert STDOUT screenshots?> 

Open the file multiqc_report.html (use Liver Server in VS Code or scp to take a local copy). 

On the navigation headings on the left, click on `Fail trimmed samples`. Observe that all 6 samples have < 5,000 reads remaining after trimming. 

We can also see this information in a plain text output file: 
```
cat ./exercise5/multiqc/star_salmon/multiqc_datamultiqc_fail_trimmed_samples-plot.txt 
```
What has caused these samples to fail? 

View the [rnaseq workflow code](https://github.com/nf-core/rnaseq/blob/master/workflows/rnaseq.nf)

Observe line 267 - "Filter channels to get samples that passed minimum trimmed read count"

and 

line 295 - `if (num_reads <= params.min_trimmed_reads)`

Our samples have less reads after trimming than required by the parameter [`min_trimmed_reads`](https://nf-co.re/rnaseq/3.10.1/parameters#min_trimmed_reads) which has a default value of 10,000. 

This highlights the need to thoroughly check your output to ensure that the intended anlysis has been run and the results are what you require. The message `Pipeline completed successfully` (and also exit status of zero for invidual tasks or cluster jobs if you are running on a cluster) indicates that there were no errors running the pipeline, not that your samples have produced the desired output. 

Also note that nf-core has not reported the custom extra argument provided to trimGalore. On your multiqc_report.html file, view the section 'nf-core/rnaseq Workflow Summary'. The quality parameter is not described. 

We can view the full parameters that were supplied to trimgalore in two ways:

Option 1) Trim galore log
- View one of the log files for a sample. Note that all supplied parameters, including our custom quality threshold of 40, are listed under 'SUMMARISING RUN PARAMETERS'
```
more ./exercise5/trimgalore/SRR3473988.fastq.gz_trimming_report.txt
```

Option 2) Process execution script
- Recall that every task that is run is recorded in a file `.command.sh` within the task execution directory. 
- Use the nextflow log tool to find the name of this run (unless you have remembered it!) 
- Then issue the commands as before to find the `trim_galore` process execution scripts:

```
run_name=<ENTER_YOUR_RUN_NAME>
tool=trim_galore
nextflow log ${run_name} | while read line; do      cmd=$(ls ${line}/.command.sh 2>/dev/null);     if grep -q $tool $cmd;     then          echo $cmd;     fi; done 
```

Open the file or use `cat` or a text editor to view it. 

< insert screenshot instead of code>

```
#!/bin/bash -ue
[ ! -f  SRR3473988.fastq.gz ] && ln -s SRR3473988_selected.fastq.gz SRR3473988.fastq.gz
trim_galore \
    --quality 40 \
    --cores 1 \
    --gzip \
    SRR3473988.fastq.gz

cat <<-END_VERSIONS > versions.yml
"NFCORE_RNASEQ:RNASEQ:FASTQ_FASTQC_UMITOOLS_TRIMGALORE:TRIMGALORE":
    trimgalore: $(echo $(trim_galore --version 2>&1) | sed 's/^.*version //; s/Last.*$//')
    cutadapt: $(cutadapt --version)
END_VERSIONS
```

If this were a real analysis, we would re-run with a reduced quality threshold of 30, which is higher than the original default of 20 thus ggiving us that additional stringency we were after while hopefully not being too high to fail all of the samples! 


---------------------
## Troubleshooting

1) How to know what exact label to use for processes with ext.args?
2) Why is the param supplied via ext.args not being reported as parameter that differs from defaults (in log, mqc report or stdout)?

I initially tried this in my local `nextflow.config`, following [documentation](https://nf-co.re/rnaseq/3.10.1/usage#advanced-option-on-process-level):

```
process {
    withName: 'FASTQ_FASTQC_UMITOOLS_TRIMGALORE:TRIMGALORE' {    
        ext.args    =   '--quality 40'
    }
}
```

This did not work, gave error 'There's no process matching config selector'

Process label from Chris:

```
process {
    withName : ".*:TRIMGALORE" {
        ext.args   = { "WRITE STUFF HERE " }
    }

```

Documentation does not describe the wildcard syntax which definitely seems a bit safer and easier to apply... 

---------------------
## Links/resources 








