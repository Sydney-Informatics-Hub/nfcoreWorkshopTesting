# Nimbus VM set up instructions 

The workshop will be developed for and delivered on Pawsey's Nimbus cloud infrastructure. VMs used to develop materials and for participants will be built on the ['Pawsey Bio - Ubuntu 22.04 - 2023-03'](https://support.pawsey.org.au/documentation/display/US/Nimbus+for+Bioinformatics) image. 

Pawsey have provided us with 6 testing 2c.8r VMs that have 40 Gb root disk space. We will not be attaching any external data volumes for our test instances as this won't be available to participants. You will have received an invitation to the project 'nf-core-workflows' via email in early March. You need to accept this invitation to be added to this project. 


>:warning: __REMEMBER:__  please record any tools you needed to install on your instance to run exercises you develop. We will need to install these on the final test instance which will be cloned for all attendees. 

## Set up your test instance 

To set up your test VM, you will need to first create an instance in the OpenStack dashboard: 

1. Login to Pawsey's [OpenStack Dashboard](https://nimbus.pawsey.org.au/auth/login/) with the domain set to Pawsey. Provide your allocated username and password. You will be logged into this page: 

![](https://user-images.githubusercontent.com/73086054/227118001-6896bd25-c629-4212-b452-c9002c0ec68a.png)

2. On the left-side menu select Instances and then the :cloud: `Launch Instance` button on the far right. This will open a pop up window, you will need to fill out a few sections here to create your instance. 

3. Complete the **details** section by naming your instance. All other forms on this page can be left blank. 

![](https://user-images.githubusercontent.com/73086054/227119140-bac7a7b1-4f35-493b-82c5-aff079bffd4c.png)

4. Complete the **source** section with the following:   
    - Select Boot Source: eave as Image 
    - Volume size: leave as 1. Will auto-fill later 
    - Device name: blank 
    - Available: search for 'bio' to get 'Pawsey Bio - Ubuntu 22.04 - 2023-03' and select the up arrow to allocate to your VM. 

![](https://user-images.githubusercontent.com/73086054/227119759-497bb702-7f78-4a71-9f55-9d21d764325e.png)

5. Complete the **Flavor** section by allocating n3.2c8r flavour. To allocate to your VM select the up arrow under Available: 

![](https://user-images.githubusercontent.com/73086054/227120242-0d729ba5-9e82-4b6a-88e0-0b183e439cf3.png)

6. Complete the **Networks** section by allocating a public external IP address. Select the up arrow corresponding to the 'Public external' network option: 

![](https://user-images.githubusercontent.com/73086054/227120768-07240261-fcfa-41f2-a029-244957e3dde7.png)

7. Complete the **Security Groups** section by allocating the nf-core-workflows-SSH-ICMP access using the up arrow again: 

![](https://user-images.githubusercontent.com/73086054/227121054-c35ccd7b-8ee8-4ba3-b323-b3b2c3ee8d58.png)

8. Complete the **Key Pair** section by either creating or importing a key pair. As I already have one set up, I will allocate this to my VM using the up arrow: 

![](https://user-images.githubusercontent.com/73086054/227121387-f7d36659-6c90-4847-8711-ac8d2f3c2eb1.png)

9. All other fields can be ignored. Hit the :cloud: `Launch Instance` button. Your instance will be provisioned immediately. 

## Test your connection 

To establish SSH connection to instances on VS code you'll need to have the [remote SSH extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-ssh) installed. 

1. Once installed, connect to your instance with VS code by adding the host details to your ssh config file. 
    * Ctrl+Shift+P to open command palette 
    * Select Remote-SSH: Open SSH configuraiton file
    * Add new entry, filling out host name and identity file
```
Host <GIVE THE ADDRESS A RECOGNISABLE NAME>
  HostName 146.118.XX.XXX
  IdentityFile path/to/local.pem  
  User ubuntu  	
```
2. Connect to this address
    * Ctrl+Shift+P to open command palette 
    * Select Remote-SSH: Connect to Host and select name of your host
    * Select Linux from dropdown menu and then continue 


