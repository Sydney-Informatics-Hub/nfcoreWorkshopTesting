# Exercise 6: Using 'launch' 

## Objectives 
- Interact with the [nf-core launch](https://nf-co.re/launch) tool to adjust parameters
- Discuss the 3 options to using launch and when each would be applicable
- Hands-on practice of using the launch tool
  -  Taking a manual copy of the json file to use ('No internet connection' method, applicable to HPC)
  -  Using `nf-core launch --id <ID>` ('With internet connection' method, applicable to VM)
- Keeping the created json file with your analysis for reproducibility
    
---------------------
## Testing reflection
- Directions: https://nf-co.re/launch 
- Need to fist install the tools:

```
sudo pip install zipp  
sudo pip install nf-core 
```

---------------------
## Content draft 


---------------------
## Troubleshooting

- Is there a way to add ext.args params using launch?
- Unable to run, received error `bash: /usr/local/bin/nextflow: Permission denied` despite having execution permissions and `alias nextflow='sudo /usr/local/bin/nextflow'`. Resolved with: 
```
sudo chmod ugo+rwx /usr/local/bin/nextflow
```
- Command used:
```
nf-core launch --id 1679440128_d4f0ce483d94 --params-out nf-parms-toolsLaunch.json
```

#### Georgie's test:
1. Go to https://nf-co.re/launch 
2. Under 'Launch pipeline from the web': 
    * Select a pipeline: rnaseq
    * Pipeline release: 3.10.1 
    * **LAUNCH**
3. Now configure workflow parameters for a pipeline run
    * Collect your launchID: `1679534632_6c2cc0dcdfd0`
    * -name: `Ex6_test`
    * -profile: none?
    * -work-dir: `./work`
    * -resume: `false`
    * filled in input, gtf, fasta, STAR index options 
4. Back to the command line, run:
```
nf-core launch --id 1679534632_6c2cc0dcdfd0
```

```

```

---------------------
## Links/resources 
