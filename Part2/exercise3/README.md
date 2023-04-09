# Exercise 3: Customise execution with an institutional config

## Objectives 

---------------------
## Testing reflection

---------------------
## Content draft - USED GS TEST NIMBUS CONFIG

- Copied the pawsey_nimbus.config to working directory
- Ran this:

```
nextflow run ../rnaseq/main.nf \
	-c pawsey_nimbus.config \
	-profile singularity,c2r8 \
	-params-file params.yaml \
	-resume \
	--outdir Exercise3
```

Worked great, everything pulled from cache as expected, finished in a few seconds. 

Most of the time for this exercise will be describing the configuration profile, where its hosted, whats in it, etc

Perhaps adding info on using multiple configs with `includeConfig` eg

```
# Within one config, call on another with this line:
includeConfig ‘nextflow_custom.config’
```

---------------------
## Content draft 

https://nf-co.re/rnaseq/parameters#config_profile_url

### Reminder: order of priority for parameters!

The main configuration file for this workflow is at rnaseq/nextflow.config. We can create our own custom config and make chages locally. Parameters within the `rnaseq/nextflow.config` file that are also within our local config will be overridden by the local config. 

### When/how to use a custom config
Having a local config is great for parameters like CPU and memory, where the appropriate settings are dictated by your local environment. They are also a great place to add default settings for things such as the run reports.  

### Customise our config

Open local file `nextflow.config` and add the following lines: 

```
params.max_cpus = 2
params.max_memory = '6 GB'
params.outdir = "Results"
```

### Run 
Once you have saved your `nextflow.config` file, run the following - much neater having our parameters within a params file and configuration settings within a config file! 

```
nextflow run ../rnaseq/main.nf \
	-profile singularity \
	-params-file params.yaml \
	-resume
```

Quick quiz:
- Where do you think your output will be?
  - It's not in Results!
  - The default `outdir` value specified in our custom config has been overridden by the value specified in our params.yaml. Remember the order of priority!
- If we wanted to run this analysis writing outputs to a directory named `Exercise3`, how would we do that without changing the configuration or paramaters file?
	- Insert cool drop down 'Answer' box
```
nextflow run ../rnaseq/main.nf \
	-profile singularity \
	-params-file exercise2_params.yaml \
	-resume \
	--outdir Exercise3
```


### Defining parameters for specific processes

Our training VMs are restricted in size, but typically you would be conducting bioinformatics analyses on platforms with more resources. You may want to set the compute resources that each process can use individualy. In this way you can make better use of the infrastructure.

For example, if you have set `max_cpus` to the maximum number of CPUs on your machine, and each process attempts to use all of those CPUs whether they can benefit from them or not, you are preventing multiple processes from running concurrently (if this is possible for the workflow).

You can also create a 'dev' or 'testing' mode, where you use small resources to test on the login node of your platform before launching the full workflow with larger resources.

Below is a simple example for a cluster running PBS Pro. To run using the `cluster` configuration, add `-profile cluster` to your run command, or to run in testing/development mode, add `-profile dev`. 

In the below example, the `FASTQC` process is executed on the resources assigned to the parent nextflow job, while the processes `INDEX` and `MAP` are submitted to the PBS Pro job scheduler with different memory, CPU and walltime limits as specified under the `withName` definition for each process.  

```
profiles {                                                                                                                                                               
        cluster {           
                process {
                        // Default job submission optioms: 
                        executor = 'pbspro'
                        clusterOptions = '-q normal -P er01 -lstorage=scratch/er01'

                        withName: FASTQC {
                                cpus = 1
                                time = 10.m
                                memory = 2.GB 
                                // This will cause this process to run on the resources of the parent PBS job rather than be forked to a new PBS job:
                                executor = 'local'
                        }

                        withName: INDEX {
                                cpus = 6
                                time = 15.m
                                memory = 24.GB
                        }

                        withName: MAP {
                                cpus = 12
                                time = 30.m
                                memory = 48.GB
                        }
                }
        }
        dev {
                process {
                        cpus = 2
                        mem = 8.GB
                }
        }
}
```



---------------------
## Troubleshooting

---------------------
## Links/resources 
