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
- Unable to run, received error `bash: /usr/local/bin/nextflow: Permission denied` despite having execution permissions and `alias nextflow='sudo /usr/local/bin/nextflow'`
- Command used:
```
nf-core launch --id 1679440128_d4f0ce483d94 --params-out nf-parms-toolsLaunch.json
```
---------------------
## Links/resources 
