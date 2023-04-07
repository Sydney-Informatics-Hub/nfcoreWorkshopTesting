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
