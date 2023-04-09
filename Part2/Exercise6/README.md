# Exercise 6: Using 'launch' 

## Objectives 
- Interact with the [nf-core launch](https://nf-co.re/launch) web GUI to adjust parameters
- Discuss the 3 options to using launch and when each would be applicable
- Hands-on practice of using the launch tool
  -  Taking a manual copy of the json file to use ('No internet connection' method, applicable to HPC)
  -  Using `nf-core launch --id <ID>` ('With internet connection' method, applicable to VM)
- Keeping the created json file with your analysis for reproducibility
    
---------------------
## Testing reflection

- Need to fist install the tools:

```
sudo pip install zipp  
sudo pip install nf-core 
```

- Add this to bashrc then reload session/run source
```
alias nextflow='sudo /usr/local/bin/nextflow'
```

- Adjust permissions on /usr/local/bin/nextflow:
```
sudo chmod ugo+rwx /usr/local/bin/nextflow 
```
---------------------
## Content draft 

### Fill in the launch GUI

- Open the [nf-core launch](https://nf-co.re/launch) web GUI - Select the rnaseq pipeline from the `select a pipeline` menu and select pipeline release `3.10.1` (or whatever is the latest version we have tested on)
- Select `launch` (insert an image of the blue rocket icon) 
- VIP: On the far right, unhide `Show hidden params`
- Leave `name` blank (default will be used, like it has been for our CLI runs so far) 
- For `profile`, enter `singularity` (also the pawsey nimbus config when ready) 
- Leave `work-dir` as default, toggle `resume` to true 
- Under `input`, copy paste the path to the `samplesheet.csv` 
- For `outdir`, specify `Exercise6`
- Enter your email address
- Now scroll through the remaining options, and fill in all of the parameters we have applied via our params file (including the multiqc yaml file!) 
- CPUs and MEM - this may or may not need to be done manually, if the pawsey nimbus config is available at nf-core then it wont be but if it is not, then need to add these (they are 'hidden parmas') 
- Once you are satisified that the params are all accounted for (remember that the extra_args.config should have the quality decreased from 40 to 30), click `Launch workflow`
- Note that you can select `Return to editor` to make changes to your config as required. Also note that if you do this, your `resume` will toggle back to `false`, your `profile` will have cleared itself, and the hidden parameters will have re-hidden themselves!

### Launch the workflow with the 'Launching with no internet and without nf-core/tools' option 

- Because we are on a VM, which has internet connection, we could use the first option 'If your system has an internet connection'
- However, as this was covered in Part 1, we will practice with the 'no internet connection' option, which is usually the case if you are running on a HPC. 
- Follow the instructions for this method:
  - Copy the JSON params to a file in your working directory and save it as `nf-params.json`
  - Copy the nextflow run command and execute it in your VM
  
```
{
    "input": "\/home\/ubuntu\/materials\/samplesheet.csv",
    "outdir": "Exercise6",
    "email": "cali.willet@sydney.edu.au",
    "fasta": "\/home\/ubuntu\/materials\/mm10_reference\/mm10_chr18.fa",
    "gtf": "\/home\/ubuntu\/materials\/mm10_reference\/mm10_chr18.gtf",
    "star_index": "\/home\/ubuntu\/materials\/mm10_reference\/STAR",
    "save_trimmed": true,
    "salmon_quant_libtype": "A",
    "extra_salmon_quant_args": "'--numBootstraps 10'",
    "save_unaligned": true,
    "config_profile_name": "pawsey_nimbus.config,extra_args.config",
    "max_cpus": 2,
    "max_memory": "6.GB",
    "multiqc_config": "\/home\/ubuntu\/run_then_launch\/multiqc_config.yaml",
    "show_hidden_params": true
}
```

This did not work, despite that it worked in Exercise 5:

```
nextflow run nf-core/rnaseq -r 3.10.1 -profile singularity,c2r8 -resume -params-file nf-params.json
```

The error:

```
ubuntu@calidev:~/run_then_launch$ nextflow run nf-core/rnaseq -r 3.10.1 -profile singularity,c2r8 -resume -params-file nf-params.json
N E X T F L O W  ~  version 22.10.7
Unknown configuration profile: 'c2r8'

Did you mean one of these?
    crg
```


Had to go back and add the CPU and MEM to the JSON, then run without 'c2r8':

```
nextflow run nf-core/rnaseq -r 3.10.1 -profile singularity -resume -params-file nf-params.json
```

Another issue: when using `--numBootstraps 10` for salmon quant, received an error `PLEASE UPGRADE SALMON`
- image is version 1.9, latest is 1.10.1
- When I replaced the extra salmon arg from botstrap to `--gcbias`, the run completed successfully. 
- I dont want to use GC bias as the example, because the mqc report shows no bias! 
- Need to fix the version or find a different example flag 








---------------------
## Troubleshooting

- Is there a way to add ext.args params using launch?
- First run resulted in `Downloading dependencies` - why are there dependencies needing to be downloaded, when the pipeline has been run on the same system multiple times? Do I need to set a cachedir? There was no field for this in the 'launch' GUI 

- Command used:
```
nf-core launch --id 1679440128_d4f0ce483d94 --params-out nf-parms-toolsLaunch.json
```

- Error:
```
                                                                                                      
Do you want to run this command now?  [y/n] (y): y
INFO     Launching workflow! 🚀                                                                       
CAPSULE: Downloading dependency org.slf4j:jul-to-slf4j:jar:1.7.36
CAPSULE: Downloading dependency io.nextflow:nextflow:jar:22.10.6
CAPSULE: Downloading dependency org.slf4j:jcl-over-slf4j:jar:1.7.36
CAPSULE: Downloading dependency commons-io:commons-io:jar:2.11.0
CAPSULE: Downloading dependency commons-codec:commons-codec:jar:1.15
CAPSULE: Downloading dependency com.google.guava:listenablefuture:jar:9999.0-empty-to-avoid-conflict-with-guava
CAPSULE: Downloading dependency org.codehaus.groovy:groovy:jar:3.0.13
CAPSULE: Downloading dependency ch.qos.logback:logback-core:jar:1.2.11
CAPSULE: Downloading dependency org.yaml:snakeyaml:jar:1.30
CAPSULE: Downloading dependency org.codehaus.groovy:groovy-nio:jar:3.0.13
CAPSULE: Downloading dependency org.eclipse.jgit:org.eclipse.jgit:jar:6.2.0.202206071550-r
CAPSULE: Downloading dependency com.google.code.findbugs:jsr305:jar:3.0.2
CAPSULE: Downloading dependency org.codehaus.groovy:groovy-templates:jar:3.0.13
CAPSULE: Downloading dependency ch.qos.logback:logback-classic:jar:1.2.11
CAPSULE: Downloading dependency org.slf4j:log4j-over-slf4j:jar:1.7.36
CAPSULE: Downloading dependency com.google.guava:guava:jar:31.1-jre
CAPSULE: Downloading dependency org.slf4j:slf4j-api:jar:1.7.36
CAPSULE: Downloading dependency org.codehaus.groovy:groovy-xml:jar:3.0.13
CAPSULE: Downloading dependency com.google.errorprone:error_prone_annotations:jar:2.11.0
CAPSULE: Downloading dependency com.google.j2objc:j2objc-annotations:jar:1.3
CAPSULE: Downloading dependency io.nextflow:nf-commons:jar:22.10.6
CAPSULE: Downloading dependency org.codehaus.groovy:groovy-json:jar:3.0.13
CAPSULE: Downloading dependency com.google.guava:failureaccess:jar:1.0.1
CAPSULE: Downloading dependency com.googlecode.javaewah:JavaEWAH:jar:1.1.13
CAPSULE: Downloading dependency io.nextflow:nf-httpfs:jar:22.10.6
CAPSULE: Downloading dependency org.checkerframework:checker-qual:jar:3.12.0
.nextflow/history.lock (Permission denied)                          
```
---------------------
## Links/resources 
