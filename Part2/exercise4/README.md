# Exercise 4: Customise execution - further practice

## Objectives 
- Create a custom config for multiqc
- Add the custom config details to the parameters YAML file 
- Run with the -resume flag
- This exercise provides practice in:
  - Reading the documentation and selecting parameters that are suited to the specific data analysis at hand
  - Creating another configuration file in YAML format
  - Editing a parameters YAML file
  - Observing the order of priorities for configuration - like nextflow, the multiqc parameters are read from the main multiqc tool, with our locally psecified parameters overriding these
  - Observing the behaviour of nextflow cache and the '-resume' flag; only the multiqc module was changed and so only this part was re-run, while all other tasks were recovered from cache
---------------------
## Testing reflection

---------------------
## Content draft 

Create a file in working directory called `multiqc_config.yaml`.  

Add the following to this file:  

```
fastqc_config:  

  fastqc_theoretical_gc: "mm10_txome" 
```

To the `params.yaml` file, add:  

```
multiqc_config : multiqc_config.yaml 
```

Make sure both YAML files are saved, then run as usual, ensuring the `resume` flag is included: 

```
nextflow run ../rnaseq/main.nf \
  -profile singularity \
  -params-file params.yaml \
  -resume \
  --outdir Exercise4
```

UPDATE 9/4/23: 
- Used GS nimbus test config, ran this:

```
nextflow run ../rnaseq/main.nf \
	-c pawsey_nimbus.config \
	-profile singularity,c2r8 \
	-params-file params.yaml \
	-resume \
	--outdir Exercise4
```
- Worked great, everything pulled from cache as expected, finished in a few seconds.

While it is running, observe that all of the other anaylsis steps are restored from cache except for multiqc. 

- The changes we made above added the mouse transcriptome GC profile as a track to the fastQC per-sequence GC content plot.
- To view the changes, open the Exercise4 `multiqc_report.html` file with Live Server as per previosuly (or use scp to take a copy to your local computer if you are not on VS Code)
- Compare this report with the multiqc report from Exercise 1  by looking at the section `FastQC --> Per Sequence GC Content`. Compare the two plots to observe the custom track has been successfully added.  
- Can you detect any GC bias? 
  - For this dataset there is none. 
  - Note that if we did detect GC bias, we could go back and correct for this by adding the custom salmon flag `--gcBias` to the nf-core parameter `--extra_salmon_quant_args`. If we were to do this, our params.yaml file would have the following line:

```
extra_salmon_quant_args : '--writeUnmappedNames --gcBias'
```

*** add this as hints or extra challenge format - 
- location of the multiqc_report.html file is /home/ubuntu/nfcoreWorkshopTesting/exercise2/multiqc/star_salmon - can make an extra challenge to see if user can work out where to look for it given the stdout from the run (ie work/5d/e293 blah blah) 

---------------------
## Troubleshooting

---------------------
## Links/resources 
