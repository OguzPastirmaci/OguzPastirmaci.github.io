---
layout: post
title: Using Azure cross-platform CLI within shell scripts
---

### Using Azure cross-platform CLI within shell scripts

The Azure cross-platform CLI (I will call it Xplat CLI) provides a set of commands for working with the Azure Platform and it works on Linux, Mac and Windows.

Xplat CLI provides much of the same functionality found in the Azure Management Portal, such as the ability to manage websites, virtual machines, mobile services, SQL Database and other services provided by the Azure platform. You may visit the [repository in Github](https://github.com/Azure/azure-xplat-cli) to get more information about the Xplat CLI. 

One thing I really like about the Xplat CLI is that you can use utilities like *grep*, *awk*, *sed* etc.

While the examples below are for virtual machines, you can use this approach for all the resources available in the Xplat CLI.

Firstly, let's take a look at the output of the command `azure vm list` which lists all the VMs in your Azure account:

{% gist 00ccb82b3a2b11a22736 %}


You can see that the status of the VM is listed in the third column (first column is *data:*, second column is *Name*, and third column is *Status*).

One of the many *awk*some features of *awk* is that you can use the location of the column as a variable. For our example:

- $1 is the first column (data:)
- $2 is the second column (Name)

So what we're going to do is "List all the VMs, get the *Status* of the VMs and start or stop the VMs using the location of the column in the output of `azure vm list` command depending on their *Status*".
 

#### Start all stopped VMs

{% gist 06bdce84f4d6e23479c6 %}

Let's take a look at the command:

1. List all the VMs
2. Pipe the list of the VMs to *grep* to match all the lines with the word *Stopped*.
3. Then pipe to *awk* which will run `azure vm start $2` command for each VM from step 2.

One thing to note here is that I used *Stopped* as the status. This will start all the stopped VMs (*Stopped* and *StoppedDeallocated* status) regardless how the VM was shutdown. The way you used to shut down the VM is important about pricing. Check [this link](https://azure.microsoft.com/en-us/documentation/articles/virtual-machines-questions/#how-does-azure-charge-for-my-vm) for info.



#### Shutdown all running VMs

{% gist 0cbc5b5416f69895b046 %}

Very similar to the previous command but this time it lists the running VMs (VMs with *ReadyRole* status) and shuts them down. 


#### Simple bash script with a menu to start or shutdown the VMs

And here's a very basic bash script with a menu to start or stop all the VMs in your Azure account.

{%gist eb79cf9f93827a3a2322 %}

