# Exercise 2: Customise run via parameter schema

## Objectives 
* Write a parameter file
  - Explore the dnocs to find params that are relvant to your analysis 
* Understand YAML file format
* Rerun workflow with -params-file
* Understand the use of -params-file for reproducibility and transparency 

---------------------
## Testing reflection
* FYI we did not run default command, cannot do so on instances. Both standard cli run exercises with flags can be merged into one  
* FYI good opportunity to reiterate importance of reproducibility in context of collaboration, sharing, publications
* FYI focus in this exercise is on customising processes, not their execution 
* TODO request Seqera cover configuration heirarchy in session 1 to be reiterated here 
* TODO add email for summary email at end of the run 
  - This does not work, see email_on_complete_error which captures the errors from nextflow.log when attempting to use --email parameter. This was with the latest nextflow and nf-core rnaseq versions
* TODO 
* DONE 
* Q 

---------------------
## Content draft UPDATED 9/4/23 - replace non-functional 3' clip param with others that work 

Nextflow params variables can be saved in to a JSON or YAML file called nf-params.json and used by Nextflow with the `-params-file` flag. This makes it easier to reuse these in the future.

The command takes one argument - either the name of an nf-core pipeline which will be pulled automatically, or the path to a directory containing a Nextflow pipeline (can be any pipeline, doesn't have to be nf-core).

### Check the multiqc summary from the Exercise 1 run:
- Install 'Live Server' VS Code plugin by Ritwick Dey
- Navigate to the directory `Exercise_1/multiqc/star_salmon` on your VS Code explorer pane (left panel), right click `multiqc_report.html` then select `open with Live Server` - the html report will open in your browser window (may take a few seconds) 
- Explore the output
- Note the `WARNING: Fail strand check` seciton - all 6 of our samples have failed the strand check! We will adjust this parameter in the next run 
- Activity: can you find the parameter we need to add to correct this?
- Answer: https://nf-co.re/rnaseq/3.10.1/parameters#salmon_quant_libtype and see https://salmon.readthedocs.io/en/latest/salmon.html#what-s-this-libtype to find what string to supply ('A' for auto-detect) 

### Check out the work dir and understand the `resume` feature
- Lessons from https://www.nextflow.io/blog/2019/demystifying-nextflow-resume.html and https://training.nextflow.io/basic_training/cache_and_resume/#work-directory
- Show that the nimbus config has cache=lenient

### Create parameter file
We will create a YAML format file with our inputs. This file can then be saved with the workflow, shared with collaborators etc. A useful reference for the analysis as well as making it easier to rerun/reproduce - we don't have to keep typing flags and parameters onto the command line making it hard to read and unwieldy  

We would like to make some changes to the previous run: 
 - [Correct for our strand failure](https://nf-co.re/rnaseq/3.10.1/parameters#salmon_quant_libtype) 
 - [Save the trimmed reads to the output directory](https://nf-co.re/rnaseq/3.10.1/parameters#save_trimmed)
 - Save the reads that fail to map to the mouse transcriptome. Can you find a parameter that will allow you to do this? 
 - Answer: There is no specific parameter described in the rnaseq docs to do this, BUT a new feature from v 3.10 enables you to add any STAR or salmon flag! See https://nf-co.re/rnaseq/3.10.1/parameters#extra_salmon_quant_args 
 
### How can we know what tool flags are applied by default? 
- One method is to find the process script within the rnaseq workflow and examine the syntax
- Another method (useful if you have already run the workflow with default setting) is to utilise the `nextflow log` tool

Run `nextflow log` with no arguments - observe that it shows the execution timestamp, run duration, run name, run completion status, revision ID, session ID, run command. All recent runs will be listed, with the most recent last (ie closest to your returned command prompt):
```
2023-04-09 14:31:35     43.9s           distracted_montalcini   OK      f421ddc35d      35b6178b-9cb2-4d05-977d-63073800a5c8    nextflow run ../rnaseq/main.nf --max_memory 6.GB --max_cpus 2 -profile singularity -resume -params-file params.yaml 
```

Note that you need to run this command within your main working directory - otherwise you will receive an error `It looks like no pipeline was executed in this folder (or execution history is empty)`

You can query a nextflow run with the run name, and this will list out all relevant work directories. Recall that the actual tool commands issued by the nexflow processes are all recorded in hidden script files called `.command.sh` within the execution process directory. 

One way of observing the actual run commands issued by the workflow is to view these comamnd scripts. But how to find them?!

As an activity, we will use the nextflow `log` command with our Exercise 1 run name to find the script files for the tool `salmon`. 

Run the following:

```
run_name=<ENTER_YOUR_RUN_NAME>
tool=salmon
nextflow log ${run_name} | while read line; do      cmd=$(ls ${line}/.command.sh 2>/dev/null);     if grep -q $tool $cmd;     then          echo $cmd;     fi; done 
```

View these command files eg with `cat` - can you see the full `salmon quant` command? (There are multiple salmon steps in the workflow, inlcuding index and an R script. We are looking for `salmon quant` which performs the read quantification) 

Output eg:
```
salmon quant \ 
    --geneMap mm10_chr18.gtf \ 
    --threads 2 \ 
    --libType=SF \ 
    -t genome.transcripts.fa \ 
    -a SRR3473985.Aligned.toTranscriptome.out.bam \ 
     \ 
    -o SRR3473985 
```

We now know the default `salmon quant` command used by the rnaseq workflow. And we know that we can add any salmon flags that are not covered by nf-core rnaseq by using the nf-core parameter `extra_salmon_quant_args`. 

To find out all the salmon quant flags, we can of course look at the salmon documentation online. Another option is to use the view the command line help. We can do this via the salmon tool in our singularity cachedir:

```
singularity run ~/singularity/depot.galaxyproject.org-singularity-salmon-1.9.0--h7e5ed60_1.img 
salmon --help
salmon --help-alignment
```

Scroll through the parameters, there are many available depending on your research project. Today we are looking to capture the unmapped read IDs, and this is the last salmon flag in the list: `--writeUnmappedNames`. 

Open a file `params.yaml` and add the following:

```
input: "<path>/materials/samplesheet.csv" 
outdir: "Exercise2"
gtf: "<path>/materials/mm10_reference/mm10_chr18.gtf"
fasta: "<path>/materials/mm10_reference/mm10_chr18.fa"
star_index: "<path>/materials/mm10_reference/STAR" 
save_trimmed  : true
salmon_quant_libtype : A
extra_salmon_quant_args : '--writeUnmappedNames'
```
Any of the [workflow parameters](https://nf-co.re/rnaseq/3.10.1/parameters) can be added to the parameters file in this way. 

Reminder: nextflow can use cached output! If we apply the `-resume` flag to the run, nextflow will only compute what has not been changed. We should expect the initial fastqc and STAR alignments to be restored from cache and the salmon steps to be recomputed. 

Once your params file has been saved, run the following, observing how the command is now shorter thanks to offloading some parameters to the params file. Note the use of a single `-` for 'resume' and 'params-file' as these are nextflow options not nf-core parmeters, which would use two dashes.
```
nextflow run ../rnaseq/main.nf \
  --max_memory 6.GB \
  --max_cpus 2 \
  -profile singularity \
  -resume \
  -params-file params.yaml                                            
```

Check the output:
  - Observe that there are now trimmed fastq files within the ./Exercise2/trimgalore directory that were not included in ./Exercise1/trimglaore, because we added the 'save_trimmed' parameter. 
  - Observe that there are now unmapped files in the ./Exercise2/star_salmon directory - HAVE MESSAGED CHRIS ON SLACK ABOUT THIS, I CANNOT FIND THE UNMAPPED OUTPUTS!
  
Once again, issue the nextflow log command - observe that you now have info reported for 2 runs (or more, if you had repeat commands due to errors etc) 
```
nextflow log
```

Use the most recent run name to query the nextflow work directories, and `cat ` one of the salmon command files to observe that our 2 extra salmon flags have been added:

```
run_name=<ENTER_YOUR_RUN_NAME>
tool=salmon
nextflow log ${run_name} | while read line; do      cmd=$(ls ${line}/.command.sh 2>/dev/null);     if grep -q $tool $cmd;     then          echo $cmd;     fi; done 
```

Example output: 
```
salmon quant \
    --geneMap mm10_chr18.gtf \
    --threads 2 \
    --libType=A \
    -t genome.transcripts.fa \
    -a SRR3473984.Aligned.toTranscriptome.out.bam \
    --writeUnmappedNames \
    -o SRR3473984
```

Now open the mutliqc report and observe that the fail strandedness check is no longer present: 
- On the left VS code explorer pane, navigate to folder `./Exercise2/multiqc/star_salmon` and right click multiqc_report.html, select (at the very top ) `Open with Live Server`

---------------------
## Troubleshooting

Run time was 1min4s, and all 200 tasks pulled from cache. This is not expected - STAR should have been re-run on the updated fastq input. The trimgalore outdir DOES contain the trimmed reads as requested. Why was the mapping pulled from cache when the input reads have changed?

```
ubuntu@small-testing:~/nfcoreWorkshopTesting$ zcat exercise2/results/trimgalore/SRR3473984_trimmed.fq.gz > trimmed
ubuntu@small-testing:~/nfcoreWorkshopTesting$ zcat materials/fastqs/SRR3473984_selected.fastq.gz > notTrimmed
ubuntu@small-testing:~/nfcoreWorkshopTesting$ sdiff -s trimmed notTrimmed | wc -l
68758
```


---------------------
## Links/resources 

* [Nextflow configuration docs](https://www.nextflow.io/docs/latest/config.html?highlight=params#configuration)
* [Nextflow tips: params file](https://www.nextflow.io/blog/2020/cli-docs-release.html)
* [YAML tutorial](https://www.cloudbees.com/blog/yaml-tutorial-everything-you-need-get-started)
